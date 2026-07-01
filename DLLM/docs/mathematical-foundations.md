# Mathematical Foundations of Diffusion LLMs

This document provides a compact but explicit mathematical overview of Diffusion LLMs (DLLMs), with emphasis on the relationship between continuous diffusion, discrete diffusion, masked diffusion, and autoregressive language modeling.

## 1. Why DLLMs Need Separate Mathematics

Diffusion models were first developed and scaled mainly in continuous domains such as images, where a data point is naturally represented as a vector in \( \mathbb{R}^d \). Standard Gaussian perturbations are therefore well-defined.

Language modeling is different:

- text is discrete rather than continuous;
- the output space is combinatorial and tokenized;
- a small perturbation in embedding space does not necessarily correspond to a valid nearby text sequence;
- generation quality depends heavily on long-range symbolic structure.

This is why DLLMs split into several mathematical families:

- continuous diffusion over embeddings or latent vectors;
- discrete diffusion over tokens;
- masked or absorbing-state diffusion, where corruption gradually replaces tokens with `[MASK]`;
- hybrid formulations that keep diffusion-style sampling but borrow language-model training or decoding structure.

## 2. Autoregressive Language Modeling as the Baseline

A standard autoregressive language model factorizes a sequence probability as

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t}).
\]

Training minimizes the negative log-likelihood

\[
\mathcal{L}_{\mathrm{AR}} = - \sum_{t=1}^{T} \log p_\theta(x_t \mid x_{<t}).
\]

This gives:

- exact left-to-right factorization;
- natural next-token decoding;
- easy likelihood computation.

But it also imposes:

- sequential generation;
- exposure bias from one-way decoding;
- difficulty revising arbitrary token positions jointly.

DLLMs relax the left-to-right factorization and instead learn how to reverse a corruption process.

## 3. Generic Diffusion View

Let \(x_0\) denote clean data sampled from the data distribution \(q(x_0)\).

A diffusion model defines:

1. a forward corruption process that transforms \(x_0\) into noisier states \(x_t\);
2. a reverse model that learns to map noisy states back toward clean data.

At a high level:

\[
x_0 \rightarrow x_t \rightarrow x_T,
\]

where \(x_T\) is close to a simple prior or fully corrupted state.

The learning problem is to approximate the reverse transitions

\[
p_\theta(x_{t-1} \mid x_t).
\]

Sampling then starts from \(x_T\) and iteratively applies the learned reverse process.

## 4. Continuous Diffusion for Language

Continuous DLLMs usually map tokens into a continuous embedding space and apply Gaussian-style corruption there.

### 4.1 Forward Process

For a continuous representation \(z_0 \in \mathbb{R}^d\), a standard variance-preserving forward process can be written as

\[
q(z_t \mid z_0) = \mathcal{N}\bigl(z_t; \alpha_t z_0, \sigma_t^2 I\bigr),
\]

or equivalently

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon, \qquad \epsilon \sim \mathcal{N}(0, I).
\]

Here:

- \( \alpha_t \) controls signal preservation;
- \( \sigma_t \) controls noise magnitude;
- larger \(t\) means noisier latent states.

### 4.2 Reverse Model

The model learns to predict either:

- the clean latent \(z_0\),
- the noise \( \epsilon \),
- or a score / denoising direction.

A common simplified objective is

\[
\mathcal{L}_{\mathrm{cont}} = \mathbb{E}_{z_0, \epsilon, t}
\left[
\lVert \epsilon - \epsilon_\theta(z_t, t) \rVert_2^2
\right].
\]

For text, this continuous formulation appears in works such as:

- `Diffusion-LM`;
- `Self-conditioned Embedding Diffusion`;
- `GENIE` and related latent-denoising approaches.

### 4.3 Main Tension in Language

The continuous formulation is mathematically convenient, but language itself is discrete. So the model must eventually project denoised continuous states back to tokens. This creates a mismatch:

- diffusion is smooth in latent space;
- text validity is symbolic and discontinuous in token space.

That mismatch is one reason discrete and masked diffusion formulations became important.

## 5. Discrete Diffusion

Instead of diffusing in \( \mathbb{R}^d \), discrete diffusion defines corruption directly on token states.

Let the vocabulary be \( \mathcal{V} \), and each token position take values in that finite set.

