## 美多项目埋点

- 环境安装：
  - 基本安装：

    在项目代码中找到requirements.txt，进行安装

    `pip install -r requirements.txt`

  - 安装x-admin

    `pip install https://github.com/sshwsfc/xadmin/tarball/master`

  - 安装live-server

    ```
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
    nvm install node
    #默认npm安装很慢，配置淘宝源
    npm config set registry http://registry.npm.taobao.org
    npm install -g live-server
    ```

- 启动chrome(由于存在跨域请求，故关闭chrome的安全模式，否则获取不到数据)

  **启动前关闭所有浏览器**，找到chrome浏览器安装的位置，打开cmd，进行关闭安全模式的启动

  chrome.exe --disable-web-security --user-data-dir

- views.py

```python
class HotSKUListView(ListCacheResponseMixin, ListAPIView):
    """
    热销商品, 使用缓存扩展
    """
    serializer_class = SKUSerializer
    pagination_class = None

    def get_queryset(self):
        category_id = self.kwargs['category_id']
        reco_result = recommendedSystem('detail',category_id)
        reco_result = reco_result[:10]
        print(reco_result)
        #TODO 修改
        reco_skus = SKU.objects.filter(id__in=reco_result)
        return reco_skus
################################### 更改前 #######################################

class HotSKUListView(ListCacheResponseMixin, ListAPIView):
    """
    热销商品, 使用缓存扩展
    """
    serializer_class = SKUSerializer
    pagination_class = None

    def get_queryset(self):
        category_id = self.kwargs['category_id']
        return SKU.objects.filter(category_id=category_id, is_launched=True).order_by('-sales')[:constants.HOT_SKUS_COUNT_LIMIT]


################################### 增加如下内容 ###################################
import logging
log = logging.getLogger('my_logger')
# Create your views here.
class LogView(APIView):
    """
    sku列表数据
    """
    serializer_class = SKUSerializer
    filter_backends = (OrderingFilter,)
    ordering_fields = ('create_time', 'price', 'sales')

    def get(self,request):
        data = request.query_params.dict()
        timeStamp = time.time()
        strfTime = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(timeStamp))

        exposure_loc = data.get("exposure_loc")
        uid = data.get("user_id")
        cate_id = data.get("cate_id")
        sku_id=data.get("sku_id")

        log.info("%s exposure_timesteamp<%d> exposure_loc<%s> uid<%s> sku_id<%s> cate_id<%s>"%(strfTime,timeStamp,exposure_loc,uid,sku_id,cate_id))
        return Response("ok")
 

import redis

client0 = redis.StrictRedis(db=0)  # 此前redis 0号库中已经存储了每个商品的TOP-N个相似商品
client1 = redis.StrictRedis(db=1)

#TODO 加入了推荐逻辑
def recommendedSystem(dest, id):
    # 离线计算出的相似物品推荐召回
    if dest == "detail":
        sku_id= id
        sim_skus = client0.zrevrange(sku_id, 0, 20)  # 取出最相似的前20个
        return sim_skus

    # 实时计算出的用户个性化推荐召回
    if dest == "cart":
        uid = id
        sim_skus = client1.scan(uid)
        return sim_skus
```

- urls.py 增加埋点功能对应接口的url地址

```python
urlpatterns = [
    url(r'^categories/(?P<category_id>\d+)/hotskus/$', views.HotSKUListView.as_view()),
    url(r'^categories/(?P<category_id>\d+)/skus/$', views.SKUListView.as_view()),
    url(r'^categories/click/skus/$', views.LogView.as_view()),#TODO 埋点的后台url配置
    url(r'^categories/(?P<pk>\d+)/$', views.CategoryView.as_view()),
    url(r'^skus/(?P<sku_id>\d+)/comments/$', views.SKUCommentsListView.as_view()),
]
```

- dev.py

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
        },
        #埋点
        'exposure': {
            'format': '%(message)s'
        },
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(os.path.dirname(BASE_DIR), "logs/meiduo.log"),
            'maxBytes': 300 * 1024 * 1024,
            'backupCount': 10,
            'formatter': 'verbose'
        },
        #埋点
        'file1': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(os.path.dirname(BASE_DIR), "logs/exposure.log"),
            'maxBytes': 300 * 1024 * 1024,
            'backupCount': 10,
            'formatter': 'exposure'
        }
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'propagate': True,
            'level': 'INFO'
        },
        #埋点
        'my_logger': {
            'handlers': ['file1'],
            'propagate': True,
            'level': 'INFO'
        },
    }
}

