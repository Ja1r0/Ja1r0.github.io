---
title: TRPO论文阅读
date: 2019-03-24 23:08:40
tags: 论文阅读
mathjax: true
---

TRPO论文的推导过程。

<!-- more -->

# Trust Region Policy Optimization

```
               John Schulman
```

# 概述

- 描述了一个用来**优化策略**的**迭代过程**
- 这个过程是使得优化过程**单调提高**的
- 在对理论证明过程进行几处**近似**之后，提出一个实际算法`TRPO`
- 该算法对于优化**大规模非线性策略**，例如神经网络，有较高的效率

# 符号约定

- $S$——有限状态空间
- $A$——有限动作空间
- $P:S\times A\times S\to \mathbb{R}$——转移概率分布
- $r:S\to \mathbb{R}$——奖励函数
- $\rho_0:S\to \mathbb{R}$——初始状态$s_0$的概率分布
- $\gamma \in (0,1)$——折扣因子
- $\pi:S\times A \to [0,1]$——随机策略
- $\eta(\pi)$——期望折扣回报  

$$
\eta(\pi)=\mathbb{E}_{s_0,a_0,\ldots}\left[ \sum_{t=0}^{\infty}\gamma^tr(s_t) \right]
$$

其中：
$$
s_0\sim \rho_0(s_0),\quad a_t\sim \pi (a_t\mid s_t),\quad s_{t+1}\sim P(s_{t+1}\mid s_t,a_t)
$$
并定义如下三个量：

1. 状态-动作价值函数：

$$
Q_\pi (s_t,a_t)=\mathbb{E}_{s_{t+1},a_{t+1},\ldots}\left[ \sum_{l=0}^{\infty}\gamma^l r(s_{t+l}) \right]
$$

2. 价值函数：

$$
V_{\pi}(s_t)=\mathbb{E}_{a_t,s_{t+1},\ldots} \left[\sum_{l=0}^{\infty}\gamma^l r(s_{t+l}) \right]
$$

3. 优势函数：

$$
A_\pi (s,a)=Q_\pi(s,a)-V_\pi(s)
$$

其中：
$$
a_t\sim \pi (a_t\mid s_t),\quad s_{t+1}\sim P(s_{t+1}\mid s_t,a_t)\quad for \quad t\ge 0
$$

- $V_\pi(s_t)$是状态$s_t$下，对所有可能动作$a$而言的期望价值
- $Q_\pi(s_t,a_t)$是状态$s_t$下，当采取动作$a_t$时对应的价值
- $A_\pi(s_t,a_t)$反映了状态$s_t$下，选择某一个动作对应的价值，和对于所有可能动作的期望价值的差

# 重写$\eta(\pi)$

## ——写成增量式

我们希望每一次对策略$\pi$的更新，可以使得$\eta(\pi)$单调增大，能否将$\eta(\pi)$的表达式写成如下形式：
$$
\eta(\widetilde{\pi})=\eta(\pi)+(\ldots)
$$
这样的话，我们只需要考虑如何使得$(\ldots)\ge0$，变可保证$\eta(\pi)$单调增大  

### 由$A_\pi(s,a)$重新定义$\eta(\pi)$ ：

$$
\eta(\widetilde{\pi})=\eta(\pi)+\mathbb{E}_{s_0,a_0,\ldots \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]
$$

### $proof$ ：

考察两个策略$\pi$和$\widetilde{\pi}$。   
首先注意到如下等式：
$$
A_\pi(s,a)=r(s)+\gamma V_\pi(s')-V_\pi(s)
$$
于是有如下关系
$$
\begin{eqnarray}
& &\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]   \\
&=&\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t (r(s_t)+\gamma V_\pi(s_{t+1})-V_\pi(s_t)) \right] \\
&=&\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^{t+1}V_\pi(s_{t+1})- \sum_{t=0}^{\infty}\gamma^{t}V_\pi(s_{t})+\sum_{t=0}^{\infty}\gamma^t r({s_t}) \right] \\
&=&\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ -V_\pi(s_0)+\sum_{t=0}^{\infty}\gamma^t r({s_t}) \right] \\
\end{eqnarray}
$$
注意到
$$
\begin{eqnarray}
\left\{
\begin{array}{ll}
\eta(\pi) &=& \mathbb{E}_{s_0,a_0,\ldots}\left[ \sum_{t=0}^{\infty}\gamma^tr(s_t) \right] \\
V_{\pi}(s_t) &=& \mathbb{E}_{a_t,s_{t+1},\ldots} \left[\sum_{l=0}^{\infty}\gamma^l r(s_{t+l}) \right]
\end{array}
\right.
\Rightarrow V_\pi(s_0)=\eta(\pi)
\end{eqnarray}
$$
于是有
$$
\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]=-\eta(\pi)+\eta(\widetilde{\pi})
$$
即
$$
\eta(\widetilde{\pi})=\eta(\pi)+\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]
$$

