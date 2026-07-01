# Benchmarks and Evaluation for Diffusion LLMs

This document explains how Diffusion LLMs should be evaluated, and why naive comparison with autoregressive LLMs is often misleading.

The main point is:

\[
\text{DLLM quality}
\neq
\text{one single metric}.
\]

Evaluation should separate:

- text quality;
- diversity;
- controllability;
- inference cost;
- decoding budget;
- task-specific accuracy;
- likelihood-related metrics when available.

## 1. Why Evaluation Is Harder Than It Looks

Diffusion LLMs do not only differ from autoregressive models in architecture. They also differ in:

- decoding procedure;
- refinement budget;
- amount of parallelism;
- degree of revisability;
- relation between training objective and inference strategy.

So any evaluation that ignores inference budget is incomplete.

At a high level:

\[
\text{model quality}
=
f(\text{training}, \text{decoding}, \text{budget}, \text{task}).
\]

That is why a DLLM should not be judged by a single sample or by a single benchmark score without decoding context.

## 2. Core Evaluation Dimensions

### 2.1 Text Quality

This includes:

- fluency;
- grammaticality;
- coherence;
- factual consistency when applicable;
- relevance to the prompt or condition.

In open-ended generation, text quality is often partly judged by:

- human evaluation;
- model-based evaluators;
- task-specific overlap metrics when the task has references.

### 2.2 Diversity

Diffusion models are often motivated by the idea that non-left-to-right generation may support better output diversity.

Useful diversity questions include:

- do multiple samples differ meaningfully?
- does diversity come at the cost of coherence?
- is the model merely noisy, or genuinely diverse?

This matters especially in:

- dialogue generation;
- paraphrasing;
- creative generation;
- conditional generation with multiple valid targets.

### 2.3 Controllability

Some DLLMs, especially early ones like `Diffusion-LM`, emphasize controllable generation.

Controllability evaluation asks:

- does the output satisfy the requested attribute?
- does control preserve fluency?
- does stronger control damage semantic faithfulness?

This is often the main metric when the method is marketed as controllable text generation rather than general-purpose language modeling.

### 2.4 Inference Cost

For DLLMs, this is central rather than secondary.

Important cost variables include:

- number of denoising steps;
- wall-clock latency;
- number of forward passes;
- memory usage;
- effective throughput for long outputs.

For example:

\[
\text{higher quality}
\text{ may come from }
\text{more refinement steps},
\]

so comparing a 32-step DLLM to a 1-pass AR decode without discussing compute is usually not meaningful.

### 2.5 Likelihood-Related Metrics

Where available, one may also evaluate:

- negative log-likelihood;
- perplexity;
- calibration-style measures.

This matters more for methods that explicitly optimize likelihood or ELBO-style objectives, and less for papers focused mainly on downstream generation quality.

## 3. Fair Comparison with AR LLMs

This is the most common source of confusion.

### 3.1 Token Time vs Refinement Time

Autoregressive models spend time on token positions.

DLLMs spend time on refinement steps.

So:

\[
\text{AR complexity is token-serial},
\qquad
\text{DLLM complexity is refinement-serial}.
\]

A fair comparison should state clearly:

- output length;
- number of denoising steps;
- whether all positions are processed each step;
- whether remasking is used;
- effective runtime.

### 3.2 Equal Parameters Is Not Enough

Two models may have similar parameter counts and still be incomparable if:

- one needs many more refinement steps;
- one gets bidirectional context during denoising;
- one uses stronger post-training;
- one is evaluated under a different decoding budget.

So:

\[
\text{equal parameter count}
\neq
\text{equal inference condition}.
\]

### 3.3 Equal Step Count Is Also Not Enough

Even matching the number of refinement steps is not sufficient, because:

- some steps may update every token;
- some may update only masked positions;
- some methods use adaptive remasking;
- some methods use blockwise or position-biased decoding.

The practical unit of comparison should be closer to:

- total model calls;
- total token-position updates;
- wall-clock latency;
- hardware-normalized throughput when possible.

## 4. Open-Ended Language Modeling Evaluation

