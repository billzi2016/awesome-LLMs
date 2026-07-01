# Training Objectives in Diffusion LLMs

This document focuses on how Diffusion LLMs are trained. The central point is that the phrase "Diffusion LLM" does not refer to one single loss. Instead, it refers to a family of objectives built around a corruption process and a learned reverse model.

At a high level:

\[
\text{DLLM training}
=
\text{forward corruption}
+ \text{reverse prediction target}
+ \text{loss definition}
+ \text{sampling-time interpretation}.
\]

Different papers mainly differ in those four choices.

## 1. The Generic Training Template

Let \(x_0\) be a clean text sequence drawn from the data distribution. A diffusion-style training objective usually samples a corruption level \(t\), constructs a noisy version \(x_t\), and trains a model to predict some target related to \(x_0\), \(x_{t-1}\), or a score function.

The generic form is:

\[
x_0 \sim q(x_0), \qquad
t \sim p(t), \qquad
x_t \sim q(x_t \mid x_0),
\]

then minimize

\[
\mathcal{L} =
\mathbb{E}_{x_0, t, x_t}
\left[
\ell_\theta(x_0, x_t, t)
\right].
\]

The main question is what exactly \( \ell_\theta \) should be.

## 2. Continuous Diffusion Losses

Continuous DLLMs usually operate in embedding or latent space.

### 2.1 Noise Prediction

If the forward process is

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon, \qquad \epsilon \sim \mathcal{N}(0,I),
\]

then one common objective is to predict the injected noise:

\[
\mathcal{L}_{\epsilon}
=
\mathbb{E}_{z_0,\epsilon,t}
\left[
\|\epsilon - \epsilon_\theta(z_t,t)\|_2^2
\right].
\]

This is the standard diffusion training pattern adapted from image models.

### 2.2 Clean-State Prediction

Instead of predicting noise, the model can predict the clean latent:

\[
\mathcal{L}_{x_0}
=
\mathbb{E}_{z_0,\epsilon,t}
\left[
\|z_0 - \hat{z}_{0,\theta}(z_t,t)\|_2^2
\right].
\]

The choice between noise prediction and clean-state prediction affects:

- optimization stability;
- interpretation of the reverse process;
- how decoding heuristics are designed later.

### 2.3 Self-Conditioning

Some continuous models feed back a previous estimate of the clean state:

\[
\hat{z}_0^{(t)} \rightarrow \text{model input at later refinement steps}.
\]

This does not define a new forward process, but it changes the effective training dynamics by teaching the model to iteratively refine its own guess.

## 3. Discrete Diffusion Objectives

When text is modeled directly in token space, the loss changes.

### 3.1 Transition-Based Learning

Suppose the forward process is a discrete Markov chain:

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

Then one natural goal is to learn reverse transitions:

\[
p_\theta(x_{t-1} \mid x_t).
\]

The training objective often becomes a cross-entropy over token transitions or clean-token reconstruction terms implied by the reverse chain.

### 3.2 Clean-Token Reconstruction

A practical simplification is to directly predict the original token \(x_0\) from a noisy token state:

\[
\mathcal{L}_{\mathrm{disc}}
=
\mathbb{E}_{x_0,t,x_t}
\left[
- \sum_i \log p_\theta(x_{0,i} \mid x_t, t)
\right].
\]

In practice, this may be restricted to only those positions that were corrupted.

This objective is simple, but it only works well if the corruption process and parameterization line up correctly with inference-time decoding.

## 4. Masked Diffusion Objectives

Masked diffusion is a particularly important discrete specialization.

### 4.1 Absorbing-State Corruption

Each token is gradually replaced by a special mask token \(m\):

\[
q(x_t \mid x_0)
=
\mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr).
\]

This means the forward process preserves some observed context while hiding other parts.

### 4.2 Masked Reconstruction Loss

The most common masked diffusion objective is:

\[
\mathcal{L}_{\mathrm{mask}}
=
\mathbb{E}_{x_0,t,x_t}
\left[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right],
\]

where \(M_t\) is the set of masked positions at noise level \(t\).

This objective is central to methods such as MDLM and LLaDA.

### 4.3 Weighting Across Noise Levels

Not all timesteps contribute equally. Some works implicitly or explicitly reweight training across different \(t\):

\[
\mathcal{L}
=
\mathbb{E}_{x_0,t,x_t}
\left[
w(t)\cdot
\left(
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t)
\right)
\right].
\]

This matters because:

- low-noise examples teach local repair;
- high-noise examples teach global generation from weak context.

The schedule \(w(t)\) affects which of those behaviors the model learns best.

## 5. Likelihood-Based Objectives

Some DLLM work emphasizes likelihood more directly than denoising quality.

### 5.1 Reverse-Chain Likelihood View

Let the model define:

\[
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1}\mid x_t).
\]

Then:

