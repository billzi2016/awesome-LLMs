# Diffusion LLM 的发展时间线

本文档按时间顺序概括 Diffusion LLM 是怎么发展起来的。重点不是把论文全列一遍，而是把这个方向在 formulation、训练目标、规模化和解码方式上的关键转折点梳理出来。

## 1. 在 DLLM 变成一个明确方向之前

在 “Diffusion LLM” 成为相对明确的标签之前，背景主要来自两条线：

- 图像等连续域中的 diffusion 模型；
- NLP 里的非自回归生成和 masked 风格文本生成。

当时最核心、也最未解决的问题是：

\[
\text{how can diffusion-style generation be adapted to discrete language?}
\]

后面整条时间线，其实都在围绕这个问题展开。

## 2. 2021：离散 Diffusion 在技术上变得可行

### 2.1 D3PM

**Structured Denoising Diffusion Models in Discrete State-Spaces**（Austin 等，NeurIPS 2021）是这里最关键的里程碑。

它重要的原因在于：

- 它给了 diffusion 一个比较系统的离散状态 formulation；
- 它说明腐蚀过程不一定非得是高斯噪声；
- 它为 token-level diffusion 建模打下了基础。

这篇论文最适合被理解为：从 image-style diffusion 走向 token-space diffusion 的技术桥梁。

## 3. 2022：面向语言的早期 Diffusion Formulation 出现

2022 年是 diffusion for language 从“借来的想法”变成“正在形成子领域”的一年。

### 3.1 连续与 Latent 路线

关键论文：

- `Diffusion-LM Improves Controllable Text Generation`
- `Self-conditioned Embedding Diffusion for Text Generation`
- `GENIE`
- `Latent Diffusion for Language Generation`

这一年发生的变化是：

- 文本生成被明确写成 diffusion 问题；
- latent-space 和 embedding-space denoising 成为早期主流写法；
- controllable generation 成为一个非常重要的动机。

这一阶段的整体思路仍然比较接近图像 diffusion：

\[
\text{continuous denoising first, language adaptation second}.
\]

### 3.2 条件文本生成路线

关键论文：

- `DiffuSeq`
- `DiffusER`
- `SSD-LM`

这一年进一步发生的变化是：

- diffusion 开始被用于 seq2seq 和条件生成任务；
- revision、editing、多样性成为明确卖点；
- 一些方法开始把 diffusion refinement 和结构化生成顺序混合起来。

也是从这里开始，这个方向逐渐分成了两条明显支线：

- 面向开放式语言建模；
- 面向任务型条件生成。

## 4. 2023：训练视角更清楚，混合设计更多

到 2023 年，这个方向不再只是“尝试能不能做”，而是开始更自觉地反思方法本身。

### 4.1 Likelihood 与 Scaling

关键论文：

- `Likelihood-Based Diffusion Language Models`

这一年发生的变化是：

- diffusion language modeling 开始从 likelihood 角度被分析，而不是只看 sample quality；
- 研究者开始更认真地讨论 scaling behavior；
- 和经典 LM 指标之间的连接变得更严肃。

可以说，从这里开始 DLLM 真正在问：

\[
\text{can diffusion be a real language-modeling alternative, not just a controllable-generation trick?}
\]

### 4.2 更成熟的条件 Diffusion

关键论文：

- `SeqDiffuSeq`
- `DINOISER`
- `Diffusion-NAT`

这一年发生的变化是：

- 条件生成设置变得更精细；
- 噪声日程和 decoder 设计更贴近任务本身；
- diffusion 开始更直接地和强 seq2seq baseline 竞争。

### 4.3 混合式与 AR-Biased 设计

关键论文：

- `AR-Diffusion`

这一年另一个重要变化是：

- 研究者开始意识到完全同步生成未必足够尊重语言顺序结构；
- 一些方法开始把左到右偏置重新注入 diffusion 过程。

这代表了一个重要转向：