### 5.1 Markov Corruption

In D3PM-style formulations, the forward process is defined by transition matrices \(Q_t\):

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

Over multiple steps,

\[
q(x_t \mid x_0)
= Q_t Q_{t-1} \cdots Q_1 x_0.
\]

The matrices \(Q_t\) specify how tokens are corrupted, for example:

- replaced uniformly;
- replaced according to a structured neighborhood;
- moved toward a special absorbing state.

### 5.2 Reverse Learning

The reverse model aims to learn

\[
p_\theta(x_{t-1} \mid x_t),
\]

or a parameterization that allows those reverse transitions to be constructed.

Compared with continuous diffusion, the advantage is conceptual alignment with text tokens. The challenge is that discrete score matching and likelihood estimation are harder.

## 6. Masked / Absorbing-State Diffusion

Many modern DLLMs use masked diffusion, because it matches language modeling more naturally.

### 6.1 Absorbing State

Introduce a special mask token \(m = [\mathrm{MASK}]\). Then the forward process gradually replaces clean tokens with \(m\).

A simple marginal form is:

\[
q(x_t \mid x_0) = \mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr),
\]

where:

- \(x_0\) is the clean token;
- \(m\) is the one-hot mask distribution;
- \(1-\alpha_t\) is the masking probability at time \(t\).

At small noise levels, most tokens remain visible. At large noise levels, most tokens become masked.

### 6.2 Training Objective

The model observes a partially masked sequence \(x_t\) and predicts the original tokens on masked positions:

\[
\mathcal{L}_{\mathrm{mask}}
=
\mathbb{E}_{x_0, t, x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right],
\]

where \(M_t\) is the set of masked positions at timestep \(t\).

This looks similar to masked language modeling, but there is a crucial difference:

- BERT uses a fixed or manually designed masking ratio;
- masked diffusion samples corruption levels across a continuum or schedule of noise levels.

That difference is what lets methods such as MDLM and LLaDA interpret the objective as a generative diffusion objective rather than plain denoising pretraining.

### 6.3 Why It Fits Language Better

Masked diffusion has several structural advantages:

- it preserves discrete token semantics;
- it supports parallel prediction over many positions;
- it naturally supports infilling and editing;
- it can start from full masking and iteratively unmask text.

This is why many recent large-scale DLLMs are masked diffusion models.

### 6.4 Why Masked Diffusion Is Not Just BERT

At first glance, the masked diffusion loss looks similar to masked language modeling. But the probabilistic role of the loss is different.

For BERT-style MLM, training usually samples a corruption rule with a fixed masking design and minimizes

\[
\mathcal{L}_{\mathrm{MLM}}
=
\mathbb{E}_{x_0, \widetilde{x}}
\left[
- \sum_{i \in M} \log p_\theta(x_{0,i} \mid \widetilde{x})
\right].
\]

This is a conditional denoising objective, but not by itself a full generative model over complete sequences.

For masked diffusion, the noise level \(t\) is part of the generative definition. The model is trained over a family of corruption levels:

\[
t \sim p(t), \qquad x_t \sim q(x_t \mid x_0),
\]

and minimizes

\[
\mathcal{L}_{\mathrm{MD}}
=
\mathbb{E}_{x_0, t, x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right].
\]

The important distinction is that \(t\) sweeps the whole corruption path from nearly clean to nearly fully masked states. That makes the model learn a reverse process over the entire path, not just a single denoising task.

So the relationship is:

\[
\text{MLM-like local loss}
\neq
\text{plain MLM training objective},
\]

because in DLLMs the same local token prediction loss is tied to a global diffusion sampling process.

## 7. Likelihood and Score-Based Views

Different DLLM papers use different mathematical lenses.

### 7.1 Likelihood-Oriented View

Some works, such as `Likelihood-Based Diffusion Language Models`, emphasize that training should optimize a variational or upper-bound objective linked to sequence likelihood.

The high-level idea is:

\[
-\log p_\theta(x_0) \leq \mathcal{L}_{\mathrm{diff}}(x_0),
\]

where \( \mathcal{L}_{\mathrm{diff}} \) is a diffusion training objective whose minimization improves a tractable bound related to the model likelihood.

