# Diffusion LLM 的训练目标

本文档聚焦 Diffusion LLM 是怎么训练的。最核心的一点是：`Diffusion LLM` 不是某一个固定损失函数的名字，而是一族围绕“前向腐蚀过程 + 学习到的反向模型”构建出来的训练目标。

高层可以写成：

\[
\text{DLLM training}
=
\text{forward corruption}
+ \text{reverse prediction target}
+ \text{loss definition}
+ \text{sampling-time interpretation}.
\]

不同论文之间最主要的差异，通常就在这四个选择上。

## 1. 一般性的训练模板

记 \(x_0\) 为从数据分布采样到的干净文本序列。一个 diffusion 风格训练目标通常会先采样噪声级别 \(t\)，再构造一个噪声版本 \(x_t\)，然后训练模型去预测和 \(x_0\)、\(x_{t-1}\) 或某个 score function 相关的目标。

一般形式可以写成：

\[
x_0 \sim q(x_0), \qquad
t \sim p(t), \qquad
x_t \sim q(x_t \mid x_0),
\]

然后最小化

\[
\mathcal{L} =
\mathbb{E}_{x_0, t, x_t}
\left[
\ell_\theta(x_0, x_t, t)
\right].
\]

真正的关键问题是：\( \ell_\theta \) 到底该定义成什么。

## 2. 连续 Diffusion 的损失

连续 DLLM 通常工作在 embedding 或 latent 空间里。

### 2.1 噪声预测

如果前向过程是

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon, \qquad \epsilon \sim \mathcal{N}(0,I),
\]

那么一种常见目标就是预测注入的噪声：

\[
\mathcal{L}_{\epsilon}
=
\mathbb{E}_{z_0,\epsilon,t}
\left[
\|\epsilon - \epsilon_\theta(z_t,t)\|_2^2
\right].
\]

这是从图像 diffusion 继承过来的标准训练范式。

### 2.2 干净状态预测

除了预测噪声，模型也可以直接预测干净 latent：

\[
\mathcal{L}_{x_0}
=
\mathbb{E}_{z_0,\epsilon,t}
\left[
\|z_0 - \hat{z}_{0,\theta}(z_t,t)\|_2^2
\right].
\]

选择预测噪声还是预测干净状态，会影响：

- 优化稳定性；
- 反向过程的解释方式；
- 后续解码启发式的设计。

### 2.3 Self-Conditioning

有些连续模型会把上一次对干净状态的估计反馈回模型输入：

\[
\hat{z}_0^{(t)} \rightarrow \text{later refinement steps 的模型输入}.
\]

它并没有重新定义前向过程，但会改变实际训练动力学，因为模型被训练成反复修正自己的猜测。

## 3. 离散 Diffusion 的目标

如果直接在 token 空间里建模，损失定义就会变化。

### 3.1 基于转移的学习

设前向过程是一条离散 Markov 链：

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

那么一个自然目标就是学习反向转移：

\[
p_\theta(x_{t-1} \mid x_t).
\]

此时训练目标通常会变成 token 转移上的 cross-entropy，或者由反向链隐含出的 clean-token reconstruction 项。

### 3.2 干净 Token 重建

一个常见的工程化简化是：直接从噪声 token 状态 \(x_t\) 去预测原始 token \(x_0\)：

\[
\mathcal{L}_{\mathrm{disc}}
=
\mathbb{E}_{x_0,t,x_t}
\left[
- \sum_i \log p_\theta(x_{0,i} \mid x_t, t)
\right].
\]

在实践里，往往还会只在被腐蚀的位置上计算这项损失。

这个目标很直接，但它能不能工作好，取决于腐蚀过程和参数化形式是否与后续解码方式匹配。

## 4. Masked Diffusion 的目标

Masked diffusion 是离散 diffusion 中特别重要的一条支线。

### 4.1 吸收态腐蚀

每个 token 会逐步被替换成一个特殊 mask token \(m\)：

\[
q(x_t \mid x_0)
=
\mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr).
\]

这意味着前向过程会保留一部分上下文，同时隐藏另一部分内容。

### 4.2 Masked 重建损失

最常见的 masked diffusion 目标是：

\[
\mathcal{L}_{\mathrm{mask}}
=
\mathbb{E}_{x_0,t,x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right],
\]

其中 \(M_t\) 是噪声级别 \(t\) 下被 mask 的位置集合。

像 MDLM、LLaDA 这类方法，核心都建立在这种目标上。

### 4.3 不同噪声级别的加权

并不是所有 timestep 的训练贡献都一样。有些工作会显式或隐式地对不同 \(t\) 做加权：

\[
\mathcal{L}
=
\mathbb{E}_{x_0,t,x_t}
\left[
w(t)\cdot
\left(
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right)
\right].
\]

这很重要，因为：

- 低噪声样本更偏向学局部修复；
- 高噪声样本更偏向学在弱上下文下做全局生成。

所以 \(w(t)\) 的设计会直接影响模型更擅长哪种行为。

## 5. Likelihood 导向目标

有些 DLLM 工作比起去噪质量，更强调 likelihood 本身。

### 5.1 反向链的 Likelihood 视角

设模型定义

\[
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1}\mid x_t).
\]

则有

\[
p_\theta(x_0) = \sum_{x_{1:T}} p_\theta(x_{0:T}).
\]