# 进一步改写$\eta(\pi)$

## ——显式的写出$s$和$a$

$$
\eta(\widetilde{\pi})=\eta(\pi)+\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]
$$

可以看到我们成功的把对策略进行评价的折扣回报函数$\eta(\pi)$写成了$\eta(\widetilde{\pi})=\eta(\pi)+(\ldots)$的形式，于是我们要考虑这一项何时为正，为正时则要对策略进行更新。但这一表达式并没有给出太多信息，我们来把其中的状态$s$和动作$a$显式的表现出来：
$$
\begin{eqnarray}
\eta(\widetilde{\pi}) &=& \eta(\pi)+\mathbb{E}_{\tau \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]  \\
\eta(\widetilde{\pi}) &=& \eta(\pi)+\sum_{t=0}^\infty  \sum_s P(s_t=s \mid \widetilde{\pi}) \sum_a \widetilde{\pi}(a_t=a \mid s_t) \gamma^t A_\pi(s_t,a_t) \qquad 期望是以概率为权重的加权和\\ 
\eta(\widetilde{\pi}) &=& \eta(\pi)+ \sum_s \sum_{t=0}^\infty   \gamma^t  P(s_t=s \mid \widetilde{\pi})  \sum_a \widetilde{\pi}(a_t=a \mid s_t) A_\pi(s_t,a_t)  \qquad 调整各项的位置
\end{eqnarray}
$$
定义**折扣状态访问概率**：
$$
\rho_\pi (s)=P(s_0=s \mid \pi)+\gamma P(s_1=s \mid \pi)+\gamma^2 P(s_2=s \mid \pi)+\dots=\sum_{t=0}^\infty \gamma^t  P(s_t=s \mid \pi)
$$
表示策略$\pi$下，带有折扣因子的访问到状态$s$的概率（没有归一化），此时$\eta(\widetilde{\pi})$为：
$$
\begin{eqnarray}
\eta(\widetilde{\pi}) &=& \eta(\pi)+ \sum_s \sum_{t=0}^\infty \gamma^t  P(s_t=s \mid \widetilde{\pi})  \sum_a \widetilde{\pi}(a_t=a \mid s_t) A_\pi(s_t,a_t) \\
\eta(\widetilde{\pi}) &=& \eta(\pi)+  \sum_s  \rho_\widetilde{\pi}(s) \sum_a \widetilde{\pi}(a \mid s) A_\pi(s,a)
\end{eqnarray}
$$
从这个式子可以看出，对于一个新策略$\widetilde{\pi}$如何判断其是否为更优的策略？就是对于在新策略$\widetilde{\pi}$下，对所有可能到达的状态$s$，考察其**期望优势值**，若有：
$$
\sum_a \widetilde{\pi}(a \mid s) A_\pi(s,a) \ge 0
$$
则说明$\widetilde{\pi}$为更优的策略，在所考察的状态$s$处，依据下式更新策略：
$$
\widetilde{\pi}(s)=\mathop{\arg\max}_a A_\pi(s,a)
$$
直到对于所有$\widetilde{\pi}$下可能到达的状态$s$，和状态$s$下所有可能采取的动作$a$都不再有正的$A_\pi(s,a)$，则说明收敛到最优策略。

# 第一次近似

