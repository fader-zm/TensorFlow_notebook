# 案例：信息评分数据预测-使用bagging方法

现在，您将使用信用评分数据尝试更复杂的示例。 您将使用具有以下预处理的准备数据：
1. 清除缺失值。
2. 汇总某些分类变量的类别。
3. 将分类变量编码为二进制虚拟变量。
4. 标准化数字变量。

执行下面单元格中的代码，将功能和标签加载为示例的numpy数组。

```python
Features = np.array(pd.read_csv('Credit_Features.csv'))
Labels = np.array(pd.read_csv('Credit_Labels.csv'))
Labels = Labels.reshape(Labels.shape[0],)
print(Features.shape)
print(Labels.shape)
```

嵌套交叉验证用于估计最优超参数并对随机森林模型执行模型选择。 由于随机森林模型训练有效，因此使用10倍交叉验证。 执行下面单元格中的代码以定义内部和外部折叠对象。

```python
nr.seed(123)
inside = ms.KFold(n_splits=10, shuffle = True)
nr.seed(321)
outside = ms.KFold(n_splits=10, shuffle = True)
```

下面单元格中的代码使用10倍交叉验证估计最佳超参数。这里有几点需要注意：

1. 在这种情况下，搜索两个超参数的网格：
   - max_features确定用于确定拆分的最大功能数。最小化特征的数量可以通过诱导偏差来防止模型过度拟合。
   - min_samples_leaf确定必须在树的每个终端节点上的最小样本数或叶数。维持每个终端节点的最小样本数是正则化方法。在终端叶片上的样本太少允许模型训练记忆数据，导致高方差。在终端节点上强制过多样本会导致偏差预测。
2. 由于存在类别不平衡和不良信用风险客户错误分类银行的成本差异，因此使用“平衡”参数。平衡参数确保用于训练每棵树的子样本具有平衡的情况。
3. 该模型适用于网格中的每组超参数。
4. 打印出最佳的估计超参数。

请注意，该模型使用正则化而不是特征选择，超参数搜索旨在优化正则化水平。

```python
## Define the dictionary for the grid search and the model object to search on
param_grid = {"max_features": [2, 3, 5, 10, 15], "min_samples_leaf":[3, 5, 10, 20]}
## Define the random forest model
nr.seed(3456)
rf_clf = RandomForestClassifier(class_weight = "balanced") # class_weight = {0:0.33, 1:0.67}) 

## Perform the grid search over the parameters
nr.seed(4455)
rf_clf = ms.GridSearchCV(estimator = rf_clf, param_grid = param_grid, 
                      cv = inside, # Use the inside folds
                      scoring = 'roc_auc',
                      return_train_score = True)
rf_clf.fit(Features, Labels)
print(rf_clf.best_estimator_.max_features)
print(rf_clf.best_estimator_.min_samples_leaf)
```

现在，您将在下面的单元格中运行代码以执行模型的外部交叉验证。

```python
nr.seed(498)
cv_estimate = ms.cross_val_score(rf_clf, Features, Labels, 
                                 cv = outside) # Use the outside folds

print('Mean performance metric = %4.3f' % np.mean(cv_estimate))
print('SDT of the metric       = %4.3f' % np.std(cv_estimate))
print('Outcomes by cv fold')
for i, x in enumerate(cv_estimate):
    print('Fold %2d    %4.3f' % (i+1, x))
```

检查这些结果。 请注意，AUC平均值的标准偏差比平均值小一个数量级。 这表明该模型可能很好地推广。

现在，您将使用估计的最优超参数构建和测试模型。 作为第一步，执行下面单元格中的代码以创建训练和测试数据集。

```python
## Randomly sample cases to create independent training and test data
nr.seed(1115)
indx = range(Features.shape[0])
indx = ms.train_test_split(indx, test_size = 300)
X_train = Features[indx[0],:]
y_train = np.ravel(Labels[indx[0]])
X_test = Features[indx[1],:]
y_test = np.ravel(Labels[indx[1]])
```

下面单元格中的代码使用估计的最优模型超参数定义随机森林模型对象，然后将模型拟合到训练数据。 执行此代码。

```python
nr.seed(1115)
rf_mod = RandomForestClassifier(class_weight = "balanced", 
                                max_features = rf_clf.best_estimator_.max_features, 
                                min_samples_leaf = rf_clf.best_estimator_.min_samples_leaf) 
rf_mod.fit(X_train, y_train)
```

正如所料，随机森林模型对象的超级参数反映了指定的那些。

下面单元格中的代码使用测试数据子集对模型的评估度量进行评分和打印。

执行此代码，检查结果。

```python
def score_model(probs, threshold):
    return np.array([1 if x > threshold else 0 for x in probs[:, 1]])


def print_metrics(labels, probs, threshold):
    scores = score_model(probs, threshold)
    metrics = sklm.precision_recall_fscore_support(labels, scores)
    conf = sklm.confusion_matrix(labels, scores)
    print('                 Confusion matrix')
    print('                 Score positive    Score negative')
    print('Actual positive    %6d' %
          conf[0, 0] + '             %5d' % conf[0, 1])
    print('Actual negative    %6d' %
          conf[1, 0] + '             %5d' % conf[1, 1])
    print('')
    print('Accuracy        %0.2f' % sklm.accuracy_score(labels, scores))
    print('AUC             %0.2f' % sklm.roc_auc_score(labels, probs[:, 1]))
    print('Macro precision %0.2f' %
          float((float(metrics[0][0]) + float(metrics[0][1]))/2.0))
    print('Macro recall    %0.2f' %
          float((float(metrics[1][0]) + float(metrics[1][1]))/2.0))
    print(' ')
    print('           Positive      Negative')
    print('Num case   %6d' % metrics[3][0] + '        %6d' % metrics[3][1])
    print('Precision  %6.2f' % metrics[0][0] + '        %6.2f' % metrics[0][1])
    print('Recall     %6.2f' % metrics[1][0] + '        %6.2f' % metrics[1][1])
    print('F1         %6.2f' % metrics[2][0] + '        %6.2f' % metrics[2][1])


probabilities = rf_mod.predict_proba(X_test)
print_metrics(y_test, probabilities, 0.5)
```

总的来说，这些性能指标看起来非常好。 绝大多数负面（不良信用）案件的确定是以重大误报为代价的。 报告的AUC在通过交叉验证获得的图的标准偏差内，表明该模型很好地推广。

#### 小结

在本实验中，您已完成以下任务： 

使用10倍来查找随机森林模型的估计最优超参数以对信用风险案例进行分类，该模型似乎能很好地概括。