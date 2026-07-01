# Sampling and Decoding in Diffusion LLMs

This document focuses on how Diffusion LLMs generate text at inference time. The main point is simple:

\[
\text{training objective} \neq \text{decoding strategy}.
\]

Two DLLMs with similar training losses can behave very differently at inference time depending on:

- how many denoising steps they use;
- whether they decode synchronously or asynchronously;
- whether they remask uncertain tokens;
- whether they generate all positions in parallel or in blocks;
- whether they inject left-to-right bias during refinement.

## 1. Decoding as Reverse Corruption

Let \(x_T\) denote a fully corrupted state, and \(x_0\) the final generated text. DLLM decoding applies a learned reverse process:

\[
x_T \rightarrow x_{T-1} \rightarrow \cdots \rightarrow x_0.
\]

In image diffusion, this often means reversing Gaussian noise. In language, especially in masked diffusion, this often means progressively replacing `[MASK]` tokens with content tokens.

The generic decoding question is:

\[
\text{how should we choose } p_\theta(x_{t-1}\mid x_t) \text{ in practice?}
\]

That question has several practical sub-questions:

- do we predict every token position at every step?
- do we commit tokens permanently once predicted?
- do we allow low-confidence tokens to be revised later?
- do all token positions move through diffusion time at the same rate?

## 2. Synchronous Parallel Decoding

The simplest masked DLLM decoder is fully synchronous.

### 2.1 Basic Procedure

Start from an all-mask sequence:

\[
x_T = [m, m, \dots, m].
\]

At each step \(t\), predict a token distribution for every masked position:

\[
\pi_\theta(\cdot \mid x_t, t).
\]

Then either:

- fill all masked positions greedily;
- sample from the distribution;
- or fill a subset and continue refining.

### 2.2 Why It Is Parallel

Unlike autoregressive models, the prediction is conditioned on the whole partially denoised sequence:

\[
p_\theta(x_0 \mid x_t, t),
\]

so all positions can be updated at once through bidirectional attention.

The advantage is lower wall-clock latency for long outputs. The weakness is that early token decisions may be made with weak context.

## 3. Remasking and Iterative Refinement

A core idea in modern DLLM decoding is that token predictions should remain revisable.

### 3.1 Confidence-Based Remasking

Let the confidence of position \(i\) be

\[
c_i = \max_{v \in \mathcal{V}} \pi_\theta(v \mid x_t, t).
\]

Then a common remasking strategy is:

1. predict tokens for candidate positions;
2. keep high-confidence positions fixed;
3. remask low-confidence positions;
4. continue denoising.

This turns decoding into adaptive refinement rather than one-shot filling.

### 3.2 Why Remasking Helps

Early in decoding, context is incomplete. A token can look locally confident but still be globally inconsistent. Remasking gives the model a chance to revise.

This is especially important in:

- code generation;
- reasoning-heavy tasks;
- long-form outputs;
- tasks with strong global coherence constraints.

### 3.3 A Useful Abstraction

Define a binary keep decision \(k_i^{(t)} \in \{0,1\}\) for token position \(i\) at step \(t\):

\[
k_i^{(t)} =
\begin{cases}
1, & c_i \ge \tau_t \\
0, & c_i < \tau_t
\end{cases}
\]

where \( \tau_t \) is a step-dependent threshold. Then decoding alternates between:

- token prediction;
- keep/remask decisions.

This is one of the cleanest ways to think about masked diffusion decoding.

## 4. Fixed Schedules vs Adaptive Schedules

Not all denoising schedules are equally structured.

### 4.1 Fixed Schedule

In the simplest case, the number of still-masked positions decreases according to a predefined schedule:

\[
M_{t-1} = \rho_t M_t,
\]

where \(M_t\) denotes the number of masked positions at step \(t\), and \(0 < \rho_t < 1\).

This is simple and efficient, but it ignores token difficulty.

### 4.2 Adaptive Schedule

Adaptive schedules let the model decide which positions should keep evolving. Instead of only following timestep \(t\), the decoder also uses confidence or uncertainty signals.

Conceptually:

\[
\text{progress of position } i
=
f(t, c_i, \text{context sensitivity}, \dots).
\]

This leads to better compute allocation:

- easy positions finish early;
- hard positions get more refinement budget.

