# Method Family Map of Diffusion LLMs

This document organizes major Diffusion LLM methods into a small number of coherent families. The point is not only to list papers, but to show why they differ.

At a high level, most DLLM methods can be classified by four questions:

1. What space is diffused: continuous latent space, discrete token space, or masked token space?
2. What is the training target: noise, clean state, reverse transition, or discrete score?
3. What is the decoding style: fully parallel, iterative remasking, semi-autoregressive, or position-biased refinement?
4. What is the main use case: unconditional LM, controllable generation, seq2seq generation, or large-scale instruction following?

## 1. One-Line Global Picture

A useful global picture is:

\[
\text{DLLM landscape}
=
\text{continuous family}
+ \text{discrete family}
+ \text{masked family}
+ \text{conditional / seq2seq family}
+ \text{hybrid decoding family}.
\]

These families overlap, but they are still useful because each one makes a different core design choice.

## 2. Continuous Latent-Space Family

This family performs diffusion in embeddings or latent vectors rather than directly in tokens.

### 2.1 Main Representatives

- `Diffusion-LM`
- `Self-conditioned Embedding Diffusion`
- `GENIE`
- `Latent Diffusion for Language Generation`

### 2.2 Core Design

The clean object is represented as \(z_0 \in \mathbb{R}^d\), and corruption is typically Gaussian:

\[
z_t = \alpha_t z_0 + \sigma_t \epsilon.
\]

The model then predicts:

- injected noise;
- clean latent state;
- or another denoising target in continuous space.

### 2.3 Strengths

- mathematically close to classic image diffusion;
- convenient optimization tools;
- natural support for controllable generation in latent space.

### 2.4 Weaknesses

- mismatch between continuous denoising and discrete text output;
- final projection back to tokens can be fragile;
- scaling to very strong language modeling is less straightforward than in masked-token formulations.

### 2.5 When to Think of This Family

Use this family as the starting point when a paper:

- emphasizes latent geometry;
- borrows heavily from image-style diffusion;
- discusses controllability through latent optimization.

## 3. General Discrete Diffusion Family

This family performs diffusion directly in token space rather than in continuous latent space.

### 3.1 Main Representatives

- `D3PM`
- `SEDD`
- `A Reparameterized Discrete Diffusion Model for Text Generation`

### 3.2 Core Design

The forward process is a token-level Markov chain:

\[
q(x_t \mid x_{t-1}) = \mathrm{Cat}(x_t; Q_t x_{t-1}).
\]

The reverse model learns either:

- token transition distributions;
- clean-token reconstruction;
- or discrete score-like objects.

### 3.3 Why This Family Matters

This family removes the continuous-discrete mismatch and gives a more native account of text tokens.

Its central challenge is that discrete diffusion is harder to optimize than Gaussian diffusion. That is why methods like SEDD are important: they improve the training target itself.

### 3.4 How It Differs from Masked Diffusion

Masked diffusion is a special case of discrete diffusion, but not the whole story.

General discrete diffusion may:

- replace tokens with structured alternatives;
- use richer transition matrices;
- avoid reducing corruption to only one absorbing mask state.

## 4. Masked Diffusion Family

This is the most important modern family for large-scale DLLMs.

### 4.1 Main Representatives

- `DiffusionBERT`
- `MDLM`
- `LLaDA`

### 4.2 Core Design

The forward process gradually replaces tokens with `[MASK]`:

\[
q(x_t \mid x_0)
=
\mathrm{Cat}\bigl(x_t; \alpha_t x_0 + (1-\alpha_t)m \bigr).
\]

The model predicts original tokens at masked positions:

\[
- \sum_{i \in M_t} \log p_\theta(x_{0,i} \mid x_t, t).
\]

### 4.3 Why It Became Dominant

Masked diffusion is attractive because it:

- stays in token space;
- fits bidirectional Transformers naturally;
- supports parallel refinement;
- aligns well with infilling, editing, and revision.

### 4.4 Internal Differences

Even inside this family, papers differ on:

- timestep weighting;
- decoding schedule;
- remasking policy;
- scaling recipe;
- post-training strategy.

For example:

- `DiffusionBERT` is closer to the idea of improving generative masked modeling through diffusion;
- `MDLM` emphasizes a simplified but strong masked diffusion objective;
- `LLaDA` pushes masked diffusion to large-scale pretraining and instruction tuning.

### 4.5 When to Think of This Family

This is the right default mental bucket when a method:

- starts from all-mask or high-mask states;
- relies on iterative unmasking;
- uses masked reconstruction as the main objective;
- claims large-scale LLM-like behavior.

## 5. Conditional and Seq2Seq Diffusion Family

This family focuses on generating target text conditioned on an input sequence.

### 5.1 Main Representatives

- `DiffuSeq`
- `SeqDiffuSeq`
- `DINOISER`
- `Diffusion-NAT`

### 5.2 Core Design

The training problem is not simply:

\[
\text{generate text}.
\]

It is:

