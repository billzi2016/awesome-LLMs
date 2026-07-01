# Diffusion LLM

[English](./README.md) | [贡献指南](../CONTRIBUTING.zh-CN.md) | [收录规则](../docs/curation-policy.zh-CN.md)

一个面向 Diffusion LLM 的双语、结构化、经过筛选的 Awesome 资源列表。

> 本仓库中的 `DLLM` 明确指 `Diffusion LLM`，不指其他同名缩写。

## 1. 项目范围

本仓库聚焦于使用 diffusion 风格生成过程来处理文本生成、推理、编辑、规划或其他紧密相关语言任务的语言建模工作。

重点关注：

- 基于 diffusion 的语言建模范式；
- 面向语言任务的离散 diffusion 与连续 diffusion 方法；
- Diffusion LLM 的训练、采样与解码策略；
- 为 diffusion language model 设计的对齐、可控生成与后训练方法；
- 评测、benchmark、代码库与代表性应用。

本仓库不打算覆盖：

- 与语言建模没有明确关系的通用 diffusion 资源；
- 普通自回归 LLM 论文，除非它们是 DLLM 工作中的直接对比对象或混合建模组成部分；
- 语言建模不是核心部分的泛多模态 diffusion 工作；
- 没有实质讨论 diffusion language modeling 的泛文本生成综述。

## 2. 项目目标

本项目希望成为：

- 研究者和工程人员进入 Diffusion LLM 方向时的高质量入口；
- 一个有结构的知识地图，而不是简单堆链接；
- 一个中英文内容可对应维护的双语仓库；
- 一个边界明确、方便长期维护的 Awesome 项目。

## 3. 导航

- [README.md](./README.md)：英文版本
- [CONTRIBUTING.md](../CONTRIBUTING.md)：英文贡献指南
- [CONTRIBUTING.zh-CN.md](../CONTRIBUTING.zh-CN.md)：中文贡献指南
- [../docs/curation-policy.md](../docs/curation-policy.md)：英文收录规则
- [../docs/curation-policy.zh-CN.md](../docs/curation-policy.zh-CN.md)：中文收录规则
- [../docs/link-check.md](../docs/link-check.md)：链接检查说明
- [docs/mathematical-foundations.md](./docs/mathematical-foundations.md)：英文数学说明
- [docs/mathematical-foundations.zh-CN.md](./docs/mathematical-foundations.zh-CN.md)：中文数学说明
- [docs/sampling-and-decoding.md](./docs/sampling-and-decoding.md)：英文采样与解码说明
- [docs/sampling-and-decoding.zh-CN.md](./docs/sampling-and-decoding.zh-CN.md)：中文采样与解码说明
- [docs/training-objectives.md](./docs/training-objectives.md)：英文训练目标说明
- [docs/training-objectives.zh-CN.md](./docs/training-objectives.zh-CN.md)：中文训练目标说明
- [docs/method-family-map.md](./docs/method-family-map.md)：英文方法谱系说明
- [docs/method-family-map.zh-CN.md](./docs/method-family-map.zh-CN.md)：中文方法谱系说明
- [docs/benchmark-and-evaluation.md](./docs/benchmark-and-evaluation.md)：英文评测说明
- [docs/benchmark-and-evaluation.zh-CN.md](./docs/benchmark-and-evaluation.zh-CN.md)：中文评测说明
- [docs/overview.md](./docs/overview.md)：英文总览与阅读指南
- [docs/overview.zh-CN.md](./docs/overview.zh-CN.md)：中文总览与阅读指南
- [docs/timeline.md](./docs/timeline.md)：英文发展时间线
- [docs/timeline.zh-CN.md](./docs/timeline.zh-CN.md)：中文发展时间线
- [docs/glossary.md](./docs/glossary.md)：英文术语表
- [docs/glossary.zh-CN.md](./docs/glossary.zh-CN.md)：中文术语表
- [../docs/resource-template.md](../docs/resource-template.md)：资源条目模板

