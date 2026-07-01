# Diffusion LLM 术语表

这份术语表用来统一仓库里反复出现的核心概念。它不是要替代完整技术文档，而是为了保证不同说明之间的词汇含义尽量一致。

## A

### Absorbing State

吸收态。指在前向腐蚀过程中，一旦进入该状态，在同一腐蚀规则下就不会自动回到其他状态。对 masked diffusion 来说，`[MASK]` 往往就是吸收态。

### AR-Diffusion

一种混合式 diffusion 设计，在去噪过程中显式注入左到右偏置，使不同 token 位置在不同时间“浮现”出来。

## B

### Bidirectional Context

双向上下文。指模型在预测某个位置时，同时能看到左侧和右侧 token。masked diffusion 很自然地具备这一点，而严格因果的自回归解码不具备。

## C

### Clean State

干净状态。通常写作 \(x_0\) 或 \(z_0\)，表示未被腐蚀的数据点。

### Conditional Diffusion

条件 diffusion。指模型在某个输入 \(c\) 的条件下生成输出，比如 source sentence、prompt 或 instruction。

### Continuous Diffusion

连续 diffusion。指扩散过程定义在连续空间里，比如 embedding 或 latent 向量空间，通常配合高斯型腐蚀。

## D

### D3PM

`Structured Denoising Diffusion Models in Discrete State-Spaces` 的简称，是离散 diffusion 的基础框架之一。

### Decoding Budget

解码预算。指生成时使用的推理计算量，通常用去噪步数、模型调用次数、延迟等来衡量。

### Denoising

去噪。指把噪声状态逐步恢复到更干净状态的反向过程。

### Diffusion Time

diffusion 时间。指腐蚀 / 去噪索引 \(t\)。它和 token 位置不是一回事，不能和自回归生成顺序混为一谈。

### Discrete Diffusion

离散 diffusion。指扩散过程直接定义在有限 token 状态上，而不是定义在连续向量空间里。

## E

### ELBO

Evidence Lower Bound。对 DLLM 来说，它常被用来解释 diffusion 训练如何作为 likelihood-based 建模的变分替代目标。

## F

### Forward Corruption Process

前向腐蚀过程。指把干净数据 \(x_0\) 逐步映射成噪声状态 \(x_t\) 的过程。

## I

### Instruction Finetuning

指令微调。指在 instruction-response 数据上继续训练，让模型学会任务跟随，而不只是一般文本建模。

### Iterative Refinement

迭代细化。指模型不是一次性生成完整结果，而是反复修订一个部分生成的序列。

## L

### LLaDA

`Large Language Diffusion Models` 的简称，是把 masked diffusion 推向大规模 LLM 体系的重要路线之一。

### Likelihood-Based Diffusion

likelihood 导向的 diffusion 视角。强调 diffusion 训练和 likelihood、变分上界或传统语言建模指标之间的关系。

## M

### Masked Diffusion

masked diffusion。指一种离散 diffusion 形式，其中腐蚀过程会逐步把 token 替换成 `[MASK]`，生成时则通过 iterative unmasking 或 refinement 逐步恢复文本。

### MDLM

`Simple and Effective Masked Diffusion Language Models` 的简称，是 masked diffusion language modeling 的一个关键实践节点。

### Remasking

重掩码。指一种推理策略：把低置信度 token 位置重新隐藏起来，让它们在后续去噪步骤中继续被修正。

## N

### Noise Schedule

噪声日程。指在每个 diffusion step 施加多少腐蚀的规则。

## P

### Parallel Decoding

并行解码。指在每个去噪步骤里，同时更新很多 token 位置。

### Perplexity

困惑度。语言模型常用的 likelihood 相关指标。对一些 DLLM 来说更自然可得，对另一些则不一定直接对应其训练形式。

## R

### Reverse Process

反向过程。指模型学习到的生成过程，用来近似把 \(x_t\) 恢复到更干净状态，如 \(x_{t-1}\) 或 \(x_0\)。

## S

### Score-Entropy

score-entropy。SEDD 中使用的一类离散 diffusion 训练思想，它基于概率比值，而不是直接照搬连续空间里的 score matching。

### Self-Conditioning

self-conditioning。指把上一次对干净状态的估计反馈给模型，帮助稳定或改善迭代去噪。

### Semi-Autoregressive Decoding

半自回归解码。介于全并行解码和完全自回归解码之间的生成方式，常见做法是按 block 或分阶段生成。

### Seq2Seq Diffusion

seq2seq diffusion。指在 source text 条件下进行的 diffusion 式生成，常用于摘要、翻译、改写、对话等任务。

## T

### Timestep Weighting

timestep 加权。指训练时对不同噪声级别 \(t\) 分配不同重要性的设计。

### Token-Space Diffusion

token-space diffusion。指扩散过程直接定义在 token 身份上，而不是定义在 latent 向量上。

## X

### x0-Prediction

`x_0` 预测。指模型直接从噪声状态去预测干净状态 \(x_0\) 的参数化方式。