\[
\text{generate target text conditioned on source text}.
\]

So the model typically includes:

- an encoder for the condition \(c\);
- a diffusion-style decoder for the output sequence.

### 5.3 What Makes This Family Special

Compared with unconditional DLLMs, these methods are more tightly tied to classical NLP tasks:

- summarization;
- translation;
- dialogue;
- paraphrasing;
- question generation.

### 5.4 Where It Sits in the Map

This family is not mainly about competing with next-token LLMs on open-ended continuation. It is more about replacing or extending seq2seq generation with diffusion-style decoding.

## 6. Semi-Autoregressive and Hybrid Decoding Family

Some methods do not fit cleanly into either full parallel diffusion or standard AR generation.

### 6.1 Main Representatives

- `SSD-LM`
- `AR-Diffusion`

### 6.2 Core Design

These methods explicitly reintroduce ordering structure into diffusion.

For example:

- `SSD-LM` uses simplex-based diffusion with semi-autoregressive block generation;
- `AR-Diffusion` gives different positions different effective denoising schedules.

### 6.3 Why This Family Exists

Pure parallel diffusion is attractive computationally, but language is not image-like. Token order matters strongly.

So this family asks:

\[
\text{can we keep diffusion-style refinement while restoring some sequential bias?}
\]

### 6.4 Main Tradeoff

This family usually sacrifices some pure parallelism in exchange for:

- better coherence;
- stronger left-to-right dependency modeling;
- more controllable generation order.

## 7. Family-by-Family Method Placement

Below is a compact placement map.

### 7.1 Diffusion-LM

- Family: continuous latent-space
- Main idea: continuous diffusion over latent word representations
- Best viewed as: controllable generation oriented

### 7.2 GENIE

- Family: continuous latent-space
- Main idea: pretrained diffusion LM with continuous paragraph denoising
- Best viewed as: pretraining-oriented latent diffusion

### 7.3 Latent Diffusion for Language Generation

- Family: continuous latent-space
- Main idea: diffusion over autoencoded language latents
- Best viewed as: latent compression plus diffusion

### 7.4 D3PM

- Family: general discrete diffusion
- Main idea: structured discrete-state corruption
- Best viewed as: foundational discrete framework

### 7.5 SEDD

- Family: general discrete diffusion
- Main idea: score-entropy objective for discrete diffusion
- Best viewed as: objective-level improvement for token-space diffusion

### 7.6 DiffusionBERT

- Family: masked diffusion
- Main idea: use BERT-like generative masking with diffusion interpretation
- Best viewed as: bridge between MLM-style modeling and generative diffusion

### 7.7 MDLM

- Family: masked diffusion
- Main idea: simplified masked diffusion objective with strong scaling behavior
- Best viewed as: practical masked diffusion baseline

### 7.8 LLaDA

- Family: masked diffusion
- Main idea: large-scale masked diffusion LLM with SFT
- Best viewed as: modern large-scale DLLM system

### 7.9 DiffuSeq / SeqDiffuSeq

- Family: conditional and seq2seq diffusion
- Main idea: diffusion decoder for source-conditioned target generation
- Best viewed as: seq2seq diffusion line

### 7.10 DINOISER / Diffusion-NAT

- Family: conditional and seq2seq diffusion
- Main idea: improve conditional generation through noise manipulation or self-prompted denoising
- Best viewed as: task-oriented conditional diffusion

### 7.11 SSD-LM

- Family: semi-autoregressive and hybrid decoding
- Main idea: simplex diffusion plus modular blockwise generation
- Best viewed as: hybrid between diffusion and semi-AR generation

### 7.12 AR-Diffusion

- Family: semi-autoregressive and hybrid decoding
- Main idea: position-dependent denoising order
- Best viewed as: diffusion with explicit sequential bias

## 8. A More Structural Comparison

One useful comparison table in words:

- Continuous family: strongest connection to classic diffusion mathematics, weakest direct alignment with token discreteness.
- Discrete family: strongest token-level probabilistic alignment, hardest optimization.
- Masked family: best practical fit for large bidirectional Transformer-style DLLMs.
- Seq2seq family: best fit for source-conditioned NLP tasks.
- Hybrid decoding family: best fit when one wants diffusion flexibility without giving up generation order structure completely.

## 9. How to Classify a New Paper

When a new DLLM paper appears, classify it in this order:

1. What space is corrupted?
2. Is the corruption generic discrete, absorbing-mask, or continuous Gaussian?
3. Is the task unconditional LM, conditional generation, or instruction following?
4. Is decoding fully parallel, remasking-based, blockwise, or position-biased?
5. Is the novelty mainly in the loss, the corruption process, the scaling recipe, or the decoder?

Usually, these five questions are enough to locate the paper in the method map.

## 10. Minimal Summary

The shortest useful summary is:

\[
\text{most DLLM papers differ more in training space and decoding design than in headline branding}.
\]

That is why organizing methods into families is more informative than keeping them as a flat list.
