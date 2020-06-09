---
layout: post
categories: 算法
---


# 生存分析
**生存分析**指的是一系列用来探究所感兴趣的事件的发生的时间的统计方法。常见的有1）癌症患者生存时间分析2）工程中的失败时间分析等等。

## 1.1 定义
给定一个实例 iii，我们用一个三元组来表示 (Xi,δi,Ti)其中

- Xi 表示该实例的特征向量
- Ti表示该实例的事件发生时间。
  - 如果该实例发生了我们感兴趣的事件，那么 Ti表示的是事件发生时间点到基准时间点之间的时间，同时 δi=1。
  - 如果该实例未发生我们感兴趣的事件，那么 Ti表示的是事件发生时间点到观察结束时间点的时间，同时 δi=0。

**生存分析的研究目标**就是对一个新的实例Xj来估计它所发生感兴趣事件的时间。

## 1.2 删失(censored)
在生存分析研究中，对于某些实例，可能在我们的研究期间，并没有出现任何感兴趣的事件，我们将这种情况称之为删失（censored）。
出现这种情况的可能原因有：
- 1）实例在研究阶段就是没有出现感兴趣的事件（right-censored）
- 2）在研究阶段，丢失了该实例
- 3）该实例经历了其他的事件导致无法继续跟踪

## 生存概率(Survival probability)
生存概率也叫作生存方程S(t)=Pr(T>t)，生存方程指的是实例出现感兴趣的事件的时间 T不小于给定的时间t的概率
# 先从kaplan-Meier算法开始
Kaplan-Meier 方法是一种非参数方法，根据观察到的生存时间估算生存概率（Kaplan和Meier，1958年），ti时刻的生存概率S（ti）计算如下：
```
S(ti)=S(t_i−1)/(1−d_i/n_i)
```
- S(t_i−1) = 在ti−1时刻的生存概率
- ni = 在ti之前的病人数
- di = 在ti时刻发生的事件数
- t0 = 0, S(0) = 1

其基本思想是：将生存时间由小到大依次排列，在每个死亡点上，计算其期初人数、死亡人数、死亡概率、生存概率或生存率
比如我们举下面一个例子：
![kaplan-Meier](http://image.sciencenet.cn/album/201607/29/183127mn7vb7ok7omilbzq.png)
那么对应的就有：
![kaplan-Meier](http://image.sciencenet.cn/album/201607/29/1831404lb3lt633kvwlrkw.png)
如果我们存在截尾数据可能就有区别了，比如下表:
![kaplan-Meier](http://image.sciencenet.cn/album/201607/29/183151d42otd4l161zmwv4.png）
有截尾和无结尾对应的生存曲线如下图所示：
![kaplan-Meier](http://image.sciencenet.cn/album/201607/29/183202zketkiiavuuavkk4.png）

## 代码样例
```
from lifelines import KaplanMeierFitter
from lifelines.datasets import load_waltons
waltons = load_waltons()

kmf = KaplanMeierFitter(label="waltons_data")
kmf.fit(waltons['T'], waltons['E'])
kmf.plot()
```
## 关键步骤
1 根据T和E两列，构建event_table,结构如下
```
	removed	observed	censored	entrance	at_risk
event_at					
0.0	0	0	0	34752	34752
1.0	2952	2952	0	0	34752
2.0	2200	2200	0	0	31800
3.0	1886	1886	0	0	29600
4.0	1704	1704	0	0	27714
5.0	26010	246	25764	0	26010
```
2、log_estimate, cumulative_sq_ = _additive_estimate(event_table,timeline...)
_additive_estimate 
```python
        entrances = events["entrance"].copy()
        entrances.iloc[0] = 0
        population = events["at_risk"] - entrances

        estimate_ = np.cumsum(_additive_f(population, deaths))
        var_ = np.cumsum(_additive_var(population, deaths))
```
其中_additive_f代码如下：
```
np.log(population - deaths) - np.log(population)
```
其中_additive_var代码如下：
```
(deaths / (population * (population - deaths))).replace([np.inf], 0)
```
然后survival_function_ = np.exp(log_estimate)

3、计算执行区间，confidence_interval_ = self._bounds(cumulative_sq_[:, None], alpha, ci_labels)
这段代码数学知识比较多，理解有难度
置信区间使用了exponential Greenwood公式，参考https://www.math.wustl.edu/%7Esawyer/handouts/greenwood.pdf
```python
z = inv_normal_cdf(1 - alpha / 2)
df = pd.DataFrame(index=self.timeline)
v = np.log(self.__estimate.values)
ci_labels = ["%s_lower_%g" % (self._label, 1 - alpha), "%s_upper_%g" % (self._label, 1 - alpha)]
df[ci_labels[0]] = np.exp(-np.exp(np.log(-v) - z * np.sqrt(cumulative_sq_) / v))
df[ci_labels[1]] = np.exp(-np.exp(np.log(-v) + z * np.sqrt(cumulative_sq_) / v))
```
风险概率(hazard probability)
风险概率指的是在时间ttt之前还没有发生任何事件的情况下，在时间ttt发生感兴趣的时间的概率。
$$\lim_{\delta(t)\rightarrow0}\frac{Pr(t\leq T\leq t+\delta(t)|T\geq t)}{\delta(t)}$$

 累积风险（cummulative hazard）
在针对单因子进行生存分析时，我们已经得到了生存方程S(t)S(t)S(t)，那么，根据S(t)S(t)S(t)，累积风险为：
```
H(t)=−log(S(t))
```
# cox模型