This matters because language modeling is usually judged by perplexity or likelihood-based metrics, not only sample quality.

### 7.2 Score-Entropy View

SEDD reframes discrete diffusion using ratios of marginal probabilities. The discrete score is

\[
s(x,t)_y = \frac{q_t(y)}{q_t(x)}.
\]

The learned model approximates this ratio:

\[
s_\theta(x,t)_y \approx \frac{q_t(y)}{q_t(x)}.
\]

The score-entropy objective then replaces unstable discrete score matching with a better-behaved training rule for discrete spaces.

This is important because it narrows the quality gap between discrete diffusion and autoregressive models.

### 7.3 Variational Bound Intuition

A more explicit way to view diffusion training is through a latent-variable bound. Let the full noisy trajectory be \(x_{1:T}\). Then the model defines a reverse chain

\[
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1}\mid x_t),
\]

while the forward process defines

\[
q(x_{1:T}\mid x_0) = \prod_{t=1}^{T} q(x_t\mid x_{t-1}).
\]

Using standard variational arguments, one can derive a bound of the form

\[
-\log p_\theta(x_0)
\le
\mathbb{E}_{q(x_{1:T}\mid x_0)}
\left[
-\log p(x_T)
- \sum_{t=1}^{T} \log p_\theta(x_{t-1}\mid x_t)
+ \sum_{t=1}^{T} \log q(x_t\mid x_{t-1})
\right].
\]

This expression is not always the exact training loss used in practice, but it explains the logic:

- the forward chain is fixed;
- the reverse chain is learned;
- minimizing diffusion loss improves an upper bound or variational surrogate for sequence likelihood.

For continuous diffusion, this often simplifies to noise-prediction losses. For masked diffusion, it often simplifies to weighted masked-token reconstruction terms.

## 8. Sampling in DLLMs

At inference time, DLLMs do not usually decode one token at a time in the standard autoregressive sense.

Instead, they iteratively refine a partially corrupted sequence:

\[
x_T \rightarrow x_{T-1} \rightarrow \cdots \rightarrow x_0.
\]

In masked diffusion, the process often looks like:

1. start from an all-mask sequence;
2. predict tokens for many positions in parallel;
3. optionally remask low-confidence positions;
4. refine over several denoising iterations.

This gives DLLMs several distinctive behaviors:

- parallel token updates;
- editability;
- flexible conditioning on arbitrary positions;
- a direct tradeoff between number of denoising steps and output quality.

That last point is central:

\[
\text{more denoising steps} \Rightarrow \text{higher compute, often better quality}.
\]

So DLLMs expose a compute-quality knob more explicitly than standard left-to-right decoding.

### 8.1 Confidence, Remasking, and Partial Refinement

Many masked DLLMs do not simply fill all masked positions once and stop. Instead, they repeatedly refine predictions.

Let \(x_t^{(i)}\) denote the token at position \(i\) during iteration \(t\), and let

\[
\pi_\theta(\cdot \mid x_t, t)
\]

be the predictive distribution over tokens. A common heuristic is:

1. predict tokens for masked positions;
2. compute confidence scores such as

\[
c_i = \max_{v \in \mathcal{V}} \pi_\theta(v \mid x_t, t);
\]

3. keep high-confidence positions fixed;
4. remask low-confidence positions and denoise again.

This makes the effective reverse process adaptive rather than strictly deterministic.

The intuition is:

\[
\text{easy positions stabilize early, hard positions keep evolving}.
\]

That behavior is one of the main practical differences between diffusion-style generation and one-pass masked filling.

### 8.2 Why Parallel Generation Is Possible

In autoregressive decoding, the factorization

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t})
\]

forces causal dependence across token positions during sampling.

In DLLMs, the model is instead conditioned on a noisy global state \(x_t\), and predicts many positions jointly:

\[
p_\theta(x_0 \mid x_t, t).
\]

That does not mean positions are independent. It means dependence is mediated through the shared denoising state rather than a left-to-right chain.

So parallel generation is possible because:

- the model sees the whole partially corrupted sequence;
- all positions can exchange information through bidirectional attention;
- denoising iterations replace serial token order with serial refinement steps.

In short:

\[
\text{AR LLM: serial tokens}
\qquad
\text{DLLM: serial refinements}.
\]