For general language modeling, useful evaluation dimensions include:

- perplexity or likelihood proxies;
- open-ended generation quality;
- long-form coherence;
- repetition and degeneration behavior;
- sample efficiency under decoding budgets.

In this setting, a DLLM paper should ideally answer:

1. how good are the final samples?
2. how many refinement steps are required?
3. how does quality change as the budget shrinks?
4. how does the method compare against strong AR baselines?

That third question is often underreported but extremely important.

## 5. Conditional Generation Evaluation

For seq2seq-style DLLMs, evaluation depends heavily on the task.

### 5.1 Summarization

Typical dimensions:

- faithfulness;
- coverage;
- compression quality;
- factual consistency;
- ROUGE-style overlap metrics when used.

### 5.2 Translation

Typical dimensions:

- adequacy;
- fluency;
- BLEU or related overlap metrics;
- robustness across sentence lengths.

### 5.3 Dialogue and Response Generation

Typical dimensions:

- relevance;
- diversity;
- consistency;
- specificity;
- safety where relevant.

### 5.4 Paraphrasing and Rewrite Tasks

Typical dimensions:

- semantic preservation;
- lexical diversity;
- stylistic or structural transformation quality.

These tasks are often where diffusion decoding can show strengths because multiple outputs may be valid.

## 6. Large-Scale DLLM Evaluation

Modern DLLMs such as `LLaDA` make evaluation look more like LLM evaluation.

### 6.1 Instruction Following

Questions include:

- does the model follow formatting constraints?
- does it answer the requested task?
- how robust is it under multi-turn or long prompts?

### 6.2 Reasoning

Reasoning evaluation may include:

- math benchmarks;
- multi-step reasoning tasks;
- chain-of-thought-sensitive tasks;
- benchmark suites used in modern LLM evaluation.

For DLLMs, it is especially important to ask whether reasoning quality survives under limited denoising budgets.

### 6.3 Code Generation

Code tasks stress:

- syntax correctness;
- functional correctness;
- long-range dependency handling;
- whether iterative refinement helps or hurts exactness.

Here, diffusion-style revisability may be an advantage in principle, but only if decoding is stable enough.

## 7. Diversity vs Quality Tradeoff

One recurring claim in diffusion-style text generation is that it may support better diversity than strict left-to-right decoding.

This should not be accepted without checking:

\[
\text{is the model diverse, or just unstable?}
\]

A good evaluation should separate:

- output diversity;
- semantic correctness;
- fluency;
- calibration of uncertainty.

Otherwise, noisy generations may be misread as genuine diversity.

## 8. Step-Budget Curves Matter

For DLLMs, single-point reporting is weak. A stronger evaluation reports how quality changes with inference budget.

Conceptually:

\[
\text{quality} = g(K),
\]

where \(K\) is the number of denoising steps.

Useful reports include:

- 4-step quality;
- 8-step quality;
- 16-step quality;
- 32-step quality;
- diminishing returns beyond a certain budget.

This reveals whether a method is:

- inherently efficient;
- good only at high compute;
- robust under budget constraints.

## 9. Human Evaluation Still Matters

Automatic metrics are useful, but DLLMs often generate outputs with different error profiles from AR models.

Human evaluation is especially important for:

- coherence over long outputs;
- edit quality;
- controllability success;
- nuanced failure cases;
- whether revision actually improves the sample.

The best practice is not to replace automatic metrics, but to complement them.

## 10. What a Strong DLLM Evaluation Section Should Include

A strong paper should ideally report:

1. what decoding strategy is used;
2. how many denoising steps are used;
3. whether remasking or adaptive schedules are used;
4. what compute budget or latency is required;
5. how quality changes under smaller budgets;
6. what AR baseline is used and under what inference conditions;
7. whether evaluation covers both quality and diversity;
8. whether any human evaluation is included.

If several of these are missing, comparison claims should be treated cautiously.

## 11. Minimal Summary

The shortest useful summary is:

\[
\text{evaluating a DLLM means evaluating both the model and its decoding budget}.
\]

That is the key difference from naive benchmark reading.