\[
\text{pure parallelism}
\rightarrow
\text{parallelism with structural bias}.
\]

## 5. 2024：离散与 Masked Diffusion 显著变强

这是一个非常关键的转折年。

### 5.1 SEDD

关键论文：

- `Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution`

这一年发生的变化是：

- 离散 diffusion 的训练终于变得更有竞争力；
- 研究不再停留在简单 token reconstruction；
- objective design 本身成了核心问题。

SEDD 的意义在于，它说明瓶颈不只是“diffusion 对语言太难”，还包括“离散空间里如果目标设计不对，会显得它比实际更差”。

### 5.2 MDLM

关键论文：

- `Simple and Effective Masked Diffusion Language Models`

这一年进一步发生的变化是：

- masked diffusion 成为一条非常强的实践路线；
- 简化后的 absorbing-state 目标比很多人预期中更能 scale；
- 解码效率和半自回归变体开始获得更多关注。

到这个阶段，领域问题已经从“masked diffusion 只是更像 BERT 吗？”转向了“masked diffusion 能不能真的支撑大规模语言建模？”。

## 6. 2025：大规模 DLLM 开始变得可信

### 6.1 LLaDA

关键论文：

- `Large Language Diffusion Models`

这一年发生的变化是：

- masked diffusion 被推进到大模型尺度；
- 监督微调进入 DLLM 主叙事；
- 评测方式越来越接近现代 LLM 评测。

这是一个非常有象征意义的节点：

\[
\text{DLLM}
\rightarrow
\text{not only a paper family, but a system-level model class}.
\]

## 7. 2026：Inference-Time Refinement 变成重要主题

到了 2026 年，这个方向不再只问“怎么把模型训练得更好”，还开始更系统地问“怎么把它解码得更好”。

### 7.1 更鲁棒的重掩码

例子：

- `CoRe: Context-Robust Remasking for Diffusion Language Models`

这一年发生的变化是：

- remasking policy 本身变成研究对象；
- token confidence 不再被默认视为足够；
- context sensitivity 开始进入解码设计。

### 7.2 连续化或 Flow 风格的解码视角

例子：

- `Masked Diffusion Decoding as x-Prediction Flow`

这一年进一步发生的变化是：

- 一些研究者开始重新思考“mask / 提交”这种二元决策是否太刚；
- 解码被重写成更平滑的迭代细化；
- 离散 diffusion decoding 和 flow-like 视角之间的边界开始变得有意思。

这代表了又一次转向：

\[
\text{training objective focus}
\rightarrow
\text{inference algorithm focus}.
\]

## 8. 整条历史线的主模式

如果把这条时间线压缩一下，大概可以概括成四步：

1. 离散 diffusion 先变得可写、可做；
2. 语言专属 formulation 逐渐出现；
3. objective design 明显变强；
4. 大规模 masked DLLM 和 inference-time 方法变成主轴。

压缩成一句式子就是：

\[
\text{discrete foundation}
\rightarrow
\text{language adaptation}
\rightarrow
\text{objective improvement}
\rightarrow
\text{scaling and decoding refinement}.
\]

## 9. 怎么使用这份时间线

这份文档适合用来回答下面这些问题：

- 哪些论文是 foundational，哪些是后续 refinement？
- 这个方向大概在什么时候从 latent diffusion 转向 masked diffusion？
- scaling 是从什么时候开始真正严肃起来的？
- decoding 是从什么时候变成独立研究主题的？

它尤其适合配合下面几篇一起看：

- [method-family-map.zh-CN.md](./method-family-map.zh-CN.md)
- [training-objectives.zh-CN.md](./training-objectives.zh-CN.md)
- [sampling-and-decoding.zh-CN.md](./sampling-and-decoding.zh-CN.md)

## 10. 最小总结

最短但有用的一句话是：

\[
\text{DLLM 的演化，是从“diffusion 能不能处理文本”走向“哪种训练与解码设计最适合语言规模化”。}
\]
