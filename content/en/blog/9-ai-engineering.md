---
author: "Loko"
title: "AI Engineering"
date: 2026-07-16
lastmod: 2026-07-16
description: "A must-read for getting started with AI Engineering"
tags: ["ai"]
thumbnail: /thumbnail/ai-engineering.jpg
toc: true
---

## 1. Here to End the Debate

If there's one keyword that defines the current development landscape, it's undeniably **AI**. AI tools have become nearly indispensable for developers — something you simply can't separate yourself from. I've become so accustomed to using Claude that I now rely on AI for coding practice, brainstorming blog ideas, planning side projects, and countless other tasks.

Around that time, I happened to join a startup building an AI-powered chat service. Naturally, my curiosity expanded beyond backend engineering into a new territory: **AI Engineering**.

After enduring more than a year of setbacks while searching for a new job — combined with the anxiety that AI might replace backend developers within the next few years — I found myself genuinely motivated to study AI Engineering.

## 2. What This Book Covers

Everything I studied and organized while reading this book is already available in my [personal GitHub Repository](https://github.com/nmin11/TIL/tree/main/AI%20Engineering/ai-engineering). So in this post, I'll offer a brief, chapter-by-chapter overview of what you can learn.

### Chapter 1. Introduction to Building AI Applications

Before diving into AI Engineering, the book starts by tracing how language models have evolved.

Early language models worked by predicting which words would appear in a context based on a basic unit called a **token**. As **self-supervised learning** became possible, language models expanded into **LLMs (Large Language Models)**. Research then progressed toward models capable of processing not just language but also images, videos, and other diverse data — enabling them to perceive and interact with the world much like humans. This gave rise to **foundation models** that can handle multiple tasks alongside multimodal capabilities.

<img src="/blog/ai-3-layers.png" class="hover-zoom">

Working alongside AI has made building applications remarkably easier. But this also makes **competitive advantage** a more pressing question. Among technology, data, and distribution — where can we actually gain an edge? With foundation models as the base, most companies will share similar technical capabilities, and large enterprises will dominate distribution. For a startup, then, the most viable strategy is to center everything around **data**.

### Chapter 2. Understanding Foundation Models

This chapter focuses less on how to build foundation models from scratch and more on the foundational knowledge needed to use them effectively.

<img src="/blog/seq2seq-architecture.png" class="hover-zoom">

The [seq2seq (sequence to sequence)](https://arxiv.org/abs/1409.3215) architecture, published by Google in 2014, significantly improved machine translation. seq2seq used **RNNs (Recurrent Neural Networks)** for both encoder and decoder, designed to process tokens sequentially. However, its sequential nature led to quality degradation and latency issues.

<img src="/blog/attention-is-all-you-need.png" class="hover-zoom">

The **Transformer** architecture solved these problems. Through the **attention mechanism**, it assigns weights to the importance of input tokens and processes them in parallel, dramatically improving speed. Output generation, however, remains sequential.

When understanding models, the importance of **post-training** beyond pre-training should not be overlooked. Post-training filters out inappropriate responses and enables the model to generate outputs that feel more human-like. It generally involves two stages: **Supervised Fine-Tuning (SFT)** and **Preference Fine-Tuning**.

<img src="/blog/rlhf.jpeg" class="hover-zoom">

SFT fine-tunes the model with high-quality instruction data, while Preference Fine-Tuning teaches the model to align with human preferences — typically through reinforcement learning methods such as **RLHF (Reinforcement Learning from Human Feedback)**.

### Chapter 3. Evaluation Methodology

Because foundation models are open-ended, there are no clear ground-truth answers to evaluate against — you can only observe the outputs of a black box, making evaluation inherently difficult. Even so, we must not overlook the importance of evaluation if we want to identify system weaknesses and make the system more robust.

Since the foundation of foundation models is language modeling, understanding language modeling metrics is essential. **Entropy** measures how much information a token carries — higher entropy means more information, and lower entropy means the model can more easily predict what comes next. **Cross-entropy** indicates how hard a language model finds it to predict the content of a dataset. Both can be captured through metrics like **BPB (Bits Per Byte)**, which measures how well text can be compressed compared to the original. **Perplexity** is the exponential of entropy and cross-entropy, representing the uncertainty in predicting the next token — higher uncertainty means more possible candidates for the next token.

More recently, **AI evaluators** — using AI to evaluate AI — are seeing growing adoption. While not as accurate as human evaluation, they can be helpful enough to guide development direction. However, since AI evaluation standards are not yet standardized, care must be taken around bias and ambiguity in evaluation criteria.

### Chapter 4. Evaluate AI Systems

Criteria for evaluating an application's model include **domain-specific capabilities**, **generative ability**, **instruction-following ability**, **cost**, and **latency**. Before development, we must clearly define evaluation criteria, and these should reflect business priorities.

The process of selecting a model is equally important. Decisions must be made around cost-performance trade-offs, and whether to develop a model in-house or use a commercial model's API.

Rather than settling for one-off evaluations, you should design an **evaluation pipeline** that independently assesses each component of the system. Connect evaluation metrics to business metrics to decide which indicators to prioritize.

### Chapter 5. Prompt Engineering

Prompt engineering is often mistakenly seen as something simple that even beginners can do — especially compared to fine-tuning. But prompt engineering deserves the same rigor as other ML techniques, because it enables efficient model adjustment with far fewer resources than fine-tuning.

<img src="/blog/zero-shot-one-shot-few-shot.png" class="hover-zoom">

One of the first things to consider in prompt engineering is adjusting the number of **shots**. More shots provide richer examples and generally improve reasoning quality — but they also make prompts longer, increasing inference costs. Experimentation is needed to find the right balance.

Best practices in prompt engineering include **providing sufficient context**, **breaking tasks into units**, **Chain of Thought (CoT)**, **evaluating prompt engineering tools**, and **prompt version control**.

You should also prepare for **prompt attacks** such as prompt extraction, jailbreaking, and prompt injection. Benchmarks and automated security testing tools can be valuable for this.

### Chapter 6. RAG and Agents

There are two primary mechanisms models use to construct context: **RAG (Retrieval-Augmented Generation)** and **agents**.

<img src="/blog/rag.jpg" class="hover-zoom">

RAG allows a model to retrieve relevant information from external sources to build context. Both keyword-based retrieval (TF-IDF, Elasticsearch, BM25) and embedding-based retrieval (e.g., Vector DBs) can be used — and in practice, the two are often combined into a hybrid approach.

Agents can perceive their environment and make changes to it through tools. Various agent patterns exist for this purpose, and the **ReAct** pattern — which alternates between reasoning and action, continuously evaluating and improving its own process — is one of the most widely used.

### Chapter 7. Finetuning

Fine-tuning is a form of **Transfer Learning (TL)**, applying lessons learned from one task to another. Ideally, fine-tuning is less about learning something new and more about refining the model's existing behavior.

Before jumping into fine-tuning, though, it's worth carefully examining whether it's truly necessary. Fine-tuning requires substantial computing resources as well as specialized ML expertise. If the issue is a lack of facts or information, RAG may be more efficient. And even for performance improvements, prompt engineering alone is often sufficient. The recommended approach is to try prompts first, then consider fine-tuning when you need to enforce specific formatting or improve robustness.

Fine-tuning is memory-intensive. This is why techniques like **Backpropagation** and **quantization** are used to reduce memory consumption. A prominent fine-tuning technique is **PEFT (Parameter-Efficient Fine-Tuning)**, which performs partial fine-tuning with fewer parameters while achieving performance close to full fine-tuning. Within PEFT, various methods exist — and **LoRA (Low-Rank Adaptation)** is among the most well-known.

### Chapter 8. Dataset Engineering

Training a model requires data. Building a high-quality dataset is critical to preventing bias and hallucination.

Data can also be generated through augmentation and synthesis — and recently, AI-generated data has become increasingly common. However, when using AI to generate data, special attention must be paid to data quality.

Another approach is **model distillation**, where a smaller model is trained to mimic a larger one.

### Chapter 9. Inference Optimization

If training is the process of creating a model, inference is the process of feeding it inputs and obtaining outputs.

Inference optimization is about identifying and resolving computational bottlenecks. The main bottlenecks are **compute-bound** and **memory bandwidth-bound** constraints. **Prefill** — where the model processes input tokens in parallel — is compute-bound, while **decode** — where the model generates output tokens one at a time — is memory bandwidth-bound.

Inference optimization techniques include **model size compression**, **resolving autoregressive decoding bottlenecks**, and **attention mechanism optimization**.

### Chapter 10. AI Engineering Architecture and User Feedback

When building AI applications, it's most efficient to start with the simplest architecture and progressively add more components. Begin with the most basic structure — where the user communicates directly with the model API — and expand step by step:

- Step 1: Context enrichment
- Step 2: Guardrail introduction
- Step 3: Model router and gateway
- Step 4: Caching
- Step 5: Agent patterns
- Step 6: Monitoring setup
- Step 7: AI pipeline orchestration

The resulting architecture, when fully assembled, looks roughly like this:

<img src="/blog/ai-architecture.png" class="hover-zoom">

In AI applications, **user feedback** becomes far more important than in typical applications. This means you need to thoughtfully design when and how to collect feedback — while also being clear-eyed about the inherent limitations and potential biases that come with it.

## 3. Into Practice, and What's Next

I've tried to summarize the book as concisely as possible above. But in reality, this is just a list of core concepts. Digging deeper into each concept while reading revealed many parts that were difficult to fully grasp — which made me realize I need to build something to close that knowledge gap. I want to solidify this reading experience by creating a side project.

Lately, I've been thinking more about how to continue growing my career. As I've come to rely on AI more, I feel less like I'm personally contributing to product quality improvements — and it's become harder to know what to study to become a truly capable developer. I even find myself hesitant to write my own blog posts out of fear that people might assume AI wrote them. So going forward, I think focusing on **side projects** might be the answer. Since AI has advanced this far, I want to be the kind of developer who maximizes the power of AI to produce meaningful, high-impact output.
