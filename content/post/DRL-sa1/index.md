---
title: 深度强化学习（DRL）算法 1
description: REINFORCE
slug: hello-world
date: 2023-01-09 00:00:00+0000
image: cover.jpg
categories:
    - RL
tags:
    - DRL
    - single agent
---

<script
 src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
 type="text/javascript">
</script>

>“养成习惯的过程可以分为四个简单的步骤：提示、渴求、反应和奖励。”
>
>摘录来自《掌控习惯》詹姆斯·克利尔(James Clear) 此材料可能受版权保护。

# 前言

就像引言里所描述的养成习惯的四个步骤，如果我们想让机器也有自己的“习惯”，去掉机器没有的渴求属性，就是强化学习所做的事情 —— 帮机器养成“习惯”，而 DRL 就是使用深度学习的技术去实现强化学习算法。今天是系列文章的第一篇，会介绍最基础的 policy-based 的算法 —— REINFORCE。

τ = {s0, a0, r1, s1, a1, r2, s2, a2 ... rT, sT, aT}, 那么根据 MDP 属性有:

$$ p(\tau) = \prod_{t=0}^{T}p_{\theta}(a_{t}|s_{t})p(s_{t+1}|s_{t}, a_{t}) $$

那么这里的 pθ 就是机器的策略，针对 s 会产生什么样的 a。因为当 s 和 a 都确定了，下一时刻的 s 是系统的固有属性，我们没办法控制，所以我们后面只关心 pθ 。
因为每一时刻的反应都会产生相应的回报，那么我们把每一时刻的回报加起来，就是这一系列反应产生的整体回报，我们用 R(τ) 表示。有了整体回报，我们就可以衡量机器行为的好坏。

$$R(\tau) = \sum_{t=1}^{T}r_{t}$$

机器应该更关心当下的回报还是未来的回报呢？我们引入系数 γ ，如果它是 0，我只关注当下，它的值越大，我们会越关心未来的回报。

$$R_{\theta} = E_{\tau\sim p_{\theta}(\tau)}[R(\tau)] =  \sum_{\tau}p_{\theta}(\tau)R(\tau)$$

有了上面的信息，接下来我们只要使得机器的期望回报最大，我们的机器就会牛逼起来！

$$R_{\theta} = E_{\tau\sim p_{\theta}(\tau)}[R(\tau)] =  \sum_{\tau}p_{\theta}(\tau)R(\tau)$$

# 如何获得最大的期望回报？

由上面的期望回报的公式，我们可以通过调节策略增大可以获得更大回报的反应的概率，可以用深度神经网络表示策略，把 s 输入神经网络（可以是MLP\CNN\LSTM\Transformer...）获得的输出就是每个反应的概率。s 输入到神经网络，得到 a，这里随机采样一个 a（为了探索，选最大的不一定是正确的） 然后得到下一时刻的 s，重复收集，就有了上面公式里的 τ，从而就积累了训练数据。那么有过机器学习经验的人，可以发现答案呼之欲出，就是求导，梯度下降是求最小值，那么求最大值就是加个负号。

$$\theta \leftarrow \theta + \alpha \nabla \bar{R}(\theta)$$

其中：

$$
\begin{aligned}
\nabla \bar{R}(\theta) & =\nabla_\theta \sum_\tau p(\tau) R(\tau) \\
& =\sum_\tau \nabla_\theta p(\tau) R(\tau) \\
& =\sum_\tau p(\tau) \frac{\nabla_\theta p(\tau)}{p(\tau)} R(\tau) \quad\left(\text { 因为 } \nabla_\theta \log (z)=\frac{1}{z} \nabla_\theta z\right) \\
& =\sum_\tau p(\tau) \nabla_\theta \log p(\tau) R(\tau) \\
& \approx \frac{1}{m} \sum_{i=1}^m \nabla_\theta \log p\left(\tau^{(i)}\right) R\left(\tau^{(i)}\right) \\
& =\frac{1}{m} \sum_{i=1}^m \sum_{t=1}^T \nabla_\theta \log p_\theta\left(a_t^{(i)} \mid s_t^{(i)}\right) R\left(\tau^{(i)}\right) \\
& =\frac{1}{m} \sum_{i=1}^m R\left(\tau^{(i)}\right) \sum_{t=1}^T \nabla_\theta \log p_\theta\left(a_t^{(i)} \mid s_t^{(i)}\right)
\end{aligned}
$$

# 实现
实现很简单，这里结合参考很快就可以看懂。

# 缺点
方法很简单，虽然说大道至简，但是用上述方法训练出来的机器还是不够牛逼。主要是因为方法里存在一些缺陷：
1. 策略产生的 τ 不能再次用于训练，因为策略一直在更新，策略更新了，那么更新之前产生的 τ 就得丢掉了，训练效率很低。
2. τ 里的每个 a 用的都是同样的 R，更好的情况应该是每个 a 用一个 R。

# 改进

- 针对 1，下一篇文章介绍的 PPO 提出了改进
- 针对 2：
我们可以这样设置 R（一种 Credit Assignment）：
$$R_{t}(\tau) = \sum_{t=t(a)}^{T}\gamma^{t-t(a)}r_{t},\gamma \in [0,1]$$

# 参考
1. 深度强化学习——李宏毅 https://www.bilibili.com/video/BV1MW411w79n/?spm_id_from=333.337.search-card.all.click&vd_source=880886f42f0b8f6ed75679ffe50b5384
2. Deep Reinforcement Learning: Pong from Pixels http://karpathy.github.io/2016/05/31/rl/
3. https://github.com/pytorch/examples/blob/main/reinforcement_learning/reinforce.py
4. https://colab.research.google.com/github/huggingface/deep-rl-class/blob/main/unit5/unit5.ipynb
5. What does log_prob do? https://stackoverflow.com/questions/54635355/what-does-log-prob-do