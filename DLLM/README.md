# Diffusion LLM

[中文](./README.zh-CN.md) | [Contributing](../CONTRIBUTING.md) | [Curation Policy](../docs/curation-policy.md)

A curated, structured, and bilingual awesome list for Diffusion LLMs.

> In this repository, `DLLM` means `Diffusion LLM`, not dynamic large language models or other unrelated abbreviations.

## 1. Scope

This repository focuses on language modeling work that uses diffusion-style generative processes for text generation, reasoning, editing, planning, or closely related language tasks.

It aims to organize resources around:

- diffusion-based language modeling formulations;
- discrete diffusion and continuous diffusion methods for language;
- training, sampling, and decoding strategies for diffusion LLMs;
- alignment, controllability, and post-training methods built for diffusion language models;
- evaluation, benchmarks, libraries, and representative applications.

This repository does not aim to cover:

- generic diffusion model resources with no clear language focus;
- standard autoregressive LLM papers unless they are directly used for comparison or hybrid modeling;
- general multimodal diffusion work unless the language-modeling component is central;
- broad text generation surveys that do not meaningfully discuss diffusion language modeling.

## 2. Repository Goals

This project is intended to be:

- a high-signal entry point for researchers and engineers interested in Diffusion LLMs;
- a structured map rather than an unfiltered link dump;
- a bilingual resource with aligned English and Chinese documentation;
- a maintainable repository with explicit inclusion boundaries.

## 3. Navigation

- [README.zh-CN.md](./README.zh-CN.md): Simplified Chinese version
- [CONTRIBUTING.md](../CONTRIBUTING.md): contribution guide
- [CONTRIBUTING.zh-CN.md](../CONTRIBUTING.zh-CN.md): contribution guide in Chinese
- [../docs/curation-policy.md](../docs/curation-policy.md): inclusion and rejection rules
- [../docs/curation-policy.zh-CN.md](../docs/curation-policy.zh-CN.md): Chinese curation policy
- [../docs/link-check.md](../docs/link-check.md): link verification notes
- [docs/mathematical-foundations.md](./docs/mathematical-foundations.md): mathematical notes on continuous, discrete, and masked diffusion language modeling
- [docs/mathematical-foundations.zh-CN.md](./docs/mathematical-foundations.zh-CN.md): Chinese mathematical notes
- [docs/sampling-and-decoding.md](./docs/sampling-and-decoding.md): decoding strategies, remasking, and inference-time tradeoffs
- [docs/sampling-and-decoding.zh-CN.md](./docs/sampling-and-decoding.zh-CN.md): Chinese decoding notes
- [docs/training-objectives.md](./docs/training-objectives.md): training losses, likelihood views, and post-training variants
- [docs/training-objectives.zh-CN.md](./docs/training-objectives.zh-CN.md): Chinese training-objective notes
- [docs/method-family-map.md](./docs/method-family-map.md): method families and how representative DLLMs differ
- [docs/method-family-map.zh-CN.md](./docs/method-family-map.zh-CN.md): Chinese method-family map
- [docs/benchmark-and-evaluation.md](./docs/benchmark-and-evaluation.md): evaluation dimensions, fairness issues, and decoding-budget analysis
- [docs/benchmark-and-evaluation.zh-CN.md](./docs/benchmark-and-evaluation.zh-CN.md): Chinese evaluation notes
- [docs/overview.md](./docs/overview.md): reading order and navigation across the technical notes
- [docs/overview.zh-CN.md](./docs/overview.zh-CN.md): Chinese overview and reading guide
- [docs/timeline.md](./docs/timeline.md): development trajectory from early discrete diffusion to large-scale DLLMs
- [docs/timeline.zh-CN.md](./docs/timeline.zh-CN.md): Chinese DLLM timeline
- [docs/glossary.md](./docs/glossary.md): terminology reference across the repository
- [docs/glossary.zh-CN.md](./docs/glossary.zh-CN.md): Chinese DLLM glossary
- [../docs/resource-template.md](../docs/resource-template.md): suggested resource entry template

## 4. Layered Structure

The repository will be organized from broad context to concrete implementation.

### 4.1 Overview and Surveys

This layer collects:

- surveys of diffusion language modeling;
- position papers or taxonomies for non-autoregressive and diffusion-based text generation;
- tutorial-style materials that help readers build a high-level picture of the field.

Purpose:

- help new readers understand the problem setting, terminology, and research landscape before reading individual papers.

### 4.2 Foundational Papers

This layer collects:

- early or representative papers that define core diffusion language modeling formulations;
- papers that establish major training objectives, generation processes, or theoretical views;
- work commonly used as the baseline reference for later DLLM papers.

