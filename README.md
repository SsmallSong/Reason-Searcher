
<h1 align="center"> R1-searcher:  Incentivizing the Search Capability in LLMs via Reinforcement Learning</a></h1>


<div align="center"> 
<a href="https://github.com/SsmallSong/RLRAG/edit/main//LICENSE"><img src="https://img.shields.io/badge/Code_License-MIT-blue" alt="license"></a>
<a href="https://github.com/SsmallSong/RLRAG/edit/main//LICENSE"><img src="https://img.shields.io/badge/Model_License-MIT-blue" alt="license"></a>
<a href="[https://huggingface.co/collections/yulan-team/yulan-mini-676d214b24376739b00d95f3](https://github.com/SsmallSong/RLRAG)"><img alt="Hugging Face" src="https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-blue?color=8A2BE2"></a>
 
</div>


<!-- <div align="center">
    <span style="display:inline-block; margin-right: 10px;">
        <a href="https://paperswithcode.com/sota/mathematical-reasoning-on-aime24?p=search-o1-agentic-search-enhanced-large">
            <img src="https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/search-o1-agentic-search-enhanced-large/mathematical-reasoning-on-aime24" alt="AIME24 Badge">
        </a>
    </span>
    <span style="display:inline-block; margin-right: 10px;">
        <a href="https://paperswithcode.com/sota/mathematical-reasoning-on-amc23?p=search-o1-agentic-search-enhanced-large">
            <img src="https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/search-o1-agentic-search-enhanced-large/mathematical-reasoning-on-amc23" alt="AMC23 Badge">
        </a>
    </span>
  <span style="display:inline-block; margin-right: 10px;">
        <a href="https://paperswithcode.com/sota/on-gpqa?p=search-o1-agentic-search-enhanced-large">
            <img src="https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/search-o1-agentic-search-enhanced-large/on-gpqa" alt="GPQA Badge">
        </a>
    </span>
</div> -->



<h5 align="center"> If you like our project, please give us a star ⭐ on GitHub for the latest update.</h5>


# 💡 Overview

Large reasoning models (LRMs), such as OpnAI-o1 and Deepseek-R1, have demonstrated the significant impact of reinforcement learning in enhancing the long-step reasoning capabilities of models, thereby greatly improving their reasoning performance. Despite these advantages, when faced with knowledge-intensive problems, especially multi-hop questions and time-sensitive issues, these models may lack the necessary knowledge. Therefore, it is great important to enable LLMs to invoke web search and obtain external information during the reasoning process. 

We propose **R1-searcher**, utilizing a *two-stage outcome-supervision reinforcement learning* approach to enable the model to learn to invoke web search during the reasoning process: first allowing the model to learn how to invoke web search, and then teaching it how to effectively use that search engine. This method does not require any instruction fine-tuning for cold start, and at the same time, it is compatible with existing Base LLMs or Chat LLMs. We will open-source the training code, inference code, model checkpoints, and the detailed technical report.

- Arxiv: https://arxiv.org/pdf/2503.05592
- Model:
    - Qwen-2.5-7B-Base-RAG-RL: https://huggingface.co/XXsongLALA/Qwen-2.5-7B-base-RAG-RL
    - Llama-3.1-8B-Instruct-RAG-RL: https://huggingface.co/XXsongLALA/Llama-3.1-8B-instruct-RAG-RL
- Train-data:  https://huggingface.co/datasets/XXsongLALA/RAG-RL-Hotpotqa-with-2wiki

