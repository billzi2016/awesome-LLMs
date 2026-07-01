# Diffusion LLM 的方法谱系图

本文档的目的不是简单列论文，而是把主要 Diffusion LLM 方法整理成几个相对稳定的方法族，并说明它们到底为什么不同。

高层来看，大多数 DLLM 方法都可以用四个问题来分类：

1. 扩散发生在什么空间里：连续 latent、离散 token，还是 masked token 空间？
2. 训练目标是什么：噪声、干净状态、反向转移，还是离散 score？
3. 解码方式是什么：全并行、迭代重掩码、半自回归，还是带位置偏置的 refinement？
4. 主要使用场景是什么：无条件语言建模、可控生成、seq2seq 生成，还是大规模 instruction following？

## 1. 一句话总图景

一个有用的总图景可以写成：

\[
\text{DLLM landscape}
=
\text{continuous family}
+ \text{discrete family}
+ \text{masked family}
+ \text{conditional / seq2seq family}
+ \text{hybrid decoding family}.
\]

这些 family 之间并不是完全互斥的，但它们依然有意义，因为每一类都做出了不同的核心设计选择。

## 2. 连续 Latent 空间方法族

这一族方法不是直接在 token 上做 diffusion，而是在 embedding 或 latent 向量上做 diffusion。

### 2.1 主要代表

- `Diffusion-LM`
- `Self-conditioned Embedding Diffusion`
- `GENIE`
- `Latent Diffusion for Language Generation`

### 2.2 核心设计

干净对象表示为 \(z_0 \in \mathbb{R}^d\)，腐蚀过程通常是高斯型：

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon.
\]

然后模型去预测：

- 注入的噪声；
- 干净 latent 状态；
- 或连续空间中的其他去噪目标。

### 2.3 优点

- 数学上和经典图像 diffusion 更接近；
- 优化工具比较成熟；
- 在 latent 空间里做 controllable generation 较自然。

### 2.4 缺点

- 连续去噪和离散文本输出之间存在错位；
- 最终投回 token 的过程可能比较脆弱；
- 想把语言建模能力做得很强，通常没有 masked-token 路线那么直接。

### 2.5 什么时候把论文放进这一类

如果一篇论文：

- 特别强调 latent geometry；
- 大量借鉴 image-style diffusion；
- 通过 latent optimization 讨论 controllability；

那它通常属于这一族。

## 3. 一般性离散 Diffusion 方法族

这一族方法直接在 token 空间里建模，而不是在连续 latent 空间里扩散。

### 3.1 主要代表

- `D3PM`
- `SEDD`
- `A Reparameterized Discrete Diffusion Model for Text Generation`

### 3.2 核心设计

前向过程是一条 token 级别的 Markov 链：

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

反向模型学习的可能是：

- token 转移分布；
- 干净 token 重建；
- 或离散 score 一类对象。

### 3.3 为什么这一族重要

它消除了“连续去噪 vs 离散文本”的错位，更原生地处理文本 token。

但它的核心难点是：离散 diffusion 往往比高斯 diffusion 更难优化。这也是 SEDD 这类方法重要的原因，因为它们改进的是训练目标本身。

### 3.4 它和 Masked Diffusion 的区别

Masked diffusion 是离散 diffusion 的一个特例，但不是全部。

一般性离散 diffusion 还可以：

- 把 token 替换成结构化候选；
- 使用更丰富的转移矩阵；
- 不把腐蚀过程只压缩成单一的 absorbing mask state。

## 4. Masked Diffusion 方法族

这是当前大规模 DLLM 中最重要的一族。

### 4.1 主要代表

- `DiffusionBERT`
- `MDLM`
- `LLaDA`

### 4.2 核心设计

前向过程逐步把 token 替换成 `[MASK]`：

\[
q(x_t \mid x_0)
=
\mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr).
\]

模型在 masked 位置上预测原始 token：

\[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t).
\]

### 4.3 为什么它后来变成主流

Masked diffusion 吸引人的地方在于：

- 它始终工作在 token 空间里；
- 和 bidirectional Transformer 很自然匹配；
- 支持并行细化；
- 和 infilling、editing、revision 这些能力天然相容。

### 4.4 这一族内部也有分化

即便都属于 masked diffusion，不同论文之间也会在这些地方分叉：

- timestep 加权；
- 解码日程；
- remasking 策略；
- scaling recipe；
- 后训练策略。

例如：

- `DiffusionBERT` 更像是在 generative masked modeling 和 diffusion 之间搭桥；
- `MDLM` 更强调“简化但有效”的 masked diffusion 目标；
- `LLaDA` 则把 masked diffusion 推到了大规模预训练和 instruction tuning。

### 4.5 什么时候优先把论文归到这一类

如果一个方法：

- 从全 mask 或高 mask 状态开始；
- 依赖 iterative unmasking；
- 主要目标是 masked reconstruction；
- 声称具备大模型式的语言能力；

那默认就优先放进这一族。

## 5. 条件生成与 Seq2Seq Diffusion 方法族

这一族方法聚焦于“给定输入，生成目标文本”。

### 5.1 主要代表

- `DiffuSeq`
- `SeqDiffuSeq`
- `DINOISER`
- `Diffusion-NAT`

### 5.2 核心设计

这里的训练问题不再只是

\[
\text{generate text}.
\]

而是

\[
\text{generate target text conditioned on source text}.
\]

因此模型通常会包含：