## 4. 分层结构

本仓库会按照“从整体理解到具体实现”的方式组织内容。

### 4.1 总览与综述

这一层主要收录：

- diffusion language modeling 方向的综述；
- 与非自回归生成、diffusion 文本生成相关的框架性文章或分类讨论；
- 有助于快速建立领域全局认识的教程型材料。

目的：

- 让新读者先理解问题背景、术语和研究版图，再进入具体论文。

### 4.2 基础论文

这一层主要收录：

- 定义 Diffusion LLM 核心问题设定的早期或代表性论文；
- 建立主要训练目标、生成过程或理论视角的关键工作；
- 后续 DLLM 论文经常引用的基础参考文献。

目的：

- 用核心思想组织领域，而不是只按零散实现堆积内容。

### 4.3 训练范式

这一层主要收录与 Diffusion LLM 训练方式相关的工作，例如：

- 离散 token diffusion；
- 面向语言的连续 latent 或 embedding 空间 diffusion；
- 与 diffusion 形式紧密相关的 masked denoising 变体；
- 目标函数设计、噪声调度、时间步参数化和优化策略；
- 直接服务于 DLLM 训练的蒸馏或加速方法。

目的：

- 把模型是如何构建出来的技术选择梳理清楚。

### 4.4 推理、采样与解码

这一层主要收录：

- 迭代去噪与生成流程；
- 采样加速方法；
- 面向 diffusion language model 的并行生成或特殊解码策略；
- 推理阶段的可控生成与编辑机制；
- 生成效率与质量之间的权衡方法。

目的：

- 把“模型定义”与“推理期算法和工程设计”分开整理。

### 4.5 对齐与后训练

这一层主要收录：

- 适配 diffusion language model 的偏好优化方法；
- 面向 DLLM 的指令微调、对齐、安全性和可控性方法；
- 用于提升任务表现、稳定性或实用性的后训练策略。

目的：

- 观察 diffusion LLM 是否能够建立起类似现代 LLM 系统所需的后训练能力栈。

### 4.6 评测与 Benchmark

这一层主要收录：

- 用于比较 DLLM 与自回归模型的 benchmark；
- 面向流畅性、一致性、推理能力、可控性、效率和多样性的评估方法；
- 关于失效模式、扩展规律和鲁棒性的分析工作。

目的：

- 让比较标准清晰可见，而不是把不同设定下的结论混在一起。

### 4.7 代码库、实现与工具

这一层主要收录：

- 官方实现；
- 有明确技术价值且持续维护的第三方代码库；
- 可复现的训练或推理框架；
- 有助于研究和扩展 diffusion language model 的工程资源。

目的：

- 帮助读者从论文过渡到可运行系统。

### 4.8 应用

这一层主要收录以 Diffusion LLM 为核心方法的应用型工作，例如：

- 文本生成；
- 推理；
- 编辑与改写；
- 规划；
- 特定领域的语言生成任务。

目的：

- 展示 DLLM 不只是方法讨论，也包括实际使用场景。

## 5. 第一批整理后的资源

### 5.1 综述与阅读列表

