# Diffusion LLM 总览与阅读指南

本文档是 `docs/` 目录的主入口。

当前仓库已经形成两层结构：

- `README.md`：面向公开展示的资源总入口；
- `docs/`：更深入的技术说明与阅读指南。

如果说 README 主要回答的是“哪些内容属于 Diffusion LLM”，那么 `docs/` 主要回答的是：

- 这个方向的数学基础是什么；
- 不同训练目标怎么分；
- 它的解码为什么不同于自回归生成；
- 各种代表方法之间是什么关系；
- DLLM 应该怎样评测。

## 1. 建议阅读顺序

如果想按一条线往下读，建议顺序是：

1. [mathematical-foundations.zh-CN.md](./mathematical-foundations.zh-CN.md)
2. [training-objectives.zh-CN.md](./training-objectives.zh-CN.md)
3. [sampling-and-decoding.zh-CN.md](./sampling-and-decoding.zh-CN.md)
4. [method-family-map.zh-CN.md](./method-family-map.zh-CN.md)
5. [benchmark-and-evaluation.zh-CN.md](./benchmark-and-evaluation.zh-CN.md)
6. [timeline.zh-CN.md](./timeline.zh-CN.md)
7. [glossary.zh-CN.md](./glossary.zh-CN.md)

这条顺序对应的是：

\[
\text{基本建模}
\rightarrow
\text{训练方式}
\rightarrow
\text{推理方式}
\rightarrow
\text{方法版图}
\rightarrow
\text{评测框架}.
\]

## 2. 按读者类型快速进入

### 2.1 如果你刚接触 DLLM

建议先看：

- [mathematical-foundations.zh-CN.md](./mathematical-foundations.zh-CN.md)
- [method-family-map.zh-CN.md](./method-family-map.zh-CN.md)

### 2.2 如果你主要关心数学

建议先看：

- [mathematical-foundations.zh-CN.md](./mathematical-foundations.zh-CN.md)
- [training-objectives.zh-CN.md](./training-objectives.zh-CN.md)

### 2.3 如果你主要关心推理与解码

建议先看：

- [sampling-and-decoding.zh-CN.md](./sampling-and-decoding.zh-CN.md)
- [benchmark-and-evaluation.zh-CN.md](./benchmark-and-evaluation.zh-CN.md)

### 2.4 如果你主要想整理论文

建议先看：

- [method-family-map.zh-CN.md](./method-family-map.zh-CN.md)
- [README.zh-CN.md](../README.zh-CN.md)

### 2.5 如果你主要想看历史演化

建议先看：

- [timeline.zh-CN.md](./timeline.zh-CN.md)
- [method-family-map.zh-CN.md](./method-family-map.zh-CN.md)

### 2.6 如果你主要想快速查术语

建议先看：

- [glossary.zh-CN.md](./glossary.zh-CN.md)
- [mathematical-foundations.zh-CN.md](./mathematical-foundations.zh-CN.md)

## 3. 每篇文档各自负责什么

### 3.1 数学基础

[mathematical-foundations.zh-CN.md](./mathematical-foundations.zh-CN.md) 主要解释：

- 自回归 LM 和 DLLM 的概率结构差异；
- continuous diffusion；
- discrete diffusion；
- masked diffusion；
- likelihood 与 score-based 两种视角。

### 3.2 训练目标

[training-objectives.zh-CN.md](./training-objectives.zh-CN.md) 主要解释：

- 噪声预测；
- 干净状态预测；
- 离散 token 重建；
- masked diffusion loss；
- ELBO 风格训练；
- instruction finetuning。

### 3.3 采样与解码

[sampling-and-decoding.zh-CN.md](./sampling-and-decoding.zh-CN.md) 主要解释：

- 同步并行解码；
- remasking；
- 自适应日程；
- 分块生成；
- AR-Diffusion 风格的位置偏置；
- inference-time 的速度质量权衡。

### 3.4 方法谱系图

[method-family-map.zh-CN.md](./method-family-map.zh-CN.md) 主要解释：

- 连续 latent 空间方法；
- 一般性离散 diffusion；
- masked diffusion；
- seq2seq diffusion；
- hybrid decoding 方法。

### 3.5 Benchmark 与评测

[benchmark-and-evaluation.zh-CN.md](./benchmark-and-evaluation.zh-CN.md) 主要解释：

- 质量、多样性、可控性之间怎么分；
- 和 AR 比较时有哪些公平性问题；
- decoding-budget 曲线为什么重要；
- 条件任务怎么评测；
- 大规模 DLLM 应该怎么评测。

### 3.6 发展时间线

[timeline.zh-CN.md](./timeline.zh-CN.md) 主要解释：

- 离散 diffusion 是什么时候开始变得可做的；
- latent formulation 是什么时候占主流的；
- masked diffusion 是什么时候显著变强的；
- scaling 和 inference-time refinement 是什么时候变成主轴的。

### 3.7 术语表

[glossary.zh-CN.md](./glossary.zh-CN.md) 主要解释：

- 仓库里反复出现的 DLLM 术语；
- 关键概念的短定义；
- 各篇说明之间的术语对齐。

## 4. README 和 Docs 是怎么配合的

这个仓库是刻意分成两层的。

### 4.1 README 这一层

README 主要负责：

- 仓库公开展示；
- 资源收录；
- 快速扫描；
- 双语入口。

### 4.2 Docs 这一层

`docs/` 主要负责：

- 技术解释；
- 概念澄清；
- 方法比较；
- 阅读引导；
- 后续扩展成更细的专题说明。

一句话概括就是：

\[
\text{README 是地图，docs 是讲解手册}.
\]

## 5. 后续自然的扩展方向

当前 `docs/` 已经覆盖了核心骨架。接下来比较自然的扩展方向包括：

- benchmark 表格；
- 任务 taxonomy；
- 常见误解说明。

## 6. 最小总结

如果只保留一句最有用的话，可以记成：

\[
\text{README 看资源，docs 看结构。}
\]