\[
p_\theta(x_0) = \sum_{x_{1:T}} p_\theta(x_{0:T}).
\]

This marginal is usually intractable, so training optimizes a bound or surrogate.

### 5.2 ELBO-Style Objective

A standard variational form is:

\[
\mathcal{L}_{\mathrm{ELBO}}
=
\mathbb{E}_{q(x_{1:T}\mid x_0)}
\left[
\log \frac{q(x_{1:T}\mid x_0)}{p_\theta(x_{0:T})}
\right].
\]

Equivalently,

\[
-\log p_\theta(x_0)
\le
\mathcal{L}_{\mathrm{ELBO}}.
\]

This interpretation is important because it connects diffusion training to classical likelihood-based language modeling rather than only to denoising heuristics.

### 5.3 Why Language Modeling Cares

For language models, likelihood matters because it is tied to:

- perplexity;
- scaling-law analysis;
- calibration;
- principled comparison with autoregressive models.

This is why likelihood-oriented DLLM work is conceptually important even when decoding quality is the user-facing metric.

## 6. Score-Based Objectives in Discrete Space

Discrete diffusion cannot directly reuse the continuous score matching machinery.

### 6.1 Ratio-Based Score

SEDD introduces a score over discrete states via probability ratios:

\[
s(x,t)_y = \frac{q_t(y)}{q_t(x)}.
\]

The model learns

\[
s_\theta(x,t)_y \approx s(x,t)_y.
\]

### 6.2 Why This Matters

Instead of forcing continuous-style score estimation into a discrete space, this formulation uses an object that is native to the discrete distribution itself.

This changes both:

- the training target;
- the geometry of the optimization problem.

Conceptually, score-entropy methods say:

\[
\text{better discrete objective}
\Rightarrow
\text{better reverse process learning}.
\]

## 7. Conditional and Seq2Seq Objectives

Not every DLLM is trained for unconditional generation.

### 7.1 Conditional Diffusion

Given conditioning input \(c\), the objective becomes:

\[
\mathcal{L}_{\mathrm{cond}}
=
\mathbb{E}_{x_0,c,t,x_t}
\left[
\ell_\theta(x_0, x_t, c, t)
\right].
\]

This covers:

- summarization;
- translation;
- paraphrasing;
- question generation;
- dialogue generation.

### 7.2 Encoder-Decoder Diffusion

In models such as DiffuSeq or SeqDiffuSeq, the condition \(c\) is encoded by an encoder, while the output sequence is denoised through a diffusion decoder.

So the training target is no longer just "recover clean text", but:

\[
\text{recover clean target text conditioned on source input}.
\]

This brings DLLMs closer to traditional seq2seq learning than to pure language modeling.

## 8. Post-Training and Instruction Finetuning

Large modern DLLMs are often not judged by pretraining loss alone.

### 8.1 Supervised Finetuning

Given instruction-response pairs \((u, y)\), one can continue training a masked diffusion model using:

\[
\mathcal{L}_{\mathrm{SFT}}
=
\mathbb{E}_{u,y,t,y_t}
\left[
- \sum_{i \in M_t} \log p_\theta(y_i \mid y_t, u, t)
\right].
\]

This is the diffusion analogue of instruction tuning in autoregressive LLMs.

### 8.2 Why SFT Changes the Story

Pretraining only answers whether the model can learn a good reverse process on raw text.

SFT asks a different question:

\[
\text{can the same reverse process become useful for task following?}
\]

That shift is central for models such as LLaDA.

## 9. What Changes the Most in Practice

When practitioners compare DLLM training objectives, the most important differences are often:

1. what is being predicted: noise, clean latent, clean token, or discrete score;
2. what corruption family is used: Gaussian, structured discrete, or absorbing-state mask;
3. how timesteps are sampled or weighted;
4. whether the model is unconditional, conditional, or instruction-tuned;
5. how closely the training objective matches the intended decoding strategy.

The last point is particularly important:

\[
\text{good training loss}
\neq
\text{good generation quality}
\]

unless the loss prepares the model for the kind of decoding it will actually face.

## 10. A Practical Reading Map

To understand training objectives progressively:

1. read `Diffusion-LM` for continuous latent denoising;
2. read `D3PM` for discrete-state corruption;
3. read `Likelihood-Based Diffusion Language Models` for ELBO / likelihood emphasis;
4. read `SEDD` for discrete score-entropy training;
5. read `MDLM` for simplified masked diffusion objectives;
6. read `LLaDA` for large-scale masked diffusion plus instruction finetuning.

## 11. Minimal Summary

If you want the shortest useful summary, it is this:

\[
\text{DLLM objective}
=
\text{learn how to reverse text corruption}.
\]

But the field differs on:

- what corruption means;
- what the reverse target is;
- how likelihood is interpreted;
- how decoding will later use the trained model.

Those choices are what separate one DLLM family from another.