- **Diffusion Models for Non-autoregressive Text Generation: A Survey** - Li 等，IJCAI 2023。聚焦非自回归文本生成中的 diffusion 方法，对问题设定、优化方式和设计空间做系统梳理。[[论文](https://arxiv.org/abs/2303.06574)]
- [RUCAIBox/Awesome-Text-Diffusion-Models](https://github.com/RUCAIBox/Awesome-Text-Diffusion-Models) - 上述综述论文的官方配套仓库，适合作为更广泛的 text diffusion 阅读入口。
- **Promises, Outlooks and Challenges of Diffusion Language Modeling** - Deschenaux 和 Gulcehre，2024。偏分析视角，总结 diffusion language modeling 的优势、局限与待解决问题。[[论文](https://arxiv.org/abs/2406.11473)]

### 5.2 基础论文

- **Structured Denoising Diffusion Models in Discrete State-Spaces** - Austin 等，NeurIPS 2021。提出 D3PM，是后续大量离散文本 diffusion 工作的重要基础框架。[[论文](https://arxiv.org/abs/2107.03006)]
- **Diffusion-LM Improves Controllable Text Generation** - Li 等，NeurIPS 2022。连续 diffusion language model 的代表性工作之一，重点展示细粒度可控生成能力。[[论文](https://arxiv.org/abs/2205.14217)]
- **Self-conditioned Embedding Diffusion for Text Generation** - Strudel 等，2022。研究在 embedding 空间中直接进行连续 diffusion，用于条件和无条件文本生成。[[论文](https://arxiv.org/abs/2211.04236)]
- **SSD-LM: Semi-autoregressive Simplex-based Diffusion Language Model for Text Generation and Modular Control** - Han 等，2022。提出 simplex 空间 diffusion 与半自回归分块生成，并强调模块化可控生成。[[论文](https://arxiv.org/abs/2210.17432)]
- **Likelihood-Based Diffusion Language Models** - Gulrajani 和 Hashimoto，2023。关注 diffusion language model 的最大似然训练和扩展规律，包括 Plaid 1B。[[论文](https://arxiv.org/abs/2305.18619)]

### 5.3 训练范式

- **Text Generation with Diffusion Language Models: A Pre-training Approach with Continuous Paragraph Denoise** - Lin 等，2022。提出 GENIE，用连续段落去噪目标来做预训练 diffusion language model。[[论文](https://arxiv.org/abs/2212.11685)] [[代码](https://github.com/microsoft/ProphetNet/tree/master/GENIE)]
- **DiffuSeq: Sequence to Sequence Text Generation with Diffusion Models** - Gong 等，ICLR 2023。面向条件文本生成的 seq2seq diffusion 框架，在多样性方面很有代表性。[[论文](https://arxiv.org/abs/2210.08933)] [[代码](https://github.com/Shark-NLP/DiffuSeq)]
- **SeqDiffuSeq: Text Diffusion with Encoder-Decoder Transformers** - Yuan 等，2022。基于 encoder-decoder Transformer 的 seq2seq diffusion 模型，引入 self-conditioning 和自适应噪声日程。[[论文](https://arxiv.org/abs/2212.10325)]
- **Latent Diffusion for Language Generation** - Lovelace 等，2022。在语言 autoencoder 的 latent 空间中学习 diffusion，把 diffusion 与预训练语言模型结合起来。[[论文](https://arxiv.org/abs/2212.09462)]
- **DiffusionBERT: Improving Generative Masked Language Models with Diffusion Models** - He 等，2022。利用 BERT 初始化去学习 absorbing-state diffusion 的反向过程。[[论文](https://arxiv.org/abs/2211.15029)]
- **A Reparameterized Discrete Diffusion Model for Text Generation** - Zheng 等，2023。给离散 diffusion 采样提出等价的重参数化写法，以改善训练与解码。[[论文](https://arxiv.org/abs/2302.05737)]
- **DINOISER: Diffused Conditional Sequence Learning by Manipulating Noises** - Ye 等，2023。通过自适应噪声范围设计提升条件 diffusion 序列生成的训练与推理效果。[[论文](https://arxiv.org/abs/2302.10025)]
- **Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution** - Lou 等，ICML 2024。提出 SEDD 和 score-entropy 训练方法，显著提升离散 diffusion 语言建模效果。[[论文](https://arxiv.org/abs/2310.16834)]
- **Simple and Effective Masked Diffusion Language Models** - Sahoo 等，NeurIPS 2024。说明简化后的 masked discrete diffusion 目标可以把与自回归模型的差距明显缩小。[[论文](https://arxiv.org/abs/2406.07524)] [[项目页](https://s-sahoo.com/mdlm/)]

### 5.4 推理、采样与解码

- **AR-Diffusion: Auto-Regressive Diffusion Model for Text Generation** - Wu 等，2023。把左到右依赖关系引入 diffusion 去噪过程，用于改善生成质量与速度。[[论文](https://arxiv.org/abs/2305.09515)] [[代码](https://github.com/microsoft/ProphetNet/tree/master/AR-diffusion)]
- **DiffusER: Discrete Diffusion via Edit-based Reconstruction** - Reid 等，2022。把文本生成表述为迭代式编辑重建，对 revision-oriented generation 很有代表性。[[论文](https://arxiv.org/abs/2210.16886)]
- **Diffusion-NAT: Self-Prompting Discrete Diffusion for Non-Autoregressive Text Generation** - Zhou 等，2023。把离散 diffusion 与 BART 式去噪、iterative self-prompting 结合起来做 text-to-text generation。[[论文](https://arxiv.org/abs/2305.04044)]
- **Simple and Effective Masked Diffusion Language Models** - Sahoo 等，NeurIPS 2024。除了训练目标外，也包含高效采样与任意长度半自回归生成策略。[[论文](https://arxiv.org/abs/2406.07524)] [[项目页](https://s-sahoo.com/mdlm/)]

### 5.5 对齐与后训练

- **Diffusion Language Models Can Perform Many Tasks with Scaling and Instruction-Finetuning** - Ye 等，2023。研究 diffusion language model 在多任务场景下的 scaling、任务微调和 instruction finetuning。[[论文](https://arxiv.org/abs/2308.12219)]
- **Large Language Diffusion Models** - Nie 等，2025。提出 LLaDA，一个从头训练并进一步做监督微调的 8B masked diffusion LLM。[[论文](https://arxiv.org/abs/2502.09992)] [[项目页](https://ml-gsai.github.io/LLaDA-demo/)] [[代码](https://github.com/ML-GSAI/LLaDA)]

### 5.6 评测与 Benchmark

- **Promises, Outlooks and Challenges of Diffusion Language Modeling** - Deschenaux 和 Gulcehre，2024。对 SEDD 风格 diffusion LM 和自回归基线做对比，并讨论实际权衡。[[论文](https://arxiv.org/abs/2406.11473)]
- **Diffusion Language Models: An Experimental Analysis** - Bertolani 等，2026。对现代 diffusion language models 在多个 benchmark、推理预算和生成设定下做系统比较。[[论文](https://arxiv.org/abs/2606.19475)]
- [LLaDA Evaluation Code](https://github.com/ML-GSAI/LLaDA) - 包含评测脚本和 OpenCompass 相关资源，可作为 DLLM benchmark 复现入口之一。

### 5.7 代码库、实现与工具

- [Shark-NLP/DiffuSeq](https://github.com/Shark-NLP/DiffuSeq) - DiffuSeq 与 DiffuSeq-v2 的官方实现，面向条件 diffusion 文本生成。
- [microsoft/ProphetNet: GENIE](https://github.com/microsoft/ProphetNet/tree/master/GENIE) - GENIE 的官方实现，适合查看预训练 diffusion language modeling 方案。
- [microsoft/ProphetNet: AR-diffusion](https://github.com/microsoft/ProphetNet/tree/master/AR-diffusion) - AR-Diffusion 的官方实现，偏向依赖感知的 diffusion 解码。
- [ML-GSAI/LLaDA](https://github.com/ML-GSAI/LLaDA) - LLaDA 的官方 PyTorch 实现，也包含后续 diffusion LLM 的评测与推理相关工具。
- [RUCAIBox/Awesome-Text-Diffusion-Models](https://github.com/RUCAIBox/Awesome-Text-Diffusion-Models) - 较适合跟踪早期 continuous / discrete text diffusion 文献的配套仓库。

### 5.8 应用

- **Diffusion-LM Improves Controllable Text Generation** - Li 等，NeurIPS 2022。适合作为属性控制和结构控制文本生成的代表性参考。[[论文](https://arxiv.org/abs/2205.14217)]
- **DiffusER: Discrete Diffusion via Edit-based Reconstruction** - Reid 等，2022。适合作为编辑、改写和 prototype-conditioned generation 场景的代表工作。[[论文](https://arxiv.org/abs/2210.16886)]
- **DiffuSeq: Sequence to Sequence Text Generation with Diffusion Models** - Gong 等，ICLR 2023。覆盖对话、问题生成、改写、文本简化等条件生成任务。[[论文](https://arxiv.org/abs/2210.08933)] [[代码](https://github.com/Shark-NLP/DiffuSeq)]
- **Diffusion-NAT: Self-Prompting Discrete Diffusion for Non-Autoregressive Text Generation** - Zhou 等，2023。可作为离散 diffusion 结合预训练去噪模型做 text-to-text generation 的代表应用。[[论文](https://arxiv.org/abs/2305.04044)]

并不是所有“diffusion + text”资源都会被收录。这一版先优先放入有代表性、能帮助建立 DLLM 主干脉络的论文和官方仓库。

## 6. 收录原则

通常情况下，资源应至少满足以下之一：

- 有稳定来源的论文；
- 官方代码仓库；
- 有价值的 benchmark 或 dataset 页面；
- 与 Diffusion LLM 明确相关的文档或技术说明。

更具体的标准请见 [docs/curation-policy.zh-CN.md](../docs/curation-policy.zh-CN.md)。

## 7. 贡献方式

如果你想补充论文、代码、benchmark、数据集或教程：

- 先阅读 [CONTRIBUTING.zh-CN.md](../CONTRIBUTING.zh-CN.md)；
- 遵循 [docs/curation-policy.zh-CN.md](../docs/curation-policy.zh-CN.md)；
- 当资源变更同时影响中英文内容时，请同步维护 [README.md](./README.md) 和 [README.zh-CN.md](./README.zh-CN.md)。

## 8. 当前状态

本仓库已经从纯结构搭建阶段进入第一轮整理阶段。

当前版本已经建立了 DLLM 的分层地图，并补入了第一批论文、项目页和官方实现。下一步应继续按边界谨慎扩充覆盖面，同时保持各 section 的职责稳定、描述客观。

如果你希望先补技术原理而不是继续扩条目，可以直接阅读 [docs/mathematical-foundations.zh-CN.md](./docs/mathematical-foundations.zh-CN.md)。
如果你想先看整个 `docs/` 目录应该按什么顺序读，可以直接阅读 [docs/overview.zh-CN.md](./docs/overview.zh-CN.md)。
如果你更关心推理阶段行为、remasking 和速度质量权衡，可以直接阅读 [docs/sampling-and-decoding.zh-CN.md](./docs/sampling-and-decoding.zh-CN.md)。
如果你想看不同 DLLM 路线到底在优化什么损失、怎么做后训练，可以直接阅读 [docs/training-objectives.zh-CN.md](./docs/training-objectives.zh-CN.md)。
如果你想先把各种代表方法之间的关系看清楚，可以直接阅读 [docs/method-family-map.zh-CN.md](./docs/method-family-map.zh-CN.md)。
如果你想看 DLLM 到底该怎么和 AR LLM 公平比较、该看哪些评测维度，可以直接阅读 [docs/benchmark-and-evaluation.zh-CN.md](./docs/benchmark-and-evaluation.zh-CN.md)。
如果你想按时间顺序理解这个方向怎么演化，可以直接阅读 [docs/timeline.zh-CN.md](./docs/timeline.zh-CN.md)。
如果你想快速查术语含义，可以直接阅读 [docs/glossary.zh-CN.md](./docs/glossary.zh-CN.md)。
