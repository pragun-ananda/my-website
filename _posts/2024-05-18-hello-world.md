---
layout: post
title: TextGenShap - Shapley values for LLM Output
categories:
- Interpretability
- Research
---

This blog post describes my experience working as a research engineer on Google’s [Cloud AI research team](https://research.google/teams/cloud-ai/). The work was focused on TextGenShap ([Enounen et al., 2023](https://arxiv.org/abs/2312.01279)), a method to efficiently generate explainability scores for LLM generated output. The research shows that in the context of using LLMs for information retrieval (specifically for long-document question answering), we can get good results by ranking candidate answers by their explainability scores. My focus on this project was extending the algorithm's implementation to support recent models (like [Gemma](https://ai.google.dev/gemma) and [Llama 3](https://llama.meta.com/llama3/)) and arbitrary datasets. I also spent time refactoring the bare-bones implementation to a production quality version. Included in this blog post is an (1) overview of the research method, (2) a description of the work that I did on the project, and (3) my main takeaways. Feel free to skip to the sections that seem the most relevant to you. All of the code for the paper is open-sourced [here](https://github.com/google-research/google-research/tree/master/llm_longdoc_interpretability) on Github.

# Research Overview

LLMs excel at massively multi-task learning (ex: summarization, retrieval, few-shot learning to name a few.). Oftentimes, their aptitude in different settings is difficult to explain. This commonly leads to LLMs being treated as a black-box. Interpretability is the branch of machine learning focused on understanding a machine learning model’s predictions. 

A traditional method of explaining feature attribution in machine learning is through Shapley values ([Lundberg and Lee et al., 2017](https://arxiv.org/abs/1705.07874)). Shapley values are well-suited for discriminative tasks (ex: classification or regression). They answer the question: “How much has each feature value contributed to this prediction compared to the average prediction?” (Source: [Interpretable ML book](https://christophm.github.io/interpretable-ml-book/shapley.html)). 

They are not, however, well-suited for generative tasks - not to mention that they are expensive to compute for LLMs, especially when context length is large. TextGenShap adapts the conventional Shapley value to one that works for generative LLM output. It also takes advantage of the hierarchical structure of long-form text to represent and efficiently compute explainability values at different granularity levels (ex: document-level, sentence-level, and word-level). It achieves speedups through several architectural optimizations to T5 ([Raffel et al., 2020](https://arxiv.org/abs/1910.10683)), the model used for inference. Implemented optimizations include FlashAttention ([Dao et al., 2022](https://arxiv.org/abs/2205.14135)), speculative decoding ([Leviathan et al., 2022](https://arxiv.org/abs/2211.17192), [Miao et al., 2023](https://arxiv.org/abs/2305.09781)), and in-place encoder resampling. 

The datasets used in experiments are (1) NaturalQuestions ([Kwiatkowski et al., 2019](https://aclanthology.org/Q19-1026/)) and (2) MIRACL ([Zhang et al., 2022](https://arxiv.org/abs/2210.09984)), Wikipedia datasets designed for open-domain or long-document question answering. 

# Personal Contributions

My overall role on the project was to convert the existing research experiment code to a production-grade version running on Google Cloud AI infrastructure to support demos to several leadership teams in Google Cloud Research and Google Deepmind. This includes:

- Building an inference abstraction that supports newer models (Llama3, Gemma, Gemini) not just T5 (which was used in initial experiments).
- Adding the containerization, model deployment, and TextGenShap batch job infrastructure on Vertex. Later, building serving support for real-time execution was a separate requirement. 
- Tuning several algorithm hyperparameters (permutation sampling, granularity of document hierarchies) to achieve a reasonable tradeoff between accuracy and latency for batch execution vs. real-time execution. 
- Meeting with different stakeholders to understand use cases for Cloud AI customers and  running experiments with differently structured datasets to show TextGenShap’s use cases in long-document information retrieval.

# Main Takeaways

I really enjoyed working on this project for a few reasons:

1) I was exposed to and able to collaborate with several influential AI researchers, specifically in the field of interpretability/explainability. 

I'm so grateful that I was introduced the field of interpretability through this project. Over the past year, I've tried to build intuition as to what area of AI research I'd be interested in pursuing a PhD in. Personally, I'd love to work in an area that has social impact as well interesting engineering challenges. As I've learned more about interpretability, I think it might be a great fit.

Unfortunately, the field of interpretability does not get as much attention as it deserves considering the rapid progress in AI we see today has enormous implications on future society. If we don't develop a deeper understanding of AI systems, it's impossible to have control over how they're applied. I sadly don't see a large investment in this research area from any major AI lab aside from Anthropic. By far, the biggest investments I see are directed at answering the question: "How do we make these models bigger and better?. 

2) I've always enjoyed working on infrastructure problems and this project had some interesting opportunities to make engineering tradeoffs along the way.

The driving force behind me joining Google out of college was my interest in learning how the biggest software systems in the world are built. I now work on Google's core search infrastructure and it's fascinating to see the scale that the company operates at.  

3) I've always wanted to work in research. I love learning and I love working with people. Research has endless opportunities to learn and grow from others, as well as to teach others. 

My work on this project for Google’s Cloud AI Research team was actually entirely voluntary. Google supports employee side projects through what it calls “20% projects” meaning my involvement was a 20% time commitment I chose to make in addition to my full-time role. For me, working on this project was an obvious choice since I eventually want to work as an AI researcher full-time. Anyways, maybe describing my current full-time role can be a topic for another blog post :)
