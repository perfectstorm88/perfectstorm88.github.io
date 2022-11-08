# 需求场景

计算一段波动曲线中的波谷，动态显示搜索的过程，代码如下：

```python
# 通过三角函数+随机数模拟曲线
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from pylab import mpl
# start 模拟数据
x = np.arange(0,np.pi*6,0.01)  # （10*60,）
y = np.sin(x)+x*-0.05+np.random.rand(len(x))*0.1


"""
入参：
# window 数据窗口大小
# step 采样步长，最小是1，跟计算性能有关，越小需要的计算量越大
# stride 步幅，最小5,10

按着stride步幅逐个取窗口大小为window的数据块，按step进行取样，计算均值，然后取均值最小的数据块
"""
def find_wave_trough(y,window=40,step=2,stride =20):
    wave_trough= np.mean(y[0:window:step])
    offset = 0
    for offset in range(0,len(y)-window,stride):
        block = y[offset:offset+window:step]
        wave_trough2 = np.mean(block)
        if wave_trough > wave_trough2:
            wave_trough = wave_trough2
    return wave_trough


# 画图
fig, ax = plt.subplots(ncols=1,nrows=1,figsize = (13,5))
ax.plot(x,y) # 原始曲线
wave_trough = find_wave_trough(y[0:window],window,step,stride)
y2 = [wave_trough]*len(x)
line1, = ax.plot(x[0:window],y[0:window],color='red',linewidth=3) # 搜索过程
line2, = ax.plot(x,y2,color='green',linestyle='dashed',linewidth=1) # 搜索到的波谷位置
annotate = ax.annotate(f'wave_trough={wave_trough:0.5f}\nt={0:0.2f}s', (0, 0),
            xytext=(0.2, 0.9), textcoords='axes fraction',
            # arrowprops=dict(facecolor='black', shrink=0.05),
            fontsize=16,
            horizontalalignment='left', verticalalignment='top')

# 动态图更新函数，通过set函数更新内容
def update(t):
   limit = t * stride % (len(x)-window) + window
   wave_trough = find_wave_trough(y[0:limit],window,step,stride)
   y2 = [wave_trough]*len(x)
   line1.set_data((x[limit-window:limit],y[limit-window:limit]))
   line2.set_data((x,y2))
   annotate.set(text=f'wave_trough={wave_trough:0.5f}\nt={t*stride/10:0.2f}s')

# 动态图展示
ani = animation.FuncAnimation(fig, update,frames=400,fargs=())
show() # 或者保存为mp4 ani.save('Liss.mp4')
```

## FuncAnimation函数说明
`FuncAnimation(fig, func, frames=None,fargs=None)` FuncAnimation的参数比较多，这里只列举出最重要的四个。

- fig: 指得是目前曲线所在在的figure。
- func: 这个函数主要是修改lines的属性，并返回。
- frames: 这是func参数中，跟每一帧相关的参数，靠它曲线才会发生变化。
  - 实际上调用arange(frames)产生了这么多数据，每一次传递一个数据给update函数的第一个参数，这样就相当于时间在变化。
- fargs： 对应func中其他参数，第一个参数为t
- interval frame的间隔，默认为200毫秒
- repeat_delay 重复动画的间隔，默认为0毫秒
- 

# 更多资料
参考[官网样例](https://matplotlib.org/stable/gallery/animation/random_walk.html#)，如：
- Decay 模拟衰变过程
- 贝叶斯更新模拟
- The double pendulum problem 双摆问题
- Animated histogram 动画直方图：使用 histogram 的 BarContainer 为动画直方图绘制一堆矩形。
- Rain simulation 雨滴模拟
- Animated 3D random walk 动画 3D 随机游走
- Oscilloscope 示波器
- Animated line plot 动画线图
- Animated image using a precomputed list of images 使用预先计算的图像列表的动画图像


# 参考
- [用Matplotlib制作动画](https://zhuanlan.zhihu.com/p/32444081)