# Diffusion LLM 的数学基础

本文档对 Diffusion LLM（DLLM）做一个尽量紧凑但公式明确的数学说明，重点解释连续 diffusion、离散 diffusion、masked diffusion 与自回归语言建模之间的关系。

## 1. 为什么 DLLM 需要单独讲数学

Diffusion 模型最早主要在图像等连续域中发展，因为图像天然可以表示成 \( \mathbb{R}^d \) 中的向量，标准高斯扰动也因此有直接定义。

但语言建模不同：

- 文本是离散的，不是连续的；
- 输出空间是组合式的 token 序列；
- embedding 空间里的“小扰动”不一定对应合法文本；
- 生成质量强依赖长程符号结构。

因此，DLLM 很快分化出几类不同数学路线：

- 在 embedding 或 latent 向量上做连续 diffusion；
- 直接在 token 上做离散 diffusion；
- 用 `[MASK]` 逐步腐蚀文本的 masked / absorbing-state diffusion；
- 保留 diffusion 采样方式，但借用语言模型训练或解码结构的混合方法。

## 2. 自回归语言模型作为基线

标准自回归语言模型把序列概率分解为

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t}).
\]

训练目标是最小化负对数似然

\[
\mathcal{L}_{\mathrm{AR}} = - \sum_{t=1}^{T} \log p_\theta(x_t \mid x_{<t}).
\]

这种写法带来：

- 精确的左到右概率分解；
- 自然的 next-token decoding；
- 容易计算 likelihood。

但它也带来：

- 生成必须串行；
- 左到右暴露偏差；
- 难以对任意位置做联合修订。

DLLM 的核心思想，就是不再坚持左到右分解，而是学习如何逆转一个“腐蚀过程”。

## 3. 一般性的 Diffusion 视角

记 \(x_0\) 为从数据分布 \(q(x_0)\) 采样得到的干净样本。

Diffusion 模型定义两部分：

1. 一个前向腐蚀过程，把 \(x_0\) 逐渐变成更噪声的状态 \(x_t\)；
2. 一个反向模型，学习如何把噪声状态逐步恢复回干净数据。

抽象地写就是：

\[
x_0 \rightarrow x_t \rightarrow x_T,
\]

其中 \(x_T\) 通常接近一个简单先验或完全腐蚀状态。

学习目标是近似反向转移：

\[
p_\theta(x_{t-1} \mid x_t).
\]

采样时则从 \(x_T\) 出发，反复应用学到的反向过程。

## 4. 面向语言的连续 Diffusion

连续 DLLM 通常先把 token 映射到连续 embedding 空间，再在该空间施加高斯式腐蚀。

### 4.1 前向过程

对连续表示 \(z_0 \in \mathbb{R}^d\)，一种常见的 variance-preserving 前向过程可写为

\[
q(z_t \mid z_0) = \mathcal{N}\bigl(z_t; \alpha_t z_0, \sigma_t^2 I\bigr),
\]

等价地也可写为

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon, \qquad \epsilon \sim \mathcal{N}(0, I).
\]

这里：

- \( \alpha_t \) 控制原始信号保留多少；
- \( \sigma_t \) 控制噪声强度；
- \(t\) 越大，状态越噪。

### 4.2 反向模型

模型通常学习预测以下之一：

- 干净 latent \(z_0\)；
- 噪声 \( \epsilon \)；
- 或一个 score / denoising direction。

常见的简化训练目标是

\[
\mathcal{L}_{\mathrm{cont}} = \mathbb{E}_{z_0, \epsilon, t}
\left[
\lVert \epsilon - \epsilon_\theta(z_t, t) \rVert_2^2
\right].
\]

在文本领域，这类连续 formulation 常见于：

- `Diffusion-LM`；
- `Self-conditioned Embedding Diffusion`；
- `GENIE` 及相关 latent denoising 方法。

### 4.3 语言里的核心张力

连续 formulation 数学上很顺手，但语言本体是离散的，所以模型最终还是要把连续去噪状态投回 token。这里会出现一个天然错位：

- diffusion 在 latent 空间里是平滑的；
- 文本合法性在 token 空间里是符号化、非连续的。

这也是为什么后续离散 diffusion 和 masked diffusion 变得非常重要。

## 5. 离散 Diffusion

离散 diffusion 不再在 \( \mathbb{R}^d \) 上扩散，而是直接在 token 状态上定义腐蚀过程。

设词表为 \( \mathcal{V} \)，每个位置的取值来自这个有限集合。

### 5.1 Markov 腐蚀

在 D3PM 一类方法中，前向过程由转移矩阵 \(Q_t\) 定义：

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