![benchmark_picture](https://github.com/SsmallSong/R1-Searcher/blob/main/assets/benchmarks_visible.jpg)

# ✨ Key Insights
- By relying solely on outcome-supervised reinforcement learning, we can activate the model's intrinsic search capabilities using only the query-answer pair, regardless of whether we are dealing with Base LLMs or Chat LLMs.
- Recent reinforcement learning algorithms, such as GRPO, PPO, and Reinforce++ both can effectively activate the internal search capabilities of the LLMs.
- There is no requirement for complex prompt engineering or process supervision during training.
- The capability of the Base LLMs largely influences whether the model can directly start training from zero.
- LongCoT reasoning after RL is an more effectively and efficient test time scaling method than existing tree-search based methods, e.g., Monte Carlo Tree Search.
- By using a local retrieval for RL training, the model can generalize well to other datasets and online searches scenarios.
- The final 7B parameters LLMs achieve the significant performance improvements compared to existing complex method or even close-sourced LLMs (e.g., GPT-4o-mini).

# ✨ Method
## Overall

We employ a Two-Stage Reward Guided RL Training approach:

Stage 1: Learn to invoke search with only format-reward.

Stage 2: Learn to solve questions with invoking search with format-reward and answer-reward.


## Algorithm
We use only outcome-supervised reinforcement learning for training, so we need to consider two main aspects: (1) the reinforcement learning algorithm, and (2) the design of the reward.

- RL Algorithm: We use an improved version of reinforce++, specifically by mimicking the GRPO algorithm. For each questions, we average the rewards of *n* samples, which stabilizes the training process. For the solution format, we utilize `<think>...</think>` tag for thinking, xxx for searching, and `<answer>...</answer>` for answering, `<begin_of_search>...<end_of_search>` for invoking search tool and `<begin_of_documents>...<end_of_documents>` for  returned retrieval documents.
- Reward Design：We utilize the F1-based rule-reward. We also experimented with Exact Match (EM) and Cover Exact Match (CEM) as rewards; however, these metrics do not serve as good benchmarks for these tasks, and a more detailed analysis will be released soon.  For the answer reward, we use the F-score between the golden answer and the predicted answer .For the format reward, in Stage 1, if the model performs retrieval and the solution meets the format requirements, 0.5 points are added to the answer reward. In Stage 2, the retrieval requirement is removed, and a penalty of 2 points is subtracted from the answer reward if the solution does not meet the format requirements. Detailed implementation, including hyperparameters can be found in our code.

## Data

We choose a portion of the training sets from HotpotQA and 2WikiMultiHopQA as our training data. We use Qwen-2.5-7B-Instruct to perform rollouts on the training dataset. 

Based on the number of rollouts required to answer a question correctly, we classify the data into three categories: easy (<10 rollouts), medium (10 < and < 20 rollouts), and difficult (>20 rollouts). These categories are then mixed in a specific ratio to form our training data. All of our training data can be found here:  https://huggingface.co/datasets/XXsongLALA/RAG-RL-Hotpotqa-with-2wiki.




# 📄 Evaluation
Following ReARTeR(https://arxiv.org/pdf/2501.07861), we select four representative benchmarks: HotpotQA, 2WikiMultiHopQA, Musique, and Bamboogle.

HotpotQA and 2WikiMultiHopQA are considered in-domain as we use their training-set, while Musique and Bamboogle are classified as out-of-domain, allowing us to assess the generalization capabilities of our model. We randomly sample 500 examples from the development sets of HotpotQA, 2WikiMultiHopQA,  and Musique to serve as our test sets. For Bamboogle, we use all of the test set (125 samples) as our test set.. 

Wikipedia passages serve as the retrieval corpus for all datasets, specifically employing the [Wikipedia corpus released by KILT](https://github.com/facebookresearch/KILT) in August 2019. Additionally, due to the recency of the knowledge contained in Bamboogle, we incorporate online web search testing to conduct further evaluations, thereby examining the alignment of our model with online search capabilities. 

For the evaluation metrics, we use the ACC_R (Cover-Exect-Match) and ACC_L (LLM-as-Judge).
![benchmark](https://github.com/SsmallSong/R1-Searcher/blob/main/assets/benchmarks.jpg)
As we can see, when using the same LLaMA-3.1-8B-Instruct base model, our method has achieved significant improvements compared to existing methods, even surpassing closed-source models such as GPT-4o-mini. Furthermore, when switching to the more powerful base model, Qwen-2.5-7B-Base, we directly conduct reinforcement learning from scratch. Eventually, we can achieve better results and attain the best performance on all in-domain and out-of-domain datasets, demonstrating the exceptional generalization capabilities of our model.

For Bamboogle, we additionally utilize Google for online searches. As we can see, compared to relying solely on a local knowledge base, the incorporation of online search yields superior results, indicating that it is feasible to seamlessly integrate online search capabilities into our model.
![bamboogle](https://github.com/SsmallSong/R1-Searcher/blob/main/assets/bamboogle_online.jpg)



# 🏃 Quick Start



# 📄 Citation
Please kindly cite our report if they are helpful for your research.

```
@article{R1-searcher,
  title={R1-searcher:  Stimulating the Search Capability of LLM from Zero via Reinforcement Learning},
  author={Huatong Song, Jinhao Jiang, Yingqian Min, Jie Chen, Zhipeng Chen, Wayne Xin Zhao, Ji-Rong Wen, Yang Lu, Xu Miu},
  url={https://github.com/SsmallSong/R1-searcher},
  year={2025}
}
```

# 📄 License

This project is released under the [MIT License](LICENSE).

# 📞 Contact

For any questions or feedback, please reach out to us at [songhuatong123@ruc.edu.cn](songhuatong123@ruc.edu.cn).
