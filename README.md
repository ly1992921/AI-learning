# AI-learning 🤖

> **从产品经理视角系统学习人工智能技术**
>
> 本知识库面向希望深入理解AI底层原理的产品经理、技术管理者和行业研究者。
> 聚焦技术知识、学术知识学习与补短板，同时覆盖商业化、产品策略、市场分析等维度。

---

## 📋 目录

- [学习路径总览](#学习路径总览)
- [知识库结构](#知识库结构)
- [100篇必读论文清单](#100篇必读论文清单)
- [与具身智能的衔接](#与具身智能的衔接)
- [学习进度追踪](#学习进度追踪)
- [资源索引](#资源索引)

---

## 🗺️ 学习路径总览

```
Phase 1 (0-2月): AI基础与数学补完
Phase 2 (2-4月): 机器学习与深度学习核心
Phase 3 (4-6月): 计算机视觉与大模型
Phase 4 (6-8月): AI系统、芯片与具身智能
Phase 5 (8-10月): 商业化、产品策略与市场分析
Phase 6 (持续): 前沿论文精读与行业追踪
```

### 阶段详细规划

#### Phase 1: AI基础与数学补完 (Week 1-8)

| 周次 | 主题 | 核心内容 | 目标产出 |
|------|------|----------|----------|
| W1-2 | 线性代数与矩阵计算 | 向量空间、特征分解、SVD、PCA | 理解AI中的数学表达 |
| W3-4 | 概率论与统计 | 贝叶斯推断、信息论、最大似然 | 理解不确定性建模 |
| W5-6 | 微积分与优化 | 梯度下降、凸优化、拉格朗日乘子 | 理解模型训练原理 |
| W7-8 | Python编程与数据基础 | NumPy、Pandas、数据处理流程 | 能独立跑通简单模型 |

#### Phase 2: 机器学习与深度学习核心 (Week 9-16)

| 周次 | 主题 | 核心内容 | 目标产出 |
|------|------|----------|----------|
| W9-10 | 经典机器学习 | 回归、决策树、SVM、集成学习 | 理解传统ML方法边界 |
| W11-12 | 神经网络基础 | MLP、反向传播、激活函数 | 理解前馈网络原理 |
| W13-14 | CNN架构演进 | LeNet→ResNet→EfficientNet | 理解视觉模型演进 |
| W15-16 | RNN与序列建模 | LSTM、GRU、注意力机制 | 理解序列数据建模 |

#### Phase 3: 计算机视觉与大模型 (Week 17-24)

| 周次 | 主题 | 核心内容 | 目标产出 |
|------|------|----------|----------|
| W17-18 | Transformer架构 | Self-Attention、多头注意力、位置编码 | 深入理解Transformer |
| W19-20 | 大语言模型 | GPT系列、BERT、T5 | 理解LLM训练与推理 |
| W21-22 | 多模态AI | CLIP、DALL-E、多模态融合 | 理解跨模态学习 |
| W23-24 | Vision Transformer | ViT、Swin Transformer、MAE | 理解视觉Transformer |

#### Phase 4: AI系统、芯片与具身智能 (Week 25-32)

| 周次 | 主题 | 核心内容 | 目标产出 |
|------|------|----------|----------|
| W25-26 | AI推理系统 | 模型部署、量化、剪枝、蒸馏 | 理解推理优化策略 |
| W27-28 | AI芯片架构 | GPU/TPU/NPU架构、存算一体 | 理解硬件约束与设计 |
| W29-30 | 强化学习基础 | MDP、Q-Learning、Policy Gradient | 理解RL决策框架 |
| W31-32 | 具身智能进阶 | VLA、RT系列、Sim-to-Real | 衔接LinuxHermes知识 |

#### Phase 5: 商业化、产品策略与市场 (Week 33-40)

| 周次 | 主题 | 核心内容 | 目标产出 |
|------|------|----------|----------|
| W33-34 | AI产品方法论 | AI PM能力模型、产品设计原则 | 形成AI产品思维 |
| W35-36 | AI商业模式 | MaaS、API经济、开源商业化 | 理解AI商业化路径 |
| W37-38 | 市场竞争分析 | 中美AI格局、芯片竞争、生态 | 形成行业洞察 |
| W39-40 | 合规与伦理 | AI治理、数据隐私、安全对齐 | 理解产品合规边界 |

---

## 📁 知识库结构

```
AI-learning/
├── 01-AI基础与数学/          # 线性代数、概率统计、微积分、Python
├── 02-机器学习基础/           # 经典ML算法、特征工程、模型评估
├── 03-深度学习框架/           # PyTorch/TensorFlow、神经网络基础
├── 04-计算机视觉/             # CNN、目标检测、图像生成、ViT
├── 05-自然语言处理与大模型/    # Transformer、LLM、Prompt工程、RAG
├── 06-强化学习与具身智能/      # RL基础、VLA、机器人学习、Sim-to-Real
├── 07-AI系统与工程/           # 训练基础设施、推理优化、MLOps
├── 08-AI芯片与硬件/           # GPU/TPU/NPU、存算一体、边缘AI
├── 09-商业化与产品策略/        # AI PM、商业模式、竞品分析、市场
├── 10-前沿论文精读/           # 按主题组织的精读笔记
├── papers/                   # 论文清单与索引
│   ├── A级必读 (10篇)        # 奠基性论文，必须精读
│   ├── B级推荐 (20篇)        # 重要论文，建议精读
│   └── C级参考 (70篇)        # 延伸阅读，按需查阅
├── roadmap/                  # 学习路径规划
├── resources/                # 外部资源链接
└── notes/                    # 个人学习笔记
```

---

## 📚 100篇必读论文清单

### 🏆 A级必读 (10篇) — 奠基性论文，必须精读

> A级论文是AI领域的"原典"，构成了整个领域的理论基础。每篇都需要反复阅读、做笔记、理解其数学推导和实验设计。

| 编号 | 论文 | 作者 | 年份 | 会议/期刊 | 核心贡献 | 为什么必读 |
|------|------|------|------|-----------|----------|------------|
| A-01 | [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | Vaswani et al. | 2017 | NeurIPS | 提出Transformer架构，Self-Attention机制 | 现代AI的基石，大模型的起点 |
| A-02 | [ImageNet Classification with Deep CNNs (AlexNet)](https://papers.nips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html) | Krizhevsky et al. | 2012 | NeurIPS | 深度CNN在ImageNet上突破性表现 | 深度学习复兴的标志 |
| A-03 | [Deep Residual Learning for Image Recognition (ResNet)](https://arxiv.org/abs/1512.03385) | He et al. | 2016 | CVPR | 残差连接解决深层网络退化问题 | 现代CNN架构的基础 |
| A-04 | [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) | Devlin et al. | 2019 | NAACL | 双向Transformer预训练，NLP范式转变 | NLP预训练时代的里程碑 |
| A-05 | [Language Models are Few-Shot Learners (GPT-3)](https://arxiv.org/abs/2005.14165) | Brown et al. | 2020 | NeurIPS | 175B参数模型展现涌现能力 | 大模型时代的开端 |
| A-06 | [Deep Learning](https://www.nature.com/articles/nature14539) | LeCun, Bengio, Hinton | 2015 | Nature | 深度学习三大巨头的综述 | 理解DL历史与核心思想 |
| A-07 | [Mastering the Game of Go with Deep Neural Networks and Tree Search (AlphaGo)](https://www.nature.com/articles/nature16961) | Silver et al. | 2016 | Nature | DNN + Monte Carlo Tree Search击败人类 | RL+DNN结合的经典案例 |
| A-08 | [Generative Adversarial Networks](https://arxiv.org/abs/1406.2661) | Goodfellow et al. | 2014 | NeurIPS | 对抗训练框架 | 生成式AI的理论基础 |
| A-09 | [A Survey of Large Language Models](https://arxiv.org/abs/2303.18223) | Zhao et al. | 2023 | arXiv | 大模型全面综述 | 快速建立LLM全局视野 |
| A-10 | [Learning Transferable Visual Models From Natural Language Supervision (CLIP)](https://arxiv.org/abs/2103.00020) | Radford et al. | 2021 | ICML | 跨模态对比学习 | 多模态AI的重要突破 |

---

### ⭐ B级推荐 (20篇) — 重要论文，建议精读

> B级论文代表各子领域的重要进展，帮助建立细分领域的深度认知。

#### B1-B5: 计算机视觉

| 编号 | 论文 | 作者 | 年份 | 会议 | 核心贡献 |
|------|------|------|------|------|----------|
| B-01 | [Faster R-CNN: Towards Real-Time Object Detection](https://arxiv.org/abs/1506.01497) | Ren et al. | 2015 | NeurIPS | 两阶段目标检测里程碑 |
| B-02 | [You Only Look Once: Unified, Real-Time Object Detection (YOLO)](https://arxiv.org/abs/1506.02640) | Redmon et al. | 2016 | CVPR | 单阶段检测开创性工作 |
| B-03 | [An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale (ViT)](https://arxiv.org/abs/2010.11929) | Dosovitskiy et al. | 2021 | ICLR | Transformer用于图像分类 |
| B-04 | [High-Resolution Image Synthesis with Latent Diffusion Models (Stable Diffusion)](https://arxiv.org/abs/2112.10752) | Rombach et al. | 2022 | CVPR | 潜在扩散模型 |
| B-05 | [Segment Anything](https://arxiv.org/abs/2304.02643) | Kirillov et al. | 2023 | ICCV | 零样本图像分割 |

#### B6-B10: 自然语言处理与大模型

| 编号 | 论文 | 作者 | 年份 | 会议 | 核心贡献 |
|------|------|------|------|------|----------|
| B-06 | [Chain-of-Thought Prompting Elicits Reasoning in LLMs](https://arxiv.org/abs/2201.11903) | Wei et al. | 2022 | NeurIPS | 思维链推理 |
| B-07 | [Llama 2: Open Foundation and Fine-Tuned Chat Models](https://arxiv.org/abs/2307.09288) | Touvron et al. | 2023 | arXiv | 开源大模型标杆 |
| B-08 | [InstructGPT: Training Language Models to Follow Instructions](https://arxiv.org/abs/2203.02155) | Ouyang et al. | 2022 | NeurIPS | RLHF对齐训练 |
| B-09 | [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) | Lewis et al. | 2020 | NeurIPS | RAG框架 |
| B-10 | [Sparks of Artificial General Intelligence: Early Experiments with GPT-4](https://arxiv.org/abs/2303.12712) | Bubeck et al. | 2023 | arXiv | GPT-4能力评估 |

#### B11-B15: 强化学习与具身智能

| 编号 | 论文 | 作者 | 年份 | 会议 | 核心贡献 |
|------|------|------|------|------|----------|
| B-11 | [Playing Atari with Deep Reinforcement Learning (DQN)](https://arxiv.org/abs/1312.5602) | Mnih et al. | 2013 | NeurIPS | 深度Q网络 |
| B-12 | [Proximal Policy Optimization Algorithms](https://arxiv.org/abs/1707.06347) | Schulman et al. | 2017 | arXiv | PPO算法 |
| B-13 | [RT-1: Robotics Transformer for Real-World Control at Scale](https://arxiv.org/abs/2212.06817) | Brohan et al. | 2023 | RSS | 大规模机器人Transformer |
| B-14 | [RT-2: Vision-Language-Action Models](https://arxiv.org/abs/2307.15818) | Brohan et al. | 2023 | arXiv | VLA模型 |
| B-15 | [Diffusion Policy: Visuomotor Policy Learning via Action Diffusion](https://arxiv.org/abs/2303.04137) | Chi et al. | 2023 | RSS | 扩散策略用于机器人控制 |

#### B16-B20: AI系统与芯片

| 编号 | 论文 | 作者 | 年份 | 会议 | 核心贡献 |
|------|------|------|------|------|----------|
| B-16 | [Efficient Large-Scale Language Model Training on GPU Clusters Using Megatron-LM](https://arxiv.org/abs/2104.04473) | Narayanan et al. | 2021 | SC | 大规模分布式训练 |
| B-17 | [FlashAttention: Fast and Memory-Efficient Exact Attention](https://arxiv.org/abs/2205.14135) | Dao et al. | 2022 | NeurIPS | 注意力计算优化 |
| B-18 | [In-Datacenter Performance Analysis of a Tensor Processing Unit](https://arxiv.org/abs/1704.04760) | Jouppi et al. | 2017 | ISCA | TPU架构详解 |
| B-19 | [MLPerf Training Benchmark](https://arxiv.org/abs/1910.01500) | Mattson et al. | 2020 | arXiv | AI训练基准测试 |
| B-20 | [A Survey on Efficient Inference for Large Language Models](https://arxiv.org/abs/2404.14294) | Miao et al. | 2024 | arXiv | LLM推理优化全面综述 |

---

### 📖 C级参考 (70篇) — 延伸阅读，按需查阅

> C级论文覆盖更广泛的主题和最新进展，根据兴趣方向选择性阅读。

#### C1-C10: 深度学习基础与优化

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-01 | [Adam: A Method for Stochastic Optimization](https://arxiv.org/abs/1412.6980) | Kingma & Ba | 2015 | 优化器 |
| C-02 | [Batch Normalization: Accelerating Deep Network Training](https://arxiv.org/abs/1502.03167) | Ioffe & Szegedy | 2015 | 批归一化 |
| C-03 | [Dropout: A Simple Way to Prevent Neural Networks from Overfitting](https://www.jmlr.org/papers/v15/srivastava14a.html) | Srivastava et al. | 2014 | 正则化 |
| C-04 | [Stochastic Gradient Descon](https://arxiv.org/abs/1405.3186) | - | - | SGD理论 |
| C-05 | [On the Difficulty of Training Recurrent Neural Networks](https://arxiv.org/abs/1211.5063) | Pascanu et al. | 2013 | RNN梯度问题 |
| C-06 | [Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification](https://arxiv.org/abs/1502.01852) | He et al. | 2015 | ReLU与初始化 |
| C-07 | [Xavier/Glorot Initialization](http://proceedings.mlr.press/v9/glorot10a.html) | Glorot & Bengio | 2010 | 权重初始化 |
| C-08 | [SGDR: Stochastic Gradient Descent with Warm Restarts](https://arxiv.org/abs/1608.03983) | Loshchilov & Hutter | 2017 | 学习率调度 |
| C-09 | [Sharpness-Aware Minimization for Efficiently Improving Generalization](https://arxiv.org/abs/2010.01412) | Foret et al. | 2021 | SAM优化 |
| C-10 | [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) | Hu et al. | 2021 | 参数高效微调 |

#### C11-C20: 计算机视觉进阶

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-11 | [Fully Convolutional Networks for Semantic Segmentation](https://arxiv.org/abs/1411.4038) | Long et al. | 2015 | 语义分割 |
| C-12 | [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/abs/1505.04597) | Ronneberger et al. | 2015 | U-Net架构 |
| C-13 | [Feature Pyramid Networks for Object Detection](https://arxiv.org/abs/1612.03144) | Lin et al. | 2017 | FPN |
| C-14 | [Deformable Convolutional Networks](https://arxiv.org/abs/1703.06211) | Dai et al. | 2017 | 可变形卷积 |
| C-15 | [Non-local Neural Networks](https://arxiv.org/abs/1711.07971) | Wang et al. | 2018 | 非局部注意力 |
| C-16 | [Swin Transformer: Hierarchical Vision Transformer](https://arxiv.org/abs/2103.14030) | Liu et al. | 2021 | Swin Transformer |
| C-17 | [Masked Autoencoders Are Scalable Vision Learners (MAE)](https://arxiv.org/abs/2111.06377) | He et al. | 2021 | 视觉自监督 |
| C-18 | [DINO: Emerging Properties in Self-Supervised Vision Transformers](https://arxiv.org/abs/2104.14294) | Caron et al. | 2021 | 自监督视觉 |
| C-19 | [GLIP: Grounded Language-Image Pre-training](https://arxiv.org/abs/2112.03857) | Li et al. | 2022 | 视觉-语言 grounding |
| C-20 | [SAM 2: Segment Anything in Images and Videos](https://arxiv.org/abs/2408.00714) | Ravi et al. | 2024 | 视频分割 |

#### C21-C30: 自然语言处理与大模型

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-21 | [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473) | Bahdanau et al. | 2015 | Attention机制起源 |
| C-22 | [The Annotated Transformer](https://nlp.seas.harvard.edu/annotated-transformer/) | Rush | 2018 | Transformer教程 |
| C-23 | [RoBERTa: A Robustly Optimized BERT Pretraining Approach](https://arxiv.org/abs/1907.11692) | Liu et al. | 2019 | BERT优化 |
| C-24 | [Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer (T5)](https://arxiv.org/abs/1910.10683) | Raffel et al. | 2020 | T5架构 |
| C-25 | [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361) | Kaplan et al. | 2020 | 缩放定律 |
| C-26 | [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) | Bai et al. | 2022 | AI反馈对齐 |
| C-27 | [Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761) | Schick et al. | 2023 | 工具使用 |
| C-28 | [Mixture of Experts (MoE) Explained](https://arxiv.org/abs/2203.01104) | Fedus et al. | 2022 | MoE架构 |
| C-29 | [The Llama 3 Herd of Models](https://ai.meta.com/research/publications/the-llama-3-herd-of-models/) | Meta AI | 2024 | Llama 3技术报告 |
| C-30 | [Gemini: A Family of Highly Capable Multimodal Models](https://arxiv.org/abs/2312.11805) | Team et al. | 2024 | Gemini模型 |

#### C31-C40: 多模态与生成式AI

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-31 | [DALL-E: Creating Images from Text](https://arxiv.org/abs/2102.12092) | Ramesh et al. | 2021 | 文本到图像 |
| C-32 | [Imagen: Photorealistic Text-to-Image Diffusion Models](https://arxiv.org/abs/2205.11487) | Saharia et al. | 2022 | 扩散图像生成 |
| C-33 | [Denoising Diffusion Probabilistic Models](https://arxiv.org/abs/2006.11239) | Ho et al. | 2020 | DDPM |
| C-34 | [Flow-based Deep Generative Models](https://arxiv.org/abs/1505.05770) | Dinh et al. | 2015 | 标准化流 |
| C-35 | [NeRF: Representing Scenes as Neural Radiance Fields](https://arxiv.org/abs/2003.08934) | Mildenhall et al. | 2020 | 神经辐射场 |
| C-36 | [GPT-4V(ision) System Card](https://openai.com/research/gpt-4v-system-card) | OpenAI | 2023 | 多模态LLM |
| C-37 | [Qwen-VL: A Versatile Large Vision-Language Model](https://arxiv.org/abs/2308.12966) | Bai et al. | 2023 | 中文多模态 |
| C-38 | [Sora: A Review on Background, Technology, Limitations, and Opportunities of Large Vision Models](https://arxiv.org/abs/2402.17177) | Liu et al. | 2024 | 视频生成 |
| C-39 | [VALL-E: Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers](https://arxiv.org/abs/2301.02111) | Wang et al. | 2023 | 语音合成 |
| C-40 | [Mamba: Linear-Time Sequence Modeling with Selective State Spaces](https://arxiv.org/abs/2312.00752) | Gu & Dao | 2024 | 线性注意力替代 |

#### C41-C50: 强化学习与具身智能

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-41 | [Human-level Control Through Deep Reinforcement Learning](https://www.nature.com/articles/nature14236) | Mnih et al. | 2015 | DQN Nature版本 |
| C-42 | [A3C: Asynchronous Methods for Deep Reinforcement Learning](https://arxiv.org/abs/1602.01783) | Mnih et al. | 2016 | 异步RL |
| C-43 | [Soft Actor-Critic: Off-Policy Maximum Entropy Deep RL](https://arxiv.org/abs/1801.01290) | Haarnoja et al. | 2018 | SAC算法 |
| C-44 | [World Models](https://arxiv.org/abs/1803.10122) | Ha & Schmidhuber | 2018 | 世界模型 |
| C-45 | [Dreamer: Deep Reinforcement Learning for World Models](https://arxiv.org/abs/1912.01603) | Hafner et al. | 2020 | Dreamer系列 |
| C-46 | [SURREAL: Open-Source Reinforcement Learning Framework](https://arxiv.org/abs/1811.02759) | Fan et al. | 2019 | RL系统 |
| C-47 | [Learning to Walk in Minutes Using Massively Parallel Deep RL](https://arxiv.org/abs/2109.11978) | Rudin et al. | 2022 | 并行RL训练 |
| C-48 | [Robotic Skill Acquisition via Instruction Augmentation](https://arxiv.org/abs/2211.11711) | - | 2022 | 指令增强 |
| C-49 | [Open X-Embodiment: Robotic Learning Datasets and RT-X Model](https://arxiv.org/abs/2310.08864) | OXE Team | 2023 | 跨机器人数据集 |
| C-50 | [Equivariant Diffusion for Manifold Learning](https://arxiv.org/abs/2307.05976) | - | 2023 | 等变扩散 |

#### C51-C60: AI系统与推理优化

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-51 | [ZeRO: Memory Optimizations Toward Training Trillion Parameter Models](https://arxiv.org/abs/1910.02054) | Rajbhandari et al. | 2020 | 显存优化 |
| C-52 | [GPipe: Easy Scaling with Micro-Batch Pipeline Parallelism](https://arxiv.org/abs/1811.06965) | Huang et al. | 2019 | 流水线并行 |
| C-53 | [GShard: Scaling Giant Models with Conditional Computation](https://arxiv.org/abs/2006.16668) | Lepikhin et al. | 2021 | 条件计算 |
| C-54 | [vLLM: Easy, Fast, and Cheap LLM Serving with PagedAttention](https://arxiv.org/abs/2309.06180) | Kwon et al. | 2023 | LLM推理服务 |
| C-55 | [TensorRT-LLM: A Library for Optimized LLM Inference](https://github.com/NVIDIA/TensorRT-LLM) | NVIDIA | 2023 | TensorRT推理 |
| C-56 | [Speculative Decoding for Accelerated LLM Inference](https://arxiv.org/abs/2211.17192) | Leviathan et al. | 2023 | 投机解码 |
| C-57 | [Quantization and Training of Neural Networks for Efficient Integer-Arithmetic-Only Inference](https://arxiv.org/abs/1712.05877) | Jacob et al. | 2018 | 量化推理 |
| C-58 | [Knowledge Distillation: A Survey](https://arxiv.org/abs/2006.05525) | Gou et al. | 2021 | 知识蒸馏综述 |
| C-59 | [MLSys: The New Frontier of Machine Learning Systems](https://arxiv.org/abs/1904.03257) | - | 2019 | ML系统综述 |
| C-60 | [Efficient Transformers: A Survey](https://arxiv.org/abs/2009.00032) | Tay et al. | 2022 | 高效Transformer综述 |

#### C61-C70: AI产品、商业化与伦理

| 编号 | 论文 | 作者 | 年份 | 核心主题 |
|------|------|------|------|----------|
| C-61 | [On the Opportunities and Risks of Foundation Models](https://arxiv.org/abs/2108.07258) | Bommasani et al. | 2021 | 基础模型综述 |
| C-62 | [AI Alignment: A Comprehensive Survey](https://arxiv.org/abs/2310.19852) | Ji et al. | 2023 | AI对齐综述 |
| C-63 | [Fairness and Abstraction in Sociotechnical Systems](https://dl.acm.org/doi/10.1145/3287560.3287598) | Selbst et al. | 2019 | AI公平性 |
| C-64 | [The Stochastic Parrots Perspective: LLM Risks and Ethics](https://dl.acm.org/doi/10.1145/3442188.3445922) | Bender et al. | 2021 | LLM伦理风险 |
| C-65 | [Challenges in Deploying Machine Learning: A Survey of Case Studies](https://arxiv.org/abs/2011.09926) | Amershi et al. | 2020 | ML部署挑战 |
| C-66 | [AI Product Management: From Concept to Launch](https://arxiv.org/abs/2209.06002) | - | 2022 | AI产品管理 |
| C-67 | [The Business of AI: A Strategic Perspective](https://hbr.org/2018/07/artificial-intelligence-for-the-real-world) | Davenport & Ronanki | 2018 | AI商业战略 |
| C-68 | [China's AI Development Report 2024](https://www.tsinghua.edu.cn/info/1175/108518.htm) | Tsinghua | 2024 | 中国AI发展 |
| C-69 | [Global AI Index 2024](https://www.tortoisemedia.com/intelligence/global-ai/) | Tortoise Media | 2024 | 全球AI指数 |
| C-70 | [AI Compute Trends and Implications](https://epochai.org/blog/compute-trends) | Epoch AI | 2024 | AI算力趋势 |

---

## 🔗 与具身智能的衔接

本知识库与 [LinuxHermes](https://github.com/ly1992921/LinuxHermes)（具身智能学习仓库）的衔接点：

| 本仓库章节 | LinuxHermes对应内容 | 衔接说明 |
|-----------|-------------------|----------|
| 05-大模型 | 具身智能精读/VLA | 理解LLM如何赋能机器人 |
| 06-强化学习 | RobotPaper/RL论文 | 机器人控制中的RL应用 |
| 08-AI芯片 | LearningRecording/硬件篇 | 机器人计算平台选型 |
| 09-产品策略 | PM学习配套册 | 从AI产品到机器人产品 |
| 10-前沿精读 | 具身智能精读与目录 | 论文阅读方法论共享 |

---

## 📊 学习进度追踪

使用以下模板记录学习进度：

```markdown
### 学习记录模板

**日期**: YYYY-MM-DD
**章节**: 章节名
**论文**: 论文标题
**状态**: 未开始 / 进行中 / 已完成
**笔记链接**: ./notes/xxx.md
**核心收获**:
- 要点1
- 要点2
**疑问与待解决**:
- 问题1
```

---

## 🔗 资源索引

### 在线课程
- [Stanford CS229: Machine Learning](http://cs229.stanford.edu/)
- [Stanford CS224N: NLP with Deep Learning](https://web.stanford.edu/class/cs224n/)
- [Stanford CS231n: Convolutional Neural Networks](http://cs231n.stanford.edu/)
- [Fast.ai: Practical Deep Learning](https://www.fast.ai/)
- [MIT 6.034: Artificial Intelligence](https://ocw.mit.edu/courses/6-034-artificial-intelligence-fall-2010/)

### 推荐书籍
- 《深度学习》(Goodfellow) — 理论基石
- 《Hands-On Machine Learning》(Aurelien Geron) — 实践指南
- 《Designing Machine Learning Systems》(Chip Huyen) — AI系统设计
- 《Build a Large Language Model (From Scratch)》(Sebastian Raschka) — LLM从零构建

### 社区与资讯
- [Papers With Code](https://paperswithcode.com/) — 论文+代码
- [arXiv Sanity Preserver](http://www.arxiv-sanity.com/) — 论文推荐
- [Hugging Face](https://huggingface.co/) — 模型与数据集
- [MLSys](https://mlsys.org/) — ML系统会议

---

> 📝 **维护说明**: 本知识库由 `ly1992921` 维护，内容随学习进度持续更新。
> 
> 建议配合 Obsidian 或 VS Code 使用，将仓库 clone 到本地作为 vault/workspace。
>
> 最后更新: 2026-06-30
