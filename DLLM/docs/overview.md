# Diffusion LLM Overview and Reading Guide

This document is the main entry point for the `docs/` directory.

The repository now contains two layers:

- `README.md`: the public-facing curated resource map;
- `docs/`: deeper technical notes that explain how to read the field.

If the README answers "what belongs in Diffusion LLMs?", the `docs/` directory answers:

- what the main mathematical ideas are;
- how training objectives differ;
- how decoding differs from autoregressive generation;
- how major method families relate to each other;
- how DLLMs should be evaluated.

## 1. Recommended Reading Order

If you want a linear path, use this order:

1. [mathematical-foundations.md](./mathematical-foundations.md)
2. [training-objectives.md](./training-objectives.md)
3. [sampling-and-decoding.md](./sampling-and-decoding.md)
4. [method-family-map.md](./method-family-map.md)
5. [benchmark-and-evaluation.md](./benchmark-and-evaluation.md)
6. [timeline.md](./timeline.md)
7. [glossary.md](./glossary.md)

This order goes from:

\[
\text{basic formulation}
\rightarrow
\text{training}
\rightarrow
\text{inference}
\rightarrow
\text{method map}
\rightarrow
\text{evaluation}.
\]

## 2. Fast Entry by Reader Type

### 2.1 If You Are New to DLLMs

Start with:

- [mathematical-foundations.md](./mathematical-foundations.md)
- [method-family-map.md](./method-family-map.md)

### 2.2 If You Mainly Care About Math

Start with:

- [mathematical-foundations.md](./mathematical-foundations.md)
- [training-objectives.md](./training-objectives.md)

### 2.3 If You Mainly Care About Inference

Start with:

- [sampling-and-decoding.md](./sampling-and-decoding.md)
- [benchmark-and-evaluation.md](./benchmark-and-evaluation.md)

### 2.4 If You Mainly Care About Organizing Papers

Start with:

- [method-family-map.md](./method-family-map.md)
- [README.md](../README.md)

### 2.5 If You Mainly Care About Historical Evolution

Start with:

- [timeline.md](./timeline.md)
- [method-family-map.md](./method-family-map.md)

### 2.6 If You Mainly Need Terminology Clarification

Start with:

- [glossary.md](./glossary.md)
- [mathematical-foundations.md](./mathematical-foundations.md)

## 3. What Each Document Does

### 3.1 Mathematical Foundations

[mathematical-foundations.md](./mathematical-foundations.md) explains:

- autoregressive LM vs DLLM probability structure;
- continuous diffusion;
- discrete diffusion;
- masked diffusion;
- likelihood and score-based views.

### 3.2 Training Objectives

[training-objectives.md](./training-objectives.md) explains:

- noise prediction;
- clean-state prediction;
- discrete reconstruction;
- masked diffusion loss;
- ELBO-style training;
- instruction finetuning.

### 3.3 Sampling and Decoding

[sampling-and-decoding.md](./sampling-and-decoding.md) explains:

- synchronous decoding;
- remasking;
- adaptive schedules;
- blockwise generation;
- AR-Diffusion-style refinement bias;
- inference-time tradeoffs.

### 3.4 Method Family Map

[method-family-map.md](./method-family-map.md) explains:

- continuous latent-space methods;
- general discrete diffusion;
- masked diffusion;
- seq2seq diffusion;
- hybrid decoding methods.

### 3.5 Benchmark and Evaluation

[benchmark-and-evaluation.md](./benchmark-and-evaluation.md) explains:

- quality vs diversity vs controllability;
- fairness issues in AR comparison;
- decoding-budget curves;
- conditional-task evaluation;
- large-scale DLLM evaluation.

### 3.6 Timeline

[timeline.md](./timeline.md) explains:

- when discrete diffusion became viable for language;
- when latent formulations dominated;
- when masked diffusion became strong;
- when scaling and inference-time refinement became central.

### 3.7 Glossary

[glossary.md](./glossary.md) explains:

- recurring DLLM terms;
- short definitions of key concepts;
- terminology alignment across the repository.

## 4. How README and Docs Work Together

The repository is intentionally split into two layers.

### 4.1 README Layer

The README is for:

- public repository presentation;
- curated resources;
- fast scanning;
- bilingual project entry.

### 4.2 Docs Layer

The `docs/` directory is for:

- technical explanation;
- conceptual disambiguation;
- method comparison;
- reading guidance;
- future expansion into more specialized notes.

This means:

\[
\text{README is the map, docs are the guidebook}.
\]

## 5. Suggested Future Extensions

Natural future additions include:

- a paper timeline;
- a benchmark table;
- a task taxonomy;
- a "common misconceptions" note.

## 6. Minimal Summary

If you only need one sentence:

\[
\text{read README for resources, read docs for structure}.
\]