多步展开后有

\[
q(x_t \mid x_0)
= Q_t Q_{t-1} \cdots Q_1 x_0.
\]

矩阵 \(Q_t\) 决定 token 如何被腐蚀，例如：

- 均匀替换；
- 按某种结构化邻域替换；
- 推向一个特殊吸收态。

### 5.2 反向学习

反向模型的目标是学习

\[
p_\theta(x_{t-1} \mid x_t),
\]

或者学习一个足以构造该反向转移的参数化形式。

与连续 diffusion 相比，它的优势是与文本 token 更一致；但难点在于，离散 score matching 和 likelihood 估计会更棘手。

## 6. Masked / Absorbing-State Diffusion

很多现代 DLLM 使用 masked diffusion，因为它和语言建模更贴近。

### 6.1 吸收态

引入一个特殊 mask token \(m = [\mathrm{MASK}]\)。前向过程逐步把干净 token 替换成 \(m\)。

一种简单的边缘形式可写为：

\[
q(x_t \mid x_0) = \mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr),
\]

其中：

- \(x_0\) 是原始 token；
- \(m\) 是 one-hot 的 mask 分布；
- \(1-\alpha_t\) 是时刻 \(t\) 的 mask 概率。

当噪声较小时，大多数 token 仍可见；当噪声较大时，大多数 token 会变成 masked。

### 6.2 训练目标

模型看到一个部分 mask 的序列 \(x_t\)，然后在 masked 位置上预测原始 token：

\[
\mathcal{L}_{\mathrm{mask}}
=
\mathbb{E}_{x_0, t, x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right],
\]

其中 \(M_t\) 是时刻 \(t\) 被 mask 的位置集合。

这看起来和 masked language modeling 很像，但关键区别在于：

- BERT 通常使用固定或手工设计的 mask 比例；
- masked diffusion 会在连续或分阶段噪声日程上采样不同腐蚀强度。

正是这个区别，让 MDLM、LLaDA 这类方法可以把它解释为真正的生成式 diffusion 目标，而不是普通的去噪预训练。

### 6.3 为什么它更适合语言

Masked diffusion 有几个结构优势：

- 保留离散 token 语义；
- 支持很多位置并行预测；
- 自然支持 infilling 和 editing；
- 可以从全 mask 开始逐步 unmask 出完整文本。

这也是为什么近期很多大规模 DLLM 都转向 masked diffusion。

### 6.4 为什么 Masked Diffusion 不等于 BERT

表面上看，masked diffusion 的损失和 masked language modeling 很像，但它们在概率建模中的角色并不一样。

对 BERT 式 MLM，训练通常在某个固定 mask 设计下最小化

\[
\mathcal{L}_{\mathrm{MLM}}
=
\mathbb{E}_{x_0, \widetilde{x}}
\left[
- \sum_{i \in M} \log p_\theta(x_{0,i} \mid \widetilde{x})
\right].
\]

这本质上是一个条件去噪目标，但它本身并不自动定义一个完整序列的生成模型。

对 masked diffusion，噪声级别 \(t\) 是生成过程定义的一部分。模型是在整条腐蚀路径上训练的：

\[
t \sim p(t), \qquad x_t \sim q(x_t \mid x_0),
\]

并最小化

\[
\mathcal{L}_{\mathrm{MD}}
=
\mathbb{E}_{x_0, t, x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right].
\]

关键区别在于，\(t\) 会扫过从几乎干净到几乎全 mask 的整条腐蚀路径。模型学到的不是单个去噪任务，而是整条反向恢复过程。

因此更准确的关系是：

\[
\text{局部上像 MLM 的 token 预测损失}
\neq
\text{普通 MLM 训练目标}.
\]

在 DLLM 里，这个局部损失被绑定在全局 diffusion 采样过程上。

## 7. Likelihood 视角与 Score 视角

不同 DLLM 论文使用的数学视角并不相同。

### 7.1 Likelihood 导向视角

像 `Likelihood-Based Diffusion Language Models` 这样的工作强调，训练应当优化和序列 likelihood 相关的变分目标或上界。

高层写法可以理解为：

\[
-\log p_\theta(x_0) \leq \mathcal{L}_{\mathrm{diff}}(x_0),
\]

其中 \( \mathcal{L}_{\mathrm{diff}} \) 是某种 diffusion 训练目标，最小化它会改进与模型 likelihood 相关的一个可处理上界。

这很重要，因为语言建模通常不仅看样本质量，也看 perplexity 或 likelihood。

### 7.2 Score-Entropy 视角

SEDD 用边缘概率比值来重写离散 diffusion。离散 score 定义为