## 5. Semi-Autoregressive and Block Decoding

Diffusion decoding does not have to be fully parallel.

### 5.1 Blockwise Generation

Some methods divide the sequence into blocks and refine one block earlier than another. This gives a middle ground between:

- full autoregressive decoding;
- full parallel masked decoding.

If blocks are indexed by \(b = 1, \dots, B\), then a simplified view is:

\[
p(x) \approx \prod_{b=1}^{B} p_\theta(x^{(b)} \mid x^{(<b)}, \text{noisy state}).
\]

This is not standard AR factorization, because each block may still be internally denoised by a diffusion process.

### 5.2 Why This Matters

Blockwise decoding can improve:

- long-range consistency;
- controllability;
- speed-quality tradeoffs.

It is one reason methods such as `SSD-LM` are worth separating from pure masked diffusion.

## 6. AR-Diffusion and Position-Dependent Denoising

`AR-Diffusion` is an important case because it explicitly reintroduces left-to-right asymmetry.

### 6.1 Core Idea

Different positions receive different denoising budgets. Tokens on the left are allowed to emerge earlier, so that tokens on the right can condition on them later.

In simplified form, position \(i\) may have its own effective denoising horizon:

\[
T_i \le T_j \quad \text{for } i < j.
\]

So left positions become clean sooner than right positions.

### 6.2 Interpretation

This does not fully return to autoregressive decoding. Instead, it adds causal bias inside a diffusion process:

\[
\text{diffusion refinement}
+ \text{position-dependent generation order}.
\]

This can preserve some diffusion flexibility while respecting the sequential structure of language more than fully synchronous decoders do.

## 7. Sampling Budget and Quality

DLLM decoding quality is strongly tied to the number of refinement steps.

### 7.1 Compute-Quality Tradeoff

Let \(K\) be the number of decoding iterations. Then in broad terms:

\[
K \uparrow \Rightarrow \text{higher compute, often better quality}.
\]

But returns are not linear. Past some point, extra steps may provide only marginal gains.

### 7.2 Limited-Budget Failure Mode

With too few steps:

- global structure may be unstable;
- early token commitments may be wrong;
- the model may produce shallow or generic completions.

This is why recent decoding work often focuses not on changing training, but on getting better quality from the same pretrained model under a limited inference budget.

## 8. Recent Decoding Directions

Several recent directions are worth tracking.

### 8.1 Continuous or Flow-Like Decoding

Recent work such as **Masked Diffusion Decoding as x-Prediction Flow** (June 27, 2026) argues that standard binary mask decisions are too rigid. Instead of forcing a position to be either fully committed or fully masked, it interprets clean-token prediction as a continuous refinement signal in embedding space. This points toward smoother, token-wise asynchronous decoding. [[Paper](https://arxiv.org/abs/2606.29066)]

### 8.2 Context-Robust Remasking

Recent work such as **CoRe: Context-Robust Remasking for Diffusion Language Models** (February 4, 2026) questions whether naive confidence scores are reliable enough for revision. The main idea is to test whether a token stays stable when its surrounding context is perturbed, then prioritize brittle tokens for remasking. [[Paper](https://arxiv.org/abs/2602.04096)]

These directions are important because they show that DLLM progress is increasingly happening at inference time, not only in training objectives.

## 9. Practical Comparison with AR Decoding

Autoregressive decoding is usually:

- easier to reason about;
- more stable under standard beam or sampling recipes;
- naturally aligned with next-token training.

DLLM decoding is usually:

- more parallel;
- more revision-friendly;
- more sensitive to schedule design;
- more sensitive to confidence estimation and remasking policy.

So in practice:

\[
\text{AR decoding optimizes token order},
\qquad
\text{DLLM decoding optimizes refinement order}.
\]

## 10. What to Look For in a DLLM Paper

When reading a new DLLM paper, ask:

1. Is decoding fully parallel, semi-autoregressive, or position-asynchronous?
2. Does the model permanently commit tokens, or can it revise them?
3. Is the denoising schedule fixed or adaptive?
4. Is there a separate confidence or uncertainty estimator?
5. How many decoding steps are required for good quality?
6. Are improvements coming from training, inference, or both?

Those questions usually tell you more about practical usability than the headline architecture alone.