```

### 前端代码

- detail.js

```javascript
  bury_goods_list:function(e){
            sessionStorage.cate_id=this.cat
            axios.get(this.host+'/categories/click/skus/', {
                    params: {
                        cate_id: this.cat,
                        user_id:this.user_id,
                        sku_id:this.sku_id,
                        exposure_loc: "detail"
                    },
                    responseType: 'json'
                })
                .then(response => {
                    console.log("ok")
                })
                .catch(error => {
                    console.log(error.response.data);
                })
        }
	bury_add_cart:function(){
            //TODO 埋点了添加购物车
            axios.get(this.host+'/categories/click/skus/', {
                    params: {
                        cate_id: this.cat,
                        user_id:this.user_id,
                        sku_id:this.sku_id,
                        exposure_loc: "cart"
                    },
                    responseType: 'json'
                })
                .then(response => {
                    console.log("ok")
                })
                .catch(error => {
                    console.log(error.response.data);
                })
        },
            
 /******************************修改了如下代码******************************/
 mounted: function(){
        // 添加用户浏览历史记录
        this.get_sku_id();
        if (this.user_id) {
            axios.post(this.host+'/browse_histories/', {
                sku_id: this.sku_id
            }, {
                headers: {
                    'Authorization': 'JWT ' + this.token
                }
            })
        }
        this.get_cart();
        this.get_hot_goods();
        this.get_comments();
        this.bury_goods_list()
    },
        
    add_cart: function(){
            axios.post(this.host+'/cart/', {
                    sku_id: parseInt(this.sku_id),
                    count: this.sku_count
                }, {
                    headers: {
                        'Authorization': 'JWT ' + this.token
                    },
                    responseType: 'json',
                    withCredentials: true
                })
                .then(response => {
                	//TODO 调用方法
                    this.bury_add_cart()
                    alert('添加购物车成功');
                    this.cart_total_count += response.data.count;
                })
                .catch(error => {
                	//TODO 调用方法
                    this.bury_add_cart()
                    if ('non_field_errors' in error.response.data) {
                        alert(error.response.data.non_field_errors[0]);
                    } else {
                        alert('添加购物车失败');
                    }
                    console.log(error.response.data);
                })
        },
```

关闭chrome浏览器的 -web-security

"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disable-web-security --user-data-dir

## Flume & kafka安装

**Kafka安装**

安装步骤：

安装kafka，需要先安装Zookeeper

- 1，下载zookeeper-3.4.5-cdh5.7.0，配置ZK_HOME
- 2，进入conf目录，配置zoo.cfg文件
  - 修改dataDir=/xxx(默认为系统tmp目录下，重启后数据会丢失)
- 3，启动zookeeper
  - zkServer.sh start
- 4，利用zkClient连接服务端
  - zkClient.sh

安装kafka

- 1，下载Kafka-2.11-1.10.tgz，解压

- 2，配置KAFKA_HOME

- 3，config目录下server.properties

  - broker.id=0 在集群中需要设置为唯一的
  - zookeeper.connect=localhost:2181

- 4，启动kafka

  - bin/kafka-server-start.sh config/server.properties

    看到started，说明启动成功

**Flume安装**

然后解压  tar -zxvf apache-flume-1.6.0-bin.tar.gz

然后进入 flume 的目录，修改 conf 下的 flume-env.sh，在里面配置 JAVA_HOME

配置flume环境变量：

​	vi ~/.bash_profile

​	export FLUME_HOME=xxx/apache-flume-1.6.0-cdh5.7.0-bin

​	export PATH=\$FLUME_HOME/bin:$PATH

​	source 生效

检查是否配置成功：flume-ng version查看flume版本

根据数据采集需求**配置采集方案**，描述在配置文件中(文件名可任意自定义)

**指定采集方案配置文件**，在相应的节点上启动 flume agent