\[
s(x,t)_y = \frac{q_t(y)}{q_t(x)}.
\]

模型学习逼近这个比值：

\[
s_\theta(x,t)_y \approx \frac{q_t(y)}{q_t(x)}.
\]

然后 score-entropy 目标替代了不够稳定的离散 score matching，为离散空间提供了更合适的训练规则。

这一步很关键，因为它明显缩小了离散 diffusion 与自回归模型之间的效果差距。

### 7.3 变分上界的直观理解

更显式地说，diffusion 训练也可以用 latent-variable bound 来理解。设完整噪声轨迹为 \(x_{1:T}\)，则模型定义反向链

\[
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1}\mid x_t),
\]

前向过程定义

\[
q(x_{1:T}\mid x_0) = \prod_{t=1}^{T} q(x_t\mid x_{t-1}).
\]

利用标准变分推导，可以得到形如

\[
-\log p_\theta(x_0)
\le
\mathbb{E}_{q(x_{1:T}\mid x_0)}
\left[
-\log p(x_T)
- \sum_{t=1}^{T} \log p_\theta(x_{t-1}\mid x_t)
+ \sum_{t=1}^{T} \log q(x_t\mid x_{t-1})
\right]
\]

的上界。

这个式子不一定就是实践里逐字使用的训练目标，但它解释了整体逻辑：

- 前向链是固定的；
- 反向链是要学习的；
- 最小化 diffusion loss，本质上是在改进某个与序列 likelihood 相关的上界或变分替代目标。

对连续 diffusion，这个目标常常可化简成噪声预测损失；对 masked diffusion，则常常可化简成带权的 masked token 重建项。

## 8. DLLM 的采样过程

在推理阶段，DLLM 通常不是像标准自回归模型那样一次只生成一个 token。

它更像是在不断修正一个部分腐蚀的序列：

\[
x_T \rightarrow x_{T-1} \rightarrow \cdots \rightarrow x_0.
\]

在 masked diffusion 中，过程通常类似：

1. 从全 mask 序列开始；
2. 并行预测很多位置的 token；
3. 按需把低置信度位置重新 mask；
4. 经过若干轮去噪逐步细化。

这带来几个 DLLM 的典型性质：

- 并行更新 token；
- 更容易编辑；
- 更容易在任意位置条件化；
- 可以用去噪步数直接控制质量和计算量。

最后一点尤其关键：

\[
\text{更多去噪步数} \Rightarrow \text{更高计算成本，通常也带来更好质量}.
\]

因此，DLLM 比标准左到右 decoding 更显式地暴露出一个 compute-quality 调节旋钮。

### 8.1 置信度、重掩码与局部细化

很多 masked DLLM 并不是把所有 masked 位置填一次就结束，而是会反复细化。

记 \(x_t^{(i)}\) 为迭代 \(t\) 时第 \(i\) 个位置上的 token，记

\[
\pi_\theta(\cdot \mid x_t, t)
\]

为该时刻的预测分布。一个常见做法是：

1. 对 masked 位置预测 token；
2. 计算置信度，例如

\[
c_i = \max_{v \in \mathcal{V}} \pi_\theta(v \mid x_t, t);
\]

3. 保留高置信度位置；
4. 把低置信度位置重新 mask，再做下一轮去噪。

这样一来，实际的反向过程就是自适应的，而不是一次性固定完成。

它背后的直觉是：

\[
\text{容易的位置先稳定，困难的位置继续演化}.
\]

这也是 diffusion 风格生成和“一次性 masked filling”之间的一个重要实践差异。

### 8.2 为什么可以并行生成

在自回归 decoding 里，

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t})
\]

这个分解迫使采样阶段按 token 顺序串行进行。

而在 DLLM 中，模型条件在一个全局噪声状态 \(x_t\) 上，并联合预测很多位置：

\[
p_\theta(x_0 \mid x_t, t).
\]

这并不意味着各位置独立，而是说位置之间的依赖不再通过左到右链传递，而是通过共享的去噪状态传递。

因此并行生成之所以可能，是因为：

- 模型能看到整个部分腐蚀后的序列；
- 所有位置都可以通过双向注意力交换信息；
- 串行结构从“按 token 顺序生成”变成了“按 refinement 轮次生成”。

一句话概括就是：

\[
\text{AR LLM: serial tokens}
\qquad
\text{DLLM: serial refinements}.
\]

## 9. 如何统一理解代表性方法

仓库里的主要论文可以放进同一张数学地图中。

### 9.1 连续或 Latent 空间方法