- 编码条件输入 \(c\) 的 encoder；
- 对输出序列做 diffusion 去噪的 decoder。

### 5.3 这一族的特殊性

和无条件 DLLM 相比，这类方法和传统 NLP 任务更紧密相关：

- 摘要生成；
- 机器翻译；
- 对话生成；
- 改写；
- 问题生成。

### 5.4 它在整个版图里的位置

这一族的重点不是和 open-ended continuation 的 next-token LLM 直接竞争，而是尝试用 diffusion 风格 decoding 去替代或扩展 seq2seq generation。

## 6. 半自回归与混合解码方法族

有些方法既不属于纯并行 diffusion，也不属于标准 AR generation。

### 6.1 主要代表

- `SSD-LM`
- `AR-Diffusion`

### 6.2 核心设计

这类方法会显式把“顺序结构”重新注入 diffusion。

例如：

- `SSD-LM` 使用 simplex-based diffusion，并配合半自回归分块生成；
- `AR-Diffusion` 为不同位置分配不同的有效去噪日程。

### 6.3 为什么会有这一族

纯并行 diffusion 在计算上很有吸引力，但语言并不像图像那样天然适合完全同步生成。token 顺序很重要。

所以这一族真正问的是：

\[
\text{能不能一边保留 diffusion 风格 refinement，一边恢复部分顺序偏置？}
\]

### 6.4 主要权衡

这一族通常会牺牲一部分纯并行性，换来：

- 更好的连贯性；
- 更强的左到右依赖建模；
- 更可控的生成顺序。

## 7. 逐个方法怎么放

下面给一个更紧凑的放置图。

### 7.1 Diffusion-LM

- 方法族：连续 latent 空间
- 主要想法：在词向量式 latent 表示上做连续 diffusion
- 最适合的理解：偏 controllable generation

### 7.2 GENIE

- 方法族：连续 latent 空间
- 主要想法：连续段落去噪的预训练 diffusion LM
- 最适合的理解：偏预训练导向的 latent diffusion

### 7.3 Latent Diffusion for Language Generation

- 方法族：连续 latent 空间
- 主要想法：在 autoencoded language latents 上做 diffusion
- 最适合的理解：latent compression + diffusion

### 7.4 D3PM

- 方法族：一般性离散 diffusion
- 主要想法：结构化离散状态腐蚀
- 最适合的理解：基础离散框架

### 7.5 SEDD

- 方法族：一般性离散 diffusion
- 主要想法：为离散 diffusion 设计 score-entropy 目标
- 最适合的理解：token-space diffusion 的目标改进

### 7.6 DiffusionBERT

- 方法族：masked diffusion
- 主要想法：用 BERT 式 generative masking 加上 diffusion 解释
- 最适合的理解：MLM 风格建模与 generative diffusion 之间的桥梁

### 7.7 MDLM

- 方法族：masked diffusion
- 主要想法：简化但效果强的 masked diffusion 目标
- 最适合的理解：实用型 masked diffusion baseline

### 7.8 LLaDA

- 方法族：masked diffusion
- 主要想法：大规模 masked diffusion LLM + SFT
- 最适合的理解：现代大规模 DLLM 系统

### 7.9 DiffuSeq / SeqDiffuSeq

- 方法族：条件生成与 seq2seq diffusion
- 主要想法：给 source-conditioned target generation 配 diffusion decoder
- 最适合的理解：seq2seq diffusion 路线

### 7.10 DINOISER / Diffusion-NAT

- 方法族：条件生成与 seq2seq diffusion
- 主要想法：通过噪声控制或 self-prompted denoising 改进条件生成
- 最适合的理解：偏任务型的 conditional diffusion

### 7.11 SSD-LM

- 方法族：半自回归与混合解码
- 主要想法：simplex diffusion + 模块化分块生成
- 最适合的理解：diffusion 与 semi-AR generation 的混合体

### 7.12 AR-Diffusion

- 方法族：半自回归与混合解码
- 主要想法：位置依赖的去噪顺序
- 最适合的理解：显式带顺序偏置的 diffusion

## 8. 用更结构化的话比较这些方法族

可以用下面这种方式快速对比：

- 连续 family：和经典 diffusion 数学联系最强，但和 token 离散性对齐最弱。
- 离散 family：和 token 概率建模最对齐，但优化通常最困难。
- masked family：目前最适合大规模、双向 Transformer 风格 DLLM 的实践路线。
- seq2seq family：最适合 source-conditioned NLP 任务。
- hybrid decoding family：适合想保留 diffusion 灵活性、又不想完全放弃生成顺序结构的场景。

## 9. 如何给一篇新论文分类

遇到一篇新的 DLLM 论文时，可以按这个顺序判断：

1. 它腐蚀的是什么空间？
2. 它的腐蚀是一般性离散、absorbing mask，还是连续高斯？
3. 它做的是无条件 LM、条件生成，还是 instruction following？
4. 它的解码是全并行、基于 remasking、分块式，还是带位置偏置？
5. 它的创新主要在 loss、腐蚀过程、scaling recipe，还是 decoder？

通常这五个问题就足够把一篇论文定位到方法谱系图里。

## 10. 最小总结

最短但有用的一句话是：

\[
\text{大多数 DLLM 论文的真正差异，更多在训练空间和解码设计上，而不是名字本身。}
\]

所以把方法整理成 family，会比把它们平铺成论文列表更有信息量。