上一节推导出的表达式中，在新策略$\widetilde{\pi}$下，对所有可能到达的状态$s$，和状态$s$下所有可能采取的动作$a$，考察其**期望优势值**$A_\pi(s,a)$。对于一个参数化的策略$\pi_\theta(a \mid s)$，给定状态$s$输出动作$a$是很直接的，但若用其对状态采样，获得$\rho_{\pi}(s)$则是一个漫长的过程。考虑忽略**折扣状态访问概率**因策略更新而产生的变化，用$\rho_{\pi}(s)$替代$\rho_\widetilde{\pi}(s)$，于是便有了$\eta(\widetilde{\pi})$的近似表达式$L_\pi(\widetilde{\pi})$：
$$
L_\pi(\widetilde{\pi})=\eta(\pi)+  \sum_s  \rho_{\pi}(s) \sum_a \widetilde{\pi}(a \mid s) A_\pi(s,a)
$$
那么这样近似之后还可信吗？方便对比写出原式：
$$
\eta(\widetilde{\pi}) = \eta(\pi)+  \sum_s  \rho_\widetilde{\pi}(s) \sum_a \widetilde{\pi}(a \mid s) A_\pi(s,a)
$$
对于一个关于参数向量$\theta$可微的策略函数$\pi_\theta(a \mid s)$，有这样的关系：
$$
\begin{eqnarray}
\left\{
\begin{array}{ll}
L_{\pi_{\theta_{old}}}(\pi_{\theta_{old}})=\eta(\pi_{\theta_{old}}) \\
\nabla_\theta L_{\pi_{\theta_{old}}}(\pi_{\theta})\mid _{\theta=\theta_{old}}=\nabla_\theta \eta (\pi_{\theta})\mid_{\theta=\theta_{old}}
\end{array}
\right.
\end{eqnarray}
$$

### $proof:$

对第一个式子，因为策略并没有改变，故：
$$
L_{\pi_{\theta_{old}}}(\pi_{\theta_{old}})=\eta(\pi_{\theta_{old}})+\sum_s  \rho_{\pi_{\theta_{old}}}(s) \sum_a \pi_{\theta_{old}}(a \mid s) A_{\pi_{\theta_{old}}}(s,a)=\eta(\pi_{\theta_{old}})
$$
对第二个式子，分别来计算左右两项：
$$
\nabla_\theta L_{\pi_{\theta_{old}}}(\pi_{\theta})\mid _{\theta=\theta_{old}}=\sum_s  \rho_{\pi_{\theta_{old}}}(s) \sum_a \nabla_\theta\pi_{\theta}(a \mid s) A_{\pi_{\theta_{old}}}(s,a) \mid_{\theta=\theta_{old}} \\
\nabla_\theta \eta (\pi_{\theta})\mid_{\theta=\theta_{old}}=\sum_s  \rho_{\pi_{\theta}}(s) \sum_a \nabla_\theta\pi_{\theta}(a \mid s) A_{\pi_{\theta_{old}}}(s,a) \mid_{\theta=\theta_{old}}
$$
实际操作中，$\sum_s  \rho_{\pi_{\theta}}(s)$是由样本信息得到的，当安照$\theta=\theta_{old}$，即$\pi_{\theta_{old}}$采样时，$\sum_s  \rho_{\pi_{\theta_{old}}}(s)=\sum_s  \rho_{\pi_{\theta}}(s)$，左右两项相等。

### 结论:

表明对于函数$L_{\pi_{\theta_{old}}}(\pi_{\theta})$和$\eta (\pi_{\theta})$，在$\theta_{old}$处，对$\theta_{old}$更新足够小的一步，那么对$L_{\pi_{\theta_{old}}}(\pi_{\theta})$的提高相等于对$\eta (\pi_{\theta})$的提高。

# 策略更新的定量描述

经过上面的讨论，现在就面临两个问题：

1. 如何衡量$\theta_{old}$与$\theta_{new}$之间的距离
2. 更新策略后，会带来$\eta(\pi_{\theta})$多大的提升

本文提出的衡量$\pi_{old}$和$\pi_{new}（此后用\pi_{old}表示\pi_{\theta_{old}}，用\pi_{new}表示\pi_{\theta_{new}}）$之间距离的方式为：$the\ total\  variation\  divergence$

### 定义

$the\ total\  variation\  divergence$的表达式：
$$
D_{TV}(p||q)=\frac{1}{2}\sum_i \left| p_i-q_i \right|
$$
其中$p$和$q$为离散随机变量的概率函数。如果对连续随机变量，则$p$和$q$表示概率密度函数，需将求和改为积分。定义：
$$
D_{TV}^{max}(\pi_{old},\pi_{new})=\max_sD_{TV}(\pi_{old}(\cdot \mid s)\parallel \pi_{new}(\cdot \mid s))
$$

### 重要不等式

有如下不等式关系：
$$
\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new})-\frac{4\epsilon \gamma}{(1-\gamma)^2}\alpha ^2
$$
其中：
$$
\epsilon=\max_{s,a}\left | A_\pi(s,a) \right| \\
\alpha=D_{TV}^{max}(\pi_{old},\pi_{new})
$$
注意到$total\  variation\  divergence$ 和 $KL\ divergence$之间的关系：
$$
D_{TV}(p \parallel q)^2 \le D_{KL}(p \parallel q)
$$
令
$$
D_{KL}^{max}(\pi_{old},\pi_{new})=\max_sD_{TV}(\pi_{old}(\cdot \mid s)\parallel \pi_{new}(\cdot \mid s))
$$
于是经过变换得到了本文的重要不等式：
$$
\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new})-C\cdot D_{KL}^{max}(\pi_{old},\pi_{new})
$$
其中
$$
C=\frac{4\epsilon \gamma}{(1-\gamma)^2}
$$