Purpose:

- anchor the field around core ideas rather than isolated implementations.

### 4.3 Training Paradigms

This layer collects resources about how Diffusion LLMs are trained, such as:

- discrete token diffusion;
- continuous latent or embedding-space diffusion for language;
- masked denoising variants closely tied to diffusion formulations;
- objective design, noise schedules, timestep parameterization, and optimization strategies;
- distillation or acceleration methods directly used in DLLM training.

Purpose:

- show the main technical choices behind model construction.

### 4.4 Inference, Sampling, and Decoding

This layer collects resources about:

- iterative denoising and generation procedures;
- sampling acceleration;
- speculative or parallel decoding methods for diffusion language models;
- controllable generation and editing during inference;
- efficiency-quality tradeoffs at generation time.

Purpose:

- separate model-definition questions from inference-time engineering and algorithm design.

### 4.5 Alignment and Post-Training

This layer collects:

- preference optimization methods adapted to diffusion language models;
- instruction tuning, alignment, safety, and controllability methods designed for DLLMs;
- post-training strategies that improve helpfulness, stability, or task performance.

Purpose:

- track whether diffusion LLMs can support the same practical post-training stack expected from modern LLM systems.

### 4.6 Evaluation and Benchmarks

This layer collects:

- benchmark suites used to compare DLLMs with autoregressive models;
- evaluation methods for fluency, consistency, reasoning, controllability, efficiency, and diversity;
- analysis papers on failure modes, scaling behavior, and robustness.

Purpose:

- make comparison criteria explicit instead of mixing claims from unrelated settings.

### 4.7 Libraries, Repositories, and Implementations

This layer collects:

- official implementations;
- maintained third-party codebases with clear technical value;
- reproducible training or inference frameworks for diffusion language models;
- code resources that make the field easier to study or extend.

Purpose:

- help readers move from papers to runnable systems.

### 4.8 Applications

This layer collects application-oriented work where diffusion LLMs are central to the method, such as:

- text generation;
- reasoning;
- editing or refinement;
- planning;
- domain-specific language generation tasks.

Purpose:

- show where DLLMs are actually being used, beyond methodological discussion.

## 5. Curated Resources

### 5.1 Surveys and Reading Lists

