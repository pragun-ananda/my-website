---
layout: post
title: TextGenShap - Shapley values for LLM Output
categories:
- Interpretability
- Research
---

This blog post describes my experience working as a research engineer on Google’s [Cloud AI research team](https://research.google/teams/cloud-ai/). The work was focused on TextGenShap ([Enounen et al., 2023](https://arxiv.org/abs/2312.01279)), a method to efficiently generate explainability scores for LLM generated output. The research shows that in the context of using LLMs for information retrieval (specifically for long-document question answering), we can get good results by ranking candidate answers by their explainability scores. My focus on this project was extending the algorithm's implementation to support recent models ([Gemma](https://ai.google.dev/gemma), [Llama 3](https://llama.meta.com/llama3/)) and arbitrary datasets. I also spent time refactoring the bare-bones implementation to a production quality version. Included in this blog post is an (1) overview of the research method, (2) a description of the work that I did on the project, and (3) my main takeaways. Feel free to skip to the sections that seem the most relevant to you.

# Research Overview

LLMs excel at massively multi-task learning (ex: summarization, retrieval, few-shot learning to name a few.). Oftentimes, their aptitude in different settings is difficult to explain. This commonly leads to LLMs being treated as a black-box. Interpretability is the branch of machine learning focused on understanding a machine learning model’s predictions. 

A traditional method of explaining feature attribution in machine learning is through Shapley values ([Lundberg and Lee et al., 2017](https://arxiv.org/abs/1705.07874)). Shapley values are well-suited for discriminative tasks (ex: classification or regression). They answer the question: “How much has each feature value contributed to the prediction compared to the average prediction?” (Source: [Interpretable ML book](https://christophm.github.io/interpretable-ml-book/shapley.html)). 

They are not, however, well-suited for generative tasks - not to mention that they are expensive to compute for LLMs, especially when context length is large. TextGenShap adapts the conventional Shapley value to one that works for generative LLM output. It also takes advantage of the hierarchical structure of long-form text to represent and efficiently compute explainability values at different granularity levels (ex: document-level, sentence-level, and word-level). It achieves speedups through several architectural optimizations to T5 ([Raffel et al., 2020](https://arxiv.org/abs/1910.10683)), the model used for inference. Implemented optimizations include FlashAttention ([Dao et al., 2022](https://arxiv.org/abs/2205.14135)), speculative decoding ([Leviathan et al., 2022](https://arxiv.org/abs/2211.17192), [Miao et al., 2023](https://arxiv.org/abs/2305.09781)), and in-place encoder resampling. 

The datasets used in experiments are (1) NaturalQuestions ([Kwiatkowski et al., 2019](https://aclanthology.org/Q19-1026/)) and (2) MIRACL ([Zhang et al., 2022](https://arxiv.org/abs/2210.09984)), Wikipedia datasets designed for open-domain or long-document question answering. 

# Personal Contributions

# Main Takeaways