# 策略更新的方式

经过上面的讨论我们得出了一个重要的不等关系：
$$
\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new})-C\cdot D_{KL}^{max}(\pi_{old},\pi_{new})
$$
更改一下变量脚标： 
此后以$\pi_i$表示当前策略，以$\pi_{i+1}$表示更新后的策略​
$$
\eta(\pi)\ge L_{\pi_{i}}(\pi)-C\cdot D_{KL}^{max}(\pi_{i},\pi)
$$
这是一个以$\pi$为变量的不等式，在$\pi=\pi_i$时取等

可以利用这一不等关系对参数化的策略函数进行更新。此不等式表明了$\eta(\pi)$的下限，我们用$M_i(\pi)$来表示此下限：
$$
M_i(\pi)=L_{\pi_i}(\pi)-C\cdot D_{KL}^{max}(\pi_{i},\pi)
$$
通过提升$\eta(\pi)$的下限$M_i(\pi)$，来带来$\eta(\pi)$的提升：


$$
\begin{eqnarray}
\left\{
\begin{array}{ll}
\eta(\pi_{i+1})\ge M_i(\pi_{i+1}) \\
\eta(\pi_i)=M_i(\pi_i)=L_{\pi_i}(\pi_i)
\end{array}
\right.
\Rightarrow
\eta(\pi_{i+1})-\eta(\pi_i)\ge M_i(\pi_{i+1})-M_i(\pi_i)
\end{eqnarray}
$$
这种更新是一种最小最大算法：
{% asset_img algo1.png %}
{% asset_img minimax.PNG %}

# 第二次近似

以上我们得出了可以对策略迭代更新的算法，下面来考虑将其应用于实际问题。以下所讨论的策略$\pi$均是以向量$\theta$为参数的参数化的策略$\pi_{\theta}$，集中约定一下符号表示：

- $\eta(\theta):=\eta(\pi_\theta)$
- $L_\theta(\widetilde{\theta}):=L_{\pi_{\theta}}(\pi_{\widetilde{\theta}})$
- $D_{KL}(\theta \parallel \widetilde{\theta}):=D_{KL}(\pi_{\theta}\parallel \pi_{\widetilde{\theta}})$
- $用\theta_{old}表示待更新的参数$  

之前得到的不等式为：
$$
\eta(\theta)\ge L_{\theta_{old}}(\theta)-C\cdot D_{KL}^{max}(\theta_{old}\parallel \theta)
$$
更新$\theta$的过程是这样的：找到一个$\theta$使得不等号右边值最大，然后令$\theta_{old}=\theta$：
$$
\max_\theta \left[  L_{\theta_{old}}(\theta)-C\cdot D_{KL}^{max}(\theta_{old}\parallel \theta)\right]
$$
而这一更新方式在使用中有**如下问题**：

- 如果使用理论推导出的不等式，系数$C=\frac{4\epsilon \gamma}{(1-\gamma)^2}$，那么更新步幅会很小，更新会很慢，而着眼于$D_{KL}^{max}$项，通过对这一项进行约束，将问题转换成有约束的优化问题，可以获得较大的更新步幅。这种对于新旧策略间差距的约束称为$trust \ region \ constraint$（这是TRPO区别于PPO的地方，PPO使用可调节大小的$C$）：

$$
\max_\theta L_{\theta_{old}}(\theta) \\
subject\ to \quad  D_{KL}^{max}(\theta_{old}\parallel \theta)\le \delta
$$

- 注意到$D_{KL}^{max}$的定义式：

$$
D_{KL}^{max}=\max_s D_{KL}(\pi_{\theta_{old}}(\cdot \mid s) \parallel \pi_{\theta}(\cdot \mid s) )
$$

​	   可见$ D_{KL}^{max}(\theta_{old}\parallel \theta)\le \delta$这个约束是施加于所有状态的，要对每一个状态进行考察。

基于以上两点讨论，本文进行了**第二次近似**：

#### 用$average \ KL \ divergence$代替原来的$D_{KL}^{max}$约束项

## 定义

$$
\bar{D}_{KL}^{\rho}(\theta_1,\theta_2):=\mathbb{E}_{s\sim \rho}\left[D_{KL}(\pi_{\theta_{1}}(\cdot \mid s) \parallel \pi_{\theta_{2}}(\cdot \mid s) )\right]
$$

这不是对所有可能到达的状态$s$中可取到的$D_{KL}$的最大值，而是考虑对于采样到的所有$s$的$D_{KL}$期望值。于是问题变为：
$$
\max_\theta L_{\theta_{old}}(\theta) \\
subject \ to \quad \bar{D}_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta
$$
这样近似之后，是否可行？**在实验中发现，$\bar{D}_{KL}^{\rho}$和$D_{KL}^{max}$有着相似的表现**。

# 第三次近似

## ——这一次近似了三处

接下来进一步走向实际使用，用$sample-based \ estimation$和$Monte Carlo simulation​$去估计**目标函数**和**约束条件**：   
将目标函数展开：
{% asset_img object.PNG %}

- 对$A_{\theta_{old}}(s,a)$的替换：

$$
\begin{eqnarray}
\sum_a \pi_{\theta}(a \mid s) A_{\theta_{old}}(s,a) &=& \sum_a \pi_{\theta}(a \mid s)[Q_{\theta_{old}}(s,a)-V_{\theta_{old}}(s)]\\
&=& \sum_a [\pi_{\theta}(a \mid s) Q_{\theta_{old}}(s,a)]- V_{\theta_{old}}(s) \sum_a \pi_{\theta}(a \mid s)    \\
&=& \sum_a \pi_{\theta}(a \mid s) Q_{\theta_{old}}(s,a)-V_{\theta_{old}}(s)
\end{eqnarray}
$$

# 优化目标的最终形态

$$
\max_{\theta}\mathbb{E}_{s\sim \rho_{\theta_{old}},a\sim q}\left[ \frac{\pi_{\theta}(a \mid s)}{q(a\mid s)}Q_{\theta_{old}(s,a)}\right]\\
subject \ to \quad \mathbb{E}_{s\sim \rho_{\theta_{old}}}\left[ D_{KL}(\pi_{\theta_{old}}(\cdot \mid s)\parallel \pi_{\theta}(\cdot \mid s))\right] \le \delta
$$

在实际操作时：

- 用样本均值$(sample \ averages)$替换期望$\mathbb{E}$
- 用实验估计$(empirical \ estimate)$替换$Q$值

# 采样估计方式

1. Single Path

2. Vine
   {% asset_img sample_path.png %}

   ## Single Path

3. 通过$s_0\sim \rho_0$选取一个$s_0$

4. 由这个$s_0$出发，执行$\pi_{\theta_{old}}$。经过$T$步之后结束，得到一条轨迹：$s_0,a_0,s_1,a_1,\dots,s_{T-1},a_{T-1},s_T$

5. $q(a\mid s)$即为$\pi_{\theta_{old}}(a\mid s)$

6. $Q_{\theta_{old}(s,a)}$：在每一步$t$，计算从$t$到$T$之间的这一段轨迹的折扣回报总和

# 算法流程梳理

1. 使用$Single Path$或$Vine$的方式采集样本，即状态-动作对，估计$Q-Value$
2. 将需要的量关于所有样本求平均，求出下式中需要估计出的值和约束项

$$
\max_{\theta}\mathbb{E}_{s\sim \rho_{\theta_{old}},a\sim q}\left[ \frac{\pi_{\theta}(a \mid s)}{q(a\mid s)}Q_{\theta_{old}(s,a)}\right]\\
subject \ to \quad \mathbb{E}_{s\sim \rho_{\theta_{old}}}\left[ D_{KL}(\pi_{\theta_{old}}(\cdot \mid s)\parallel \pi_{\theta}(\cdot \mid s))\right] \le \delta
$$

3. 求解这一带有约束的最优化问题，去更新策略参数$\theta$

# 优化过程

$$
\max_\theta L_{\theta_{old}}(\theta) \\
subject \ to \quad \bar{D}_{KL}^{\rho_{\theta_{old}}}(\theta_{old},\theta) \le \delta
$$

用蒙特卡洛方法估计出目标函数与约束方程中的待估计值之后，来考虑如何解这一有约束的最优化问题。目标函数和约束方程都是关于策略参数$\theta$的函数，记作：
$$
\max_\theta l(\theta) \\
subject \ to \  kl(\theta) \le \delta
$$
在$\theta_{old}$处，将 $l(\theta)$ 与$ kl(\theta)$做$Taylor$展开：  
$l(\theta) \approx l(\theta_{old})+\nabla l(\theta_{old})^T(\theta-\theta_{old})+\frac{1}{2}(\theta-\theta_{old})^TH(l)(\theta_{old})(\theta-\theta_{old}) \qquad 第一项为常数；第三项极小\\ 
\ \quad\approx g(\theta-\theta_{old})$  
其中$g=\nabla l(\theta_{old})^T=\frac{\partial}{\partial \theta}l(\theta)\mid _{\theta=\theta_{old}}$  
$kl(\theta) \approx kl(\theta_{old})+\nabla kl(\theta_{old})^T(\theta-\theta_{old})+\frac{1}{2}(\theta-\theta_{old})^T H(kl)(\theta_{old})(\theta-\theta_{old}) \qquad 第一项为0；第二项为0\\
\ \ \ \quad\approx \frac{1}{2}(\theta-\theta_{old})^TF(\theta-\theta_{old})$   
其中$F=H(kl)(\theta_{old})=\frac{\partial^2}{\partial^2\theta}kl(\theta)\mid_{\theta=\theta_{old}}$  
此时优化问题近似成：
$$
\max_\theta  g(\theta-\theta_{old}) \\
subject \ to \  \frac{1}{2}(\theta-\theta_{old})^TF(\theta-\theta_{old}) \le \delta
$$
其中$g=\nabla l(\theta_{old})^T=\frac{\partial}{\partial \theta}l(\theta)^T\mid _{\theta=\theta_{old}}$  
$F=H(kl)(\theta_{old})=\frac{\partial^2}{\partial^2\theta}kl(\theta)\mid_{\theta=\theta_{old}}$  

# 转换成无约束优化问题

构建拉格朗日函数：
$$
L(\theta,\lambda)=g(\theta-\theta_{old})-\frac{\lambda}{2}\left[(\theta-\theta_{old})^TF(\theta-\theta_{old})-\delta \right]
$$
因为约束项为不等式，故还应该满足$KKT$条件：
$$
\begin{eqnarray}
\left\{
\begin{array}{ll}
\frac{1}{2}(\theta-\theta_{old})^TF(\theta-\theta_{old}) \le \delta \\
\lambda \ge 0 \\
\lambda \frac{1}{2}[(\theta-\theta_{old})^TF(\theta-\theta_{old})-\delta] = 0 \\
\end{array}
\right. \Rightarrow \qquad \frac{1}{2}s^TFs=\delta
\end{eqnarray}
$$
其中$s$为参数$\theta$的更新方向$s=\theta-\theta_{old}$  

## 至此我们就可以真正开始寻找函数的极值了

# 最速下降法

我们考虑这样的函数：标量函数$ f(x) \in \mathbb{R}$，其有着向量变元$x\in \mathbb{R}^n$，即：
$$
f(x):\mathbb{R}^n\to \mathbb{R}
$$
其梯度为（函数值增大最快的方向）：
$$
\nabla f(x)=\left< \frac{\partial f}{\partial x_1},\frac{\partial f}{\partial x_2},\ldots,\frac{\partial f}{\partial x_n} \right >
$$
算法流程：

1. 随机选择初始点$x_0$
2. 在当前点处计算函数梯度$v_i=\nabla f(x_i)$
3. 沿负梯度方向进行点的更新$x_{i+1}=x_i-\alpha v_i$
4. 不断进行1.、2.步直到函数值为0或足够小

{% asset_img gd.png %}
最速下降法的问题：

1. 当前点梯度反方向并不一定指向函数极值点
2. 更新步长的选取$\alpha$要经过多次调整到合适的值

# 牛顿法

{% asset_img gradient.PNG %}
先来看一个凸函数：
$$
f(x)=\frac{1}{2}x^TQx+qx+c
$$
其中$Q\in \mathbb{R}^{n\times n}$为对称正定矩阵，$q\in \mathbb{R}^n$  
对于一个凸函数而言，任何局部极小值点都是该函数的全局极小点，若该函数是可微的，那么该函数极值点满足：
$$
\nabla f(x)=0
$$
于是有：
$$
Qx+b=0\quad \Rightarrow \quad Qx=-b \quad \Rightarrow\quad x=-Q^{-1}b
$$
这便直接找到$f(x)$的极值点。对比最速下降法，牛顿法利用了函数的二阶微分，即“梯度的梯度”，利用**Hessian矩阵**获得了更多的信息。当函数为凸函数时，可以直达函数极值点。  
而对于**更一般的函数**，则可通过不断对其在迭代点进行**二阶泰勒展开**进行处理：
$$
f(x_i+\Delta x) \approx f(x_i)+\nabla f(x_i)\Delta x+\frac{1}{2}H[f(x_i)]\Delta x^2
$$
用此二次函数作为原函数$f(x)$的近似，求得此二次函数的极值点：
$$
x_{i+1}=x_i-H[f(x_i)]^{-1}\nabla f(x_i)
$$
然后再在$x_{i+1}$处进行泰勒展开得到新的近似函数，并继续寻找“近似函数”的极值点。迭代进行这一过程。  
但牛顿法的局限性是很明显的：需要计算函数的Hessian矩阵——

1. 有着$O(n^2)$的存储空间要求和运算复杂度
2. 尤其在神经网络中，二阶微分的传播需要的算法完全不同于$bp$算法

# 共轭梯度法

像最速下降法一样寻找一个更新方向，不奢求像牛顿法那样一步到位，只求方向选的科学、且有较大的更新步长。

## 基本思路

对一个凸函数，
$$
f(x)=\frac{1}{2}x^TQx+qx+c
$$
其中$Q\in \mathbb{R}^{n\times n}$为对称正定矩阵，$q\in \mathbb{R}^n$  

找到一组方向向量$d_1,\ldots,d_n \in \mathbb{R}^n$，依次按此方向组中的方向对迭代点$x_{i}\in \mathbb{R}^n$进行更新，对每一个方向$d_i$找到一个合适的步长$\lambda_i$，使得$f(x)$在该方向上取得最小值。 

有这样一个关键**要求**，在每一个新的更新方向$d_k$对迭代点$x_k$进行更新时，不能影响在之前方向$d_j(j\le k-1)$上的更新结果。也即$x_{k+1}$不仅使$f(x)$在$d_k$方向上取得最小值且在$d_j(j\le k-1)$方向上均保持最小值。  

如果能找到这样一组方向$d_1,\ldots,d_n \in \mathbb{R}^n$，那么在迭代$n$次之后将找到$f(x)$的全局极值点。

而矩阵**$Q$的共轭方向组**，正是我们寻找的这样一组方向。

## 算法流程

1. 任意选择初始点$x_0$，初始更新方向$d_0=\nabla f(x_0)$
2. 若$\nabla f(x_i)=0$，说明以找到极值点
3. 若$\nabla f(x_i)\ne 0$，进行点的更新：$x_{i+1}=x_i+\lambda_i d_i$，  
   其中  
    $\lambda_i=\frac{-d_i^T\nabla f(x_i)}{d_i^TQd_i}$  
    $d_i=-\nabla f(x_i)+\gamma_{i-1}d_{i-1}$，这里$\gamma_{i-1}=\frac{d_{i-1}^TQ\nabla f(x_i)}{d_{i-1}^T Qd_{i-1}}$

## 扩展到一般函数

如牛顿法一样，先在迭代点处，将函数进行二阶$Taylor$展开，用这一二次函数对原函数做一个相对好的近似，然后一步步到达该二次函数的极值点。之后进行迭代点的更新，再新的迭代点再做$Taylor$展开，寻找极值点。这样迭代的进行下去。

## 对比牛顿法

这就是说，对于牛顿法直接解出的极值点$x=-Q^{-1}b$，共轭梯度法用了**$n$步**去走到该点。但每一步都方向明确，步长坚定，抵达每一个方向上的函数极值点，因为$Q-共轭方向组$的性质**保证了**之后的更新，不会再影响此方向上的更新结果。  
**不用对Hessian矩阵求逆**。但仍需对$Hessian$矩阵进行存储和计算。

# Hessian Free

至此我们通过不断牺牲迭代速度来减小计算量和存储空间，而$Hessian Free$方法则是很聪明的一种处理方式可以让我们摆脱$Hessian$矩阵。  
通过观察共轭梯度法的算法流程，发现$Hessian$矩阵总是以**$matrix-vector \ products$**的形式出现的，即$Hv$的形式。先来看$Hessian​$矩阵的表达式：
{% asset_img hessian.jpg %}
有以下关系：
$$
(Hv)_i=\sum_{j=1}^N \frac{\partial ^2 f}{\partial x_i x_j}(x)\cdot v_j=\nabla \frac{\partial f}{\partial x_i}(x)\cdot v
$$
而这正是函数$g=\frac{\partial f}{\partial x_i}$对于方向$v$的方向导数，根据方向导数的定义式：
$$
\nabla _v g=\lim _{\varepsilon \to 0}\frac{g(x+\varepsilon v)-g(x)}{\varepsilon}\approx \frac{g(x+\varepsilon v)-g(x)}{\varepsilon}
$$
于是得到：
$$
Hv \approx \frac{\nabla f(x+\varepsilon v)-\nabla f(x)}{\varepsilon}
$$
经过两次梯度反向传播，即可得到关于$Hessian$矩阵的$matrix-vector \ products$

# Line search

我们需要求极值的函数为此构造的拉格朗日函数：
$$
L(\theta,\lambda)=g(\theta-\theta_{old})-\frac{\lambda}{2}\left[(\theta-\theta_{old})^TF(\theta-\theta_{old})-\delta \right] \\
constraint \ \frac{1}{2}s^TFs=\delta
$$
对每一次迭代，用采用$Hessian \ Free$处理方式的共轭梯度法，计算出由当前点指向当前极值点的向量$s_u=\frac{1}{\lambda}F^{-1}g$，为满足限制条件，需要对$s_u$进行修正：
$$
s=\sqrt{\frac{2\delta}{s_u^T F s_u}}s_u
$$
利用这一修正后的向量$s$进行线性搜索：  
分别以向量$s,\frac{s}{2},\frac{s}{4},\ldots$与当前迭代点$x_i$相加，直到优化目标$\max_\theta L_{\theta_{old}}(\theta)​$有所提升。
{% asset_img trpo.jpg %}

# 回顾

写出折扣期望回报$\eta(\theta)$的增量形式：
$$
\eta(\widetilde{\pi})=\eta(\pi)+\mathbb{E}_{s_0,a_0,\ldots \sim \widetilde{\pi}}\left[ \sum_{t=0}^{\infty}\gamma^t A_\pi(s_t,a_t) \right]
$$
写出$\eta(\theta)$的$local\ approximation$：
$$
L_\pi(\widetilde{\pi})=\eta(\pi)+  \sum_s  \rho_{\pi}(s) \sum_a \widetilde{\pi}(a \mid s) A_\pi(s,a)
$$
重要不等式：
$$
\eta(\pi_{new})\ge L_{\pi_{old}}(\pi_{new})-C\cdot D_{KL}^{max}(\pi_{old},\pi_{new})
$$
其中
$$
C=\frac{4\epsilon \gamma}{(1-\gamma)^2}
$$
{% asset_img minimax.PNG %}
找到目标函数：
$$
\max_\theta \left[  L_{\theta_{old}}(\theta)-C\cdot \bar{D}_{KL}(\theta_{old}\parallel \theta)\right]
$$
近似为二次型：
$$
\max_{\theta}g(\theta-\theta_{old})-\frac{C}{2}(\theta-\theta_{old})^TF(\theta-\theta_{old})
$$
sulotion:
$$
\theta-\theta_{old}=\frac{1}{C}F^{-1}{g}
$$
用共轭梯度法（Hessian Free）去计算$F^{-1}g$

over!