- **Diffusion Models for Non-autoregressive Text Generation: A Survey** - Li et al., IJCAI 2023. A focused survey on diffusion methods for non-autoregressive text generation, including formulation, optimization, and design space. [[Paper](https://arxiv.org/abs/2303.06574)]
- [RUCAIBox/Awesome-Text-Diffusion-Models](https://github.com/RUCAIBox/Awesome-Text-Diffusion-Models) - Official companion repository for the survey above, useful as a broader reading list for text diffusion papers.
- **Promises, Outlooks and Challenges of Diffusion Language Modeling** - Deschenaux and Gulcehre, 2024. An analysis-oriented paper discussing strengths, limitations, and open questions of diffusion language modeling. [[Paper](https://arxiv.org/abs/2406.11473)]

### 5.2 Foundational Papers

- **Structured Denoising Diffusion Models in Discrete State-Spaces** - Austin et al., NeurIPS 2021. Introduces D3PMs, a core discrete diffusion framework that strongly influenced later text diffusion work. [[Paper](https://arxiv.org/abs/2107.03006)]
- **Diffusion-LM Improves Controllable Text Generation** - Li et al., NeurIPS 2022. A representative continuous diffusion language model that emphasizes fine-grained controllable generation. [[Paper](https://arxiv.org/abs/2205.14217)]
- **Self-conditioned Embedding Diffusion for Text Generation** - Strudel et al., 2022. Studies continuous diffusion directly in embedding space for conditional and unconditional text generation. [[Paper](https://arxiv.org/abs/2211.04236)]
- **SSD-LM: Semi-autoregressive Simplex-based Diffusion Language Model for Text Generation and Modular Control** - Han et al., 2022. Proposes simplex-space diffusion with semi-autoregressive block generation and modular controllability. [[Paper](https://arxiv.org/abs/2210.17432)]
- **Likelihood-Based Diffusion Language Models** - Gulrajani and Hashimoto, 2023. Focuses on maximum-likelihood training and scaling behavior for diffusion language models, including Plaid 1B. [[Paper](https://arxiv.org/abs/2305.18619)]

### 5.3 Training Paradigms

- **Text Generation with Diffusion Language Models: A Pre-training Approach with Continuous Paragraph Denoise** - Lin et al., 2022. Introduces GENIE, a pre-trained diffusion language model with a continuous paragraph denoising objective. [[Paper](https://arxiv.org/abs/2212.11685)] [[Code](https://github.com/microsoft/ProphetNet/tree/master/GENIE)]
- **DiffuSeq: Sequence to Sequence Text Generation with Diffusion Models** - Gong et al., ICLR 2023. A sequence-to-sequence diffusion framework for conditional text generation with strong diversity results. [[Paper](https://arxiv.org/abs/2210.08933)] [[Code](https://github.com/Shark-NLP/DiffuSeq)]
- **SeqDiffuSeq: Text Diffusion with Encoder-Decoder Transformers** - Yuan et al., 2022. An encoder-decoder diffusion model for seq2seq text generation with self-conditioning and adaptive noise scheduling. [[Paper](https://arxiv.org/abs/2212.10325)]
- **Latent Diffusion for Language Generation** - Lovelace et al., 2022. Learns diffusion in the latent space of a language autoencoder, treating diffusion and pretrained language models as complementary. [[Paper](https://arxiv.org/abs/2212.09462)]
- **DiffusionBERT: Improving Generative Masked Language Models with Diffusion Models** - He et al., 2022. Uses BERT initialization to learn the reverse process of absorbing-state diffusion for text generation. [[Paper](https://arxiv.org/abs/2211.15029)]
- **A Reparameterized Discrete Diffusion Model for Text Generation** - Zheng et al., 2023. Rewrites discrete diffusion sampling with an equivalent reparameterized formulation for more effective training and decoding. [[Paper](https://arxiv.org/abs/2302.05737)]
- **DINOISER: Diffused Conditional Sequence Learning by Manipulating Noises** - Ye et al., 2023. Improves conditional diffusion sequence modeling through adaptive noise-range design in both training and inference. [[Paper](https://arxiv.org/abs/2302.10025)]
- **Discrete Diffusion Modeling by Estimating the Ratios of the Data Distribution** - Lou et al., ICML 2024. Introduces SEDD and score-entropy training for discrete diffusion, significantly improving language modeling quality. [[Paper](https://arxiv.org/abs/2310.16834)]
- **Simple and Effective Masked Diffusion Language Models** - Sahoo et al., NeurIPS 2024. Shows that masked discrete diffusion with a simplified objective can close much of the gap to autoregressive language modeling. [[Paper](https://arxiv.org/abs/2406.07524)] [[Project](https://s-sahoo.com/mdlm/)]

### 5.4 Inference, Sampling, and Decoding

- **AR-Diffusion: Auto-Regressive Diffusion Model for Text Generation** - Wu et al., 2023. Combines left-to-right dependency with diffusion denoising to improve generation quality and speed. [[Paper](https://arxiv.org/abs/2305.09515)] [[Code](https://github.com/microsoft/ProphetNet/tree/master/AR-diffusion)]
- **DiffusER: Discrete Diffusion via Edit-based Reconstruction** - Reid et al., 2022. Frames generation as iterative edit-based reconstruction, making revision-oriented generation a first-class use case. [[Paper](https://arxiv.org/abs/2210.16886)]
- **Diffusion-NAT: Self-Prompting Discrete Diffusion for Non-Autoregressive Text Generation** - Zhou et al., 2023. Combines discrete diffusion with BART-style denoising and iterative self-prompting for text-to-text generation. [[Paper](https://arxiv.org/abs/2305.04044)]
- **Simple and Effective Masked Diffusion Language Models** - Sahoo et al., NeurIPS 2024. Includes efficient samplers and semi-autoregressive generation strategies for arbitrary-length text. [[Paper](https://arxiv.org/abs/2406.07524)] [[Project](https://s-sahoo.com/mdlm/)]

### 5.5 Alignment and Post-Training

- **Diffusion Language Models Can Perform Many Tasks with Scaling and Instruction-Finetuning** - Ye et al., 2023. Studies scaling, task finetuning, and instruction finetuning for diffusion language models across multiple tasks. [[Paper](https://arxiv.org/abs/2308.12219)]
- **Large Language Diffusion Models** - Nie et al., 2025. Introduces LLaDA, an 8B masked diffusion LLM trained from scratch and then supervised fine-tuned for instruction following. [[Paper](https://arxiv.org/abs/2502.09992)] [[Project](https://ml-gsai.github.io/LLaDA-demo/)] [[Code](https://github.com/ML-GSAI/LLaDA)]

### 5.6 Evaluation and Benchmarks

- **Promises, Outlooks and Challenges of Diffusion Language Modeling** - Deschenaux and Gulcehre, 2024. Evaluates SEDD-style diffusion LMs against autoregressive baselines and highlights practical tradeoffs. [[Paper](https://arxiv.org/abs/2406.11473)]
- **Diffusion Language Models: An Experimental Analysis** - Bertolani et al., 2026. A comparative study of modern diffusion language models across multiple benchmarks, inference budgets, and generation settings. [[Paper](https://arxiv.org/abs/2606.19475)]
- [LLaDA Evaluation Code](https://github.com/ML-GSAI/LLaDA) - Includes evaluation scripts and OpenCompass-related resources that are useful for reproducing DLLM benchmarking workflows.

### 5.7 Libraries, Repositories, and Implementations

- [Shark-NLP/DiffuSeq](https://github.com/Shark-NLP/DiffuSeq) - Official implementation of DiffuSeq and DiffuSeq-v2 for conditional diffusion text generation.
- [microsoft/ProphetNet: GENIE](https://github.com/microsoft/ProphetNet/tree/master/GENIE) - Official GENIE implementation for pretrained diffusion language modeling.
- [microsoft/ProphetNet: AR-diffusion](https://github.com/microsoft/ProphetNet/tree/master/AR-diffusion) - Official AR-Diffusion implementation for dependency-aware diffusion decoding.
- [ML-GSAI/LLaDA](https://github.com/ML-GSAI/LLaDA) - Official PyTorch implementation for LLaDA and follow-up diffusion LLM evaluation and inference utilities.
- [RUCAIBox/Awesome-Text-Diffusion-Models](https://github.com/RUCAIBox/Awesome-Text-Diffusion-Models) - Useful companion repository for tracking earlier text diffusion papers across continuous and discrete formulations.

### 5.8 Applications

- **Diffusion-LM Improves Controllable Text Generation** - Li et al., NeurIPS 2022. A strong reference for attribute and structure control in text generation. [[Paper](https://arxiv.org/abs/2205.14217)]
- **DiffusER: Discrete Diffusion via Edit-based Reconstruction** - Reid et al., 2022. Useful for editing, revision, and prototype-conditioned text generation scenarios. [[Paper](https://arxiv.org/abs/2210.16886)]
- **DiffuSeq: Sequence to Sequence Text Generation with Diffusion Models** - Gong et al., ICLR 2023. Covers conditional tasks such as dialogue, question generation, paraphrasing, and text simplification. [[Paper](https://arxiv.org/abs/2210.08933)] [[Code](https://github.com/Shark-NLP/DiffuSeq)]
- **Diffusion-NAT: Self-Prompting Discrete Diffusion for Non-Autoregressive Text Generation** - Zhou et al., 2023. A representative text-to-text generation application of discrete diffusion plus pretrained denoising models. [[Paper](https://arxiv.org/abs/2305.04044)]

Not every paper about diffusion and text is included here. This first batch prioritizes papers and repositories that help define the core DLLM landscape.

## 6. Inclusion Principles

Resources should generally have at least one of the following:

- a paper with a stable source;
- an official repository;
- a useful benchmark or dataset page;
- documentation or technical notes with clear relevance to Diffusion LLMs.

For detailed inclusion rules, see [docs/curation-policy.md](../docs/curation-policy.md).

## 7. Contribution

If you want to add papers, code, benchmarks, datasets, or tutorials:

- read [CONTRIBUTING.md](../CONTRIBUTING.md);
- follow [docs/curation-policy.md](../docs/curation-policy.md);
- keep [README.md](./README.md) and [README.zh-CN.md](./README.zh-CN.md) aligned when resource changes affect both languages.

## 8. Current Status

This repository has moved from pure structure-building into an initial curation stage.

The current version establishes a layered DLLM map with a first batch of papers, project pages, and official repositories. The next step is to expand coverage carefully while keeping section boundaries stable and descriptions objective.

For readers who want a more explicit technical overview, see [docs/mathematical-foundations.md](./docs/mathematical-foundations.md).
For a guided reading path across the `docs/` directory, see [docs/overview.md](./docs/overview.md).
For inference behavior and decoding tradeoffs, see [docs/sampling-and-decoding.md](./docs/sampling-and-decoding.md).
For how different DLLM families are actually trained, see [docs/training-objectives.md](./docs/training-objectives.md).
For a structured map of major method families, see [docs/method-family-map.md](./docs/method-family-map.md).
For how DLLMs should be compared and evaluated, see [docs/benchmark-and-evaluation.md](./docs/benchmark-and-evaluation.md).
For a chronological view of the field, see [docs/timeline.md](./docs/timeline.md).
For recurring terms, see [docs/glossary.md](./docs/glossary.md).
