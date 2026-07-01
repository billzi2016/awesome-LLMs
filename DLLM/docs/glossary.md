# Diffusion LLM Glossary

This glossary defines recurring terms used across the repository. The goal is not to replace full technical explanations, but to keep terminology consistent.

## A

### Absorbing State

A special state that, once entered in the forward process, does not transition back under the same corruption rule. In masked diffusion for text, `[MASK]` is often the absorbing state.

### AR-Diffusion

A hybrid diffusion design that injects left-to-right bias into the denoising process so that different token positions effectively emerge at different times.

## B

### Bidirectional Context

Context from both left and right token positions. This is natural in masked diffusion models and unlike strictly causal autoregressive decoding.

## C

### Clean State

The uncorrupted data point, usually written as \(x_0\) or \(z_0\).

### Conditional Diffusion

A diffusion model that generates output conditioned on some input \(c\), such as a source sentence, prompt, or instruction.

### Continuous Diffusion

A diffusion process defined in a continuous space such as embeddings or latent vectors, often using Gaussian corruption.

## D

### D3PM

Short for `Structured Denoising Diffusion Models in Discrete State-Spaces`, a foundational framework for discrete diffusion.

### Decoding Budget

The amount of inference-time computation used for generation, usually measured by denoising steps, model calls, latency, or related quantities.

### Denoising

The reverse process of moving from a noisy state toward a cleaner state.

### Diffusion Time

The corruption or denoising index \(t\). This is different from token position and should not be confused with autoregressive generation order.

### Discrete Diffusion

A diffusion process defined directly over finite token states rather than continuous vectors.

## E

### ELBO

Evidence Lower Bound. In DLLMs, it is often used to explain how diffusion training can act as a variational surrogate for likelihood-based modeling.

## F

### Forward Corruption Process

The process that maps clean data \(x_0\) to noisy states \(x_t\) by progressively adding corruption.

## I

### Instruction Finetuning

Post-training on instruction-response pairs so that the model learns task following rather than only generic text modeling.

### Iterative Refinement

A decoding pattern where the model repeatedly revises a partially generated sequence rather than committing to a single left-to-right pass.

## L

### LLaDA

`Large Language Diffusion Models`, a large-scale masked diffusion LLM line that helped establish DLLMs as a system-level model class rather than only a paper niche.

### Likelihood-Based Diffusion

A view of diffusion training that emphasizes likelihood, variational bounds, or other quantities comparable to classical language modeling metrics.

## M

### Masked Diffusion

A discrete diffusion formulation where corruption progressively replaces tokens with `[MASK]`, and generation happens by iterative unmasking or refinement.

### MDLM

Short for `Simple and Effective Masked Diffusion Language Models`, a major practical milestone for masked diffusion language modeling.

### Remasking

An inference-time strategy that re-hides low-confidence token positions so they can be refined again in later denoising steps.

## N

### Noise Schedule

A rule that determines how much corruption is applied at each diffusion step.

## P

### Parallel Decoding

A generation pattern where many token positions are updated simultaneously during each denoising step.

### Perplexity

A likelihood-related metric commonly used for language models. It is more naturally available for some DLLMs than others, depending on the training formulation.

## R

### Reverse Process

The learned generative process that approximates how to move from \(x_t\) to a cleaner state such as \(x_{t-1}\) or \(x_0\).

## S

### Score-Entropy

A discrete diffusion training idea used in SEDD, based on probability ratios rather than direct continuous-style score matching.

### Self-Conditioning

A technique where a previous estimate of the clean state is fed back into the model to stabilize or improve iterative denoising.

### Semi-Autoregressive Decoding

A generation scheme between fully parallel and fully autoregressive decoding, often using blocks or staged generation order.

### Seq2Seq Diffusion

Diffusion-based generation conditioned on source text, often used for summarization, translation, paraphrasing, or dialogue tasks.

## T

### Timestep Weighting

A training design choice that gives different importance to different noise levels \(t\).

### Token-Space Diffusion

A diffusion process defined directly over token identities rather than latent vectors.

## X

### x0-Prediction

A parameterization where the model predicts the clean state \(x_0\) directly from a noisy state.