## 9. Unifying Representative Methods

The main papers in this repository can be grouped under one mathematical map.

### 9.1 Continuous or Latent-Space Methods

- `Diffusion-LM`: continuous diffusion over word-vector style latent representations, especially strong for controllable generation.
- `Self-conditioned Embedding Diffusion`: embedding-space diffusion with self-conditioning.
- `GENIE`: pretraining-oriented diffusion language modeling with continuous paragraph denoising.

### 9.2 Seq2Seq and Conditional Diffusion

- `DiffuSeq`: conditional text generation with diffusion for seq2seq tasks.
- `DiffusER`: iterative edit-based reconstruction, especially relevant for revision and prototype-conditioned generation.

### 9.3 Discrete and Masked Diffusion

- `D3PM`: general discrete diffusion framework.
- `SEDD`: score-entropy training for discrete diffusion.
- `MDLM`: simplified masked diffusion objective with strong empirical performance.
- `LLaDA`: large-scale masked diffusion LLM trained from scratch and then instruction-tuned.

### 9.4 Hybrid Dependency-Aware Decoding

- `AR-Diffusion`: injects left-to-right dependency structure into diffusion denoising.

So a useful summary is:

\[
\text{DLLM} =
\text{corruption process}
+ \text{reverse denoising model}
+ \text{sampling schedule}
+ \text{tokenization / representation choice}.
\]

Different papers mainly differ in those four components.

## 10. DLLMs vs AR LLMs

The mathematical difference is not just architectural; it is probabilistic.

Autoregressive models define:

\[
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t \mid x_{<t}).
\]

Diffusion language models define:

\[
q(x_0) \xrightarrow{\text{forward corruption}} x_t
\xrightarrow{\text{learned reverse process}} p_\theta(x_0).
\]

In practice, this often means:

- AR LLMs are simpler to evaluate and decode;
- DLLMs offer more parallelism and easier non-left-to-right conditioning;
- DLLMs often need more care in schedule design, remasking policy, and inference budget.

So the core research question is not whether DLLMs imitate AR LLMs, but whether diffusion-style generation can become a competitive alternative with its own strengths.

### 10.1 Two Different Notions of Time

One source of confusion is that DLLMs introduce a second notion of "time".

In autoregressive models:

- time means token position.

In diffusion models:

- time means corruption level or denoising step.

These are not the same:

\[
\text{AR time} \neq \text{diffusion time}.
\]

This is why a DLLM may generate all token positions in parallel at one denoising step, then revisit them again at later denoising steps. The sequential structure shifts from token order to refinement order.

### 10.2 Where AR-Diffusion Sits

AR-Diffusion is useful conceptually because it mixes the two notions above. It retains a diffusion process, but injects left-to-right asymmetry into how different positions are denoised.

So it can be viewed as:

\[
\text{diffusion sampling}
+ \text{position-dependent causal bias}.
\]

This helps explain why AR-Diffusion is neither a pure masked DLLM nor a standard AR model.

## 11. Practical Reading Order

If you want to understand the field progressively:

1. read `D3PM` for discrete diffusion basics;
2. read `Diffusion-LM` and `DiffuSeq` for early language-oriented formulations;
3. read `Likelihood-Based Diffusion Language Models` and `SEDD` for likelihood and discrete-training improvements;
4. read `MDLM` for masked diffusion simplification;
5. read `LLaDA` for large-scale diffusion LLM training and instruction tuning.

## 12. Scope Note

This document is intentionally explanatory rather than fully formal. It focuses on the minimum mathematics needed to distinguish major DLLM families and to understand why recent masked diffusion LLMs differ from both BERT-style MLMs and standard autoregressive LLMs.

## 13. Minimal Mental Model

If you only keep one mathematical picture in mind, use this:

\[
\text{clean text}
\xrightarrow{\text{corrupt}}
\text{noisy text}
\xrightarrow{\text{learned reverse refinement}}
\text{clean text}.
\]

Then ask four questions for any DLLM paper:

1. What is the representation space: embeddings, tokens, or masks?
2. What is the forward corruption rule?
3. What exactly is the reverse model trained to predict?
4. How is sampling performed: fixed schedule, remasking, or hybrid decoding?

Those four questions are usually enough to place a new DLLM method into the broader landscape.