- `Diffusion-LM`：在词向量式 latent 表示上做连续 diffusion，尤其强调 controllable generation。
- `Self-conditioned Embedding Diffusion`：embedding 空间 diffusion，并引入 self-conditioning。
- `GENIE`：偏预训练导向的 diffusion language modeling，使用 continuous paragraph denoising。

### 9.2 Seq2Seq 与条件 Diffusion

- `DiffuSeq`：把 diffusion 用到 seq2seq 条件文本生成。
- `DiffusER`：把文本生成写成 iterative edit-based reconstruction，特别适合 revision 和 prototype-conditioned generation。

### 9.3 离散与 Masked Diffusion

- `D3PM`：一般性的离散 diffusion 框架。
- `SEDD`：面向离散 diffusion 的 score-entropy 训练。
- `MDLM`：简化后的 masked diffusion 目标，实证效果很强。
- `LLaDA`：从头训练、再做 instruction tuning 的大规模 masked diffusion LLM。

### 9.4 带依赖结构的混合解码

- `AR-Diffusion`：把左到右依赖结构显式注入 diffusion 去噪过程。

因此可以用一句更统一的话总结：

\[
\text{DLLM} =
\text{腐蚀过程}
+ \text{反向去噪模型}
+ \text{采样日程}
+ \text{token / 表示空间选择}.
\]

不同论文主要是在这四个部件上做不同设计。

## 10. DLLM 与 AR LLM 的差异

它们的不同不只是架构不同，更是概率建模方式不同。

自回归模型定义的是：

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t}).
\]

Diffusion language model 定义的是：

\[
q(x_0) \xrightarrow{\text{前向腐蚀}} x_t
\xrightarrow{\text{学习到的反向过程}} p_\theta(x_0).
\]

落到实践里，通常意味着：

- AR LLM 更容易评测，也更容易直接解码；
- DLLM 提供更强的并行性，以及更自然的非左到右条件化方式；
- DLLM 更依赖噪声日程、remasking 策略和推理预算设计。

所以真正的研究问题不是 DLLM 是否只是模仿 AR LLM，而是 diffusion 风格生成能否形成一种有自己优势的竞争性替代路线。

### 10.1 两种不同的“时间”

理解 DLLM 时一个常见混淆点，是这里其实有两种不同的“时间”。

对自回归模型：

- 时间表示 token 位置。

对 diffusion 模型：

- 时间表示腐蚀强度或去噪步数。

它们不是同一个概念：

\[
\text{AR time} \neq \text{diffusion time}.
\]

这也是为什么 DLLM 可以在某一个去噪步同时生成很多位置，然后在后续去噪步再次回头修正这些位置。顺序性从“token 顺序”转移成了“refinement 顺序”。

### 10.2 AR-Diffusion 处在什么位置

AR-Diffusion 在概念上很有价值，因为它把上面两种时间结构部分混合了起来。它仍然是 diffusion 采样，但在不同 token 位置上注入了左到右的不对称性。

所以它可以理解为：

\[
\text{diffusion sampling}
+ \text{position-dependent causal bias}.
\]

这也解释了为什么 AR-Diffusion 既不是纯 masked DLLM，也不是标准 AR model。

## 11. 建议阅读顺序

如果想循序渐进理解这个方向，可以按下面顺序读：

1. 先读 `D3PM`，理解离散 diffusion 基础；
2. 再读 `Diffusion-LM` 和 `DiffuSeq`，看早期语言建模 formulation；
3. 再读 `Likelihood-Based Diffusion Language Models` 和 `SEDD`，理解 likelihood 与离散训练改进；
4. 再读 `MDLM`，理解 masked diffusion 的简化；
5. 最后读 `LLaDA`，看大规模 diffusion LLM 的训练与 instruction tuning。

## 12. 范围说明

本文档刻意偏解释性，而不是做完全形式化推导。目标是用尽量少但足够区分性的数学，把 DLLM 的几条主要路线讲清楚，并说明为什么近期 masked diffusion LLM 同时不同于 BERT 式 MLM 和标准自回归 LLM。

## 13. 最小心智模型

如果只保留一张最核心的数学图景，可以记这个：

\[
\text{clean text}
\xrightarrow{\text{corrupt}}
\text{noisy text}
\xrightarrow{\text{learned reverse refinement}}
\text{clean text}.
\]

之后拿任何一篇 DLLM 论文，都先问四个问题：

1. 表示空间是什么：embedding、token，还是 mask 状态？
2. 前向腐蚀规则是什么？
3. 反向模型到底在预测什么？
4. 采样是怎么做的：固定日程、remasking，还是混合解码？

通常把这四个问题问清楚，就足够把一篇新的 DLLM 方法放进整个版图里。