这个边缘量通常不可直接计算，所以训练时会优化某个上界或替代目标。

### 5.2 ELBO 风格目标

一个标准的变分形式是：

\[
\mathcal{L}_{\mathrm{ELBO}}
=
\mathbb{E}_{q(x_{1:T}\mid x_0)}
\left[
\log \frac{q(x_{1:T}\mid x_0)}{p_\theta(x_{0:T})}
\right].
\]

等价地也可以写成

\[
-\log p_\theta(x_0)
\le
\mathcal{L}_{\mathrm{ELBO}}.
\]

这个解释很重要，因为它把 diffusion 训练和经典 likelihood-based language modeling 联系了起来，而不只是某种去噪启发式。

### 5.3 为什么语言建模很在意这一点

对语言模型来说，likelihood 很重要，因为它和下面这些东西直接相关：

- perplexity；
- scaling law 分析；
- calibration；
- 与自回归模型的原则性比较。

所以即便最终用户更关心生成质量，likelihood 导向的 DLLM 研究在概念上仍然很关键。

## 6. 离散空间里的 Score 目标

离散 diffusion 不能直接无缝照搬连续空间里的 score matching。

### 6.1 基于比值的 Score

SEDD 提出了一个离散状态上的 score 定义，用的是概率比值：

\[
s(x,t)_y = \frac{q_t(y)}{q_t(x)}.
\]

模型学习去逼近

\[
s_\theta(x,t)_y \approx s(x,t)_y.
\]

### 6.2 为什么这一步重要

这种写法不是强行把连续 score machinery 套到离散空间里，而是使用了一个更贴合离散分布本身的对象。

这会同时改变：

- 训练目标；
- 优化问题的几何结构。

概念上可以理解为：

\[
\text{更适合离散空间的目标}
\Rightarrow
\text{更好的反向过程学习}.
\]

## 7. 条件生成与 Seq2Seq 目标

并不是所有 DLLM 都是为无条件生成训练的。

### 7.1 条件 Diffusion

给定条件输入 \(c\)，目标会变成：

\[
\mathcal{L}_{\mathrm{cond}}
=
\mathbb{E}_{x_0,c,t,x_t}
\left[
\ell_\theta(x_0, x_t, c, t)
\right].
\]

这覆盖了：

- 摘要生成；
- 机器翻译；
- 改写；
- 问题生成；
- 对话生成。

### 7.2 Encoder-Decoder Diffusion

在 DiffuSeq、SeqDiffuSeq 这类模型里，条件输入 \(c\) 先由 encoder 编码，输出序列则由 diffusion decoder 去噪恢复。

因此训练目标不再只是“恢复干净文本”，而是：

\[
\text{在 source input 条件下恢复干净 target text}.
\]

这使 DLLM 更接近传统 seq2seq 学习，而不只是纯语言建模。

## 8. 后训练与 Instruction Finetuning

现代大型 DLLM 通常不会只看预训练损失。

### 8.1 监督微调

给定 instruction-response 对 \((u, y)\)，可以继续用 masked diffusion 方式训练：

\[
\mathcal{L}_{\mathrm{SFT}}
=
\mathbb{E}_{u,y,t,y_t}
\left[
- \sum_{i \in M_t} \log p_\theta(y_i \mid y_t, u, t)
\right].
\]

这可以看成是 diffusion 版本的 instruction tuning。

### 8.2 为什么 SFT 会改变问题本身

预训练回答的是：模型能不能在原始文本上学到一个好的反向恢复过程。

而 SFT 问的是另一个问题：

\[
\text{这个反向过程能不能进一步变成“会做任务”的过程？}
\]

对 LLaDA 一类模型来说，这个转变非常关键。

## 9. 实践里最重要的差异在哪里

当研究者比较 DLLM 的训练目标时，最重要的差异通常集中在：

1. 预测的到底是什么：噪声、干净 latent、干净 token，还是离散 score；
2. 用的是什么腐蚀家族：高斯、结构化离散，还是 absorbing-state mask；
3. timestep 是怎么采样和加权的；
4. 模型是无条件、条件生成，还是 instruction-tuned；
5. 训练目标和后续打算使用的解码方式是否匹配。

最后一点尤其重要：

\[
\text{good training loss}
\neq
\text{good generation quality}
\]

除非这个损失真正把模型训练成了适合其实际 decoding 场景的样子。

## 10. 一个实用的阅读路径

如果想循序渐进理解 DLLM 的训练目标，可以按下面顺序读：

1. `Diffusion-LM`：看连续 latent denoising；
2. `D3PM`：看离散状态腐蚀；
3. `Likelihood-Based Diffusion Language Models`：看 ELBO / likelihood 强调；
4. `SEDD`：看离散 score-entropy 训练；
5. `MDLM`：看简化后的 masked diffusion 目标；
6. `LLaDA`：看大规模 masked diffusion + instruction finetuning。

## 11. 最小总结

如果只保留一句最有用的话，可以记成：

\[
\text{DLLM objective}
=
\text{learn how to reverse text corruption}.
\]

但这个方向真正分化的地方在于：

- “腐蚀”到底是什么意思；
- 反向模型在预测什么；
- likelihood 怎么解释；
- 训练好的模型之后打算如何被解码使用。

这些选择，才真正把不同 DLLM 路线区分开来。
