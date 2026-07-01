# Timeline of Diffusion LLM Development

This document summarizes how the Diffusion LLM landscape evolved over time. The goal is not to list every paper, but to show the main shifts in formulation, training, scaling, and decoding.

## 1. Before DLLMs Became a Recognized Family

Before "Diffusion LLM" became a recognizable label, the background came from two directions:

- continuous diffusion models in images and other continuous domains;
- non-autoregressive and masked-style text generation in NLP.

The main unresolved question was:

\[
\text{how can diffusion-style generation be adapted to discrete language?}
\]

That question drives most of the timeline below.

## 2. 2021: Discrete Diffusion Becomes Technically Plausible

### 2.1 D3PM

**Structured Denoising Diffusion Models in Discrete State-Spaces** (Austin et al., NeurIPS 2021) is the key milestone here.

Why it matters:

- it gave diffusion a principled discrete-state formulation;
- it showed that corruption does not have to be Gaussian;
- it provided a foundation for token-level diffusion modeling.

This paper is best understood as the technical bridge from image-style diffusion to token-space diffusion.

## 3. 2022: Early Language-Specific Diffusion Formulations

2022 is the year when diffusion for language starts to become a real subfield rather than an imported idea.

### 3.1 Continuous and Latent Formulations

Key papers:

- `Diffusion-LM Improves Controllable Text Generation`
- `Self-conditioned Embedding Diffusion for Text Generation`
- `GENIE`
- `Latent Diffusion for Language Generation`

What changed:

- language generation was explicitly cast as a diffusion problem;
- latent-space and embedding-space denoising became the dominant early pattern;
- controllable generation emerged as a major motivation.

At this stage, the field was still close to image-style diffusion thinking:

\[
\text{continuous denoising first, language adaptation second}.
\]

### 3.2 Conditional Text Generation Line

Key papers:

- `DiffuSeq`
- `DiffusER`
- `SSD-LM`

What changed:

- diffusion started to be used for seq2seq and conditional tasks;
- revision, editing, and diversity became explicit selling points;
- some methods began mixing diffusion refinement with structured generation order.

This is also where the field started to split into:

- open-ended language modeling;
- task-oriented conditional generation.

## 4. 2023: Better Training Views and More Hybrid Designs

In 2023, the field became less exploratory and more methodologically self-aware.

### 4.1 Likelihood and Scaling

Key paper:

- `Likelihood-Based Diffusion Language Models`

What changed:

- diffusion language modeling was analyzed through likelihood rather than only sample quality;
- the field became more interested in scaling behavior;
- comparison with classical LM metrics became more serious.

This is the point where DLLMs started to ask:

\[
\text{can diffusion be a real language-modeling alternative, not just a controllable-generation trick?}
\]

### 4.2 Better Conditional Diffusion

Key papers:

- `SeqDiffuSeq`
- `DINOISER`
- `Diffusion-NAT`

What changed:

- conditional generation setups became more refined;
- noise scheduling and decoder design became more task-aware;
- diffusion started to compete more directly with strong seq2seq baselines.

### 4.3 Hybrid and AR-Biased Designs

Key paper:

- `AR-Diffusion`

What changed:

- the field recognized that fully synchronous generation might not respect language order well enough;
- some methods reintroduced left-to-right bias inside a diffusion process.

This is an important conceptual shift:

\[
\text{pure parallelism}
\rightarrow
\text{parallelism with structural bias}.
\]

## 5. 2024: Discrete and Masked Diffusion Become Much Stronger

This is one of the most important transition years.

### 5.1 SEDD

Key paper:

- `Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution`

What changed:

- discrete diffusion training became much more competitive;
- the field moved beyond naive token reconstruction;
- objective design became central.

SEDD matters because it showed that the main bottleneck was not only "diffusion is hard for language", but also "the wrong discrete objective makes it look worse than it should."

### 5.2 MDLM

Key paper:

- `Simple and Effective Masked Diffusion Language Models`

What changed:

- masked diffusion became a strong practical route;
- simplified absorbing-state objectives scaled better than many people expected;
- decoding efficiency and semi-autoregressive variants gained more attention.

At this point, the field moved away from asking whether masked diffusion was merely "BERT-like", and toward asking whether it could support serious language modeling at scale.

## 6. 2025: Large-Scale DLLMs Become Credible

### 6.1 LLaDA

Key paper:

- `Large Language Diffusion Models`

What changed:

- masked diffusion was pushed into large-scale LLM territory;
- supervised finetuning became part of the DLLM story;
- evaluation started to look more like modern LLM evaluation.

This is a major symbolic moment:

\[
\text{DLLM}
\rightarrow
\text{not only a paper family, but a system-level model class}.
\]

## 7. 2026: Inference-Time Refinement Becomes a Major Theme

By 2026, the field is not only asking how to train DLLMs better, but also how to decode them better.

### 7.1 Context-Robust Remasking

Example:

- `CoRe: Context-Robust Remasking for Diffusion Language Models`

What changed:

- remasking policy itself became a research target;
- token confidence was no longer assumed to be enough;
- context sensitivity entered the decoding story.

### 7.2 Continuous or Flow-Like Decoding Views

Example:

- `Masked Diffusion Decoding as x-Prediction Flow`

What changed:

- some researchers began rethinking binary remask/commit decisions;
- decoding was reframed as smoother iterative refinement;
- the line between discrete diffusion decoding and flow-like views became more interesting.

This marks another shift:

\[
\text{training objective focus}
\rightarrow
\text{inference algorithm focus}.
\]

## 8. The Main Historical Pattern

The timeline can be summarized in four phases:

1. discrete diffusion becomes possible;
2. language-specific formulations appear;
3. objective design becomes stronger;
4. large-scale masked DLLMs and inference-time methods become central.

In compressed form:

\[
\text{discrete foundation}
\rightarrow
\text{language adaptation}
\rightarrow
\text{objective improvement}
\rightarrow
\text{scaling and decoding refinement}.
\]

## 9. How to Use This Timeline

Use this document when you want to answer questions such as:

- which papers are foundational and which are later refinements?
- when did the field move from latent diffusion to masked diffusion?
- when did scaling become serious?
- when did decoding become a research topic by itself?

It is especially useful together with:

- [method-family-map.md](./method-family-map.md)
- [training-objectives.md](./training-objectives.md)
- [sampling-and-decoding.md](./sampling-and-decoding.md)

## 10. Minimal Summary

The shortest useful summary is:

\[
\text{DLLMs evolved from “can diffusion handle text?” to “which training and decoding design scales best for language?”}
\]
