

# 参数很重要，如何自动超参数调优
[用Hyperopt对LightGBM进行自动超参调优 - HomeDefaultRisk数据集](https://zhuanlan.zhihu.com/p/108222231)

- 采用贝叶斯优化(Bayesian Optim)的思想
- hyperopt的官网：https://github.com/hyperopt/hyperopt
- 返回的是一个点，space_eval
引自英文版 [Introduction: Automated Hyperparameter Tuning](https://www.kaggle.com/willkoehrsen/automated-model-tuning)

# 阐述HyperOpt思想的文章，被图吸引过去的
https://towardsdatascience.com/hyperopt-hyperparameter-tuning-based-on-bayesian-optimization-7fa32dffaf29


https://neptune.ai/blog/lightgbm-parameters-guide

# 参数调优：
https://stats.stackexchange.com/questions/317073/explanation-of-min-child-weight-in-xgboost-algorithm


## 官网的例子

官方的例子理解稍微绕一下，看下面的例子，如果采用GridSearch则需要40000次，采用Bayesian Optimization方法大概100次迭代就可以找到最优组合了，可以看出:

- **对loss影响越大的参数，其收敛到最佳值的速度更快**。
- 即使不是绝对最佳，也是性价比最好的搜索方法了

```python
# define an objective function
from hyperopt import fmin,Trials,tpe,STATUS_OK,space_eval
def objective(args):
    """可以返回一个标量，或者一个字典{"loss":0.2,"status":STATUS_OK}
       目标函数取最小值
    """
    loss= args['a']*args['b']-args['c']
    return {'loss':loss,'args':args,'status':STATUS_OK}
    

# define a search space
from hyperopt import hp
space = {
#     'a':1 + hp.lognormal('a1', 0, 1), # 知识
    'a':1+ hp.quniform('a1', 0, 100, 1), 
    'b':hp.quniform('b', -10, 10,1),     # 均匀分布，之间的随机数 
    'c':hp.quniform('c', -10, 10,1)     # 均匀分布，之间的随机数 
}
trials = Trials()
best = fmin(objective,space,algo = tpe.suggest,trials=trials,max_evals=300)
print(best)  # 返回的一个参数空间组合情况
# -> {'a1': 100.0, 'b': -10.0, 'c': 10.0}
print(space_eval(space, best)) # 对应的目标函数的入参，即参数空间转换后的
# -> {'a': 101.0, 'b': -10.0, 'c': 10.0}
```
## 绘制迭代次数与超参变化趋势
```python
df = pd.DataFrame(trials.results)
df['iteration']=df.index
df =df.merge(df['args'].apply(lambda x : pd.Series(x)), left_index=True, right_index=True)

# 绘制迭代次数和关键参数的关系
import seaborn as sns
fig, axs = plt.subplots(1, 4, figsize = (24, 6))
# Plot of four hyperparameters
for i, hyper in enumerate(['a', 'b', 'c','loss']):    
        # Scatterplot
        sns.regplot('iteration', hyper, data = df, ax = axs[i])
        axs[i].scatter(df['iteration'], df[hyper], marker = '.', s = 200, c = 'k')
        axs[i].set(xlabel = 'Iteration', ylabel = '{}'.format(hyper), title = '{} over Search'.format(hyper));
plt.tight_layout()
```

# GPU版本能直接用吗？不能

```python
model = lgb.LGBMRegressor(num_leaves= 15,
                          max_depth=3,
                          n_estimators=10000,
                          random_state=1212,
                          learning_rate = 0.001,
                          device="gpu",
                          )
```
会报下面错误:
```
LightGBMError: GPU Tree Learner was not enabled in this build.
Please recompile with CMake option -DUSE_GPU=1
```
