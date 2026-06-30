# AI-learning 论文与报告索引

> 100篇必读文献，按重要性分为A/B/C三级
> 面向AI系统产品经理（三电/ToB产品背景转型）
> 覆盖：基础模型与深度学习 | NLP与大模型 | 计算机视觉 | 多模态 | 强化学习与具身智能 | AI系统与工程 | AI芯片与硬件 | 商业化与产品策略 | AI安全与对齐

---

## 🏆 A级必读（10篇）

| 编号 | 论文标题 | 作者/来源 | 年份 | 核心贡献 | 链接 |
|------|----------|-----------|------|----------|------|
| A-01 | Attention Is All You Need | Vaswani et al. (Google) | 2017 | 提出Transformer架构，用自注意力机制替代循环和卷积，奠定了现代大模型的基础范式 | [arXiv](https://arxiv.org/abs/1706.03762) |
| A-02 | ImageNet Classification with Deep CNNs (AlexNet) | Krizhevsky et al. | 2012 | 首个在大规模图像识别上压倒传统方法的深度CNN，引爆深度学习革命 | [NeurIPS](https://proceedings.neurips.cc/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf) |
| A-03 | Deep Residual Learning for Image Recognition (ResNet) | He et al. (Microsoft) | 2015 | 提出残差连接（Skip Connection），让训练152层深度网络成为可能，CV领域里程碑 | [arXiv](https://arxiv.org/abs/1512.03385) |
| A-04 | BERT: Pre-training of Deep Bidirectional Transformers | Devlin et al. (Google) | 2018 | 提出双向预训练+Masked LM，在11项NLP任务上创造SOTA，NLP的ImageNet时刻 | [arXiv](https://arxiv.org/abs/1810.04805) |
| A-05 | Language Models are Few-Shot Learners (GPT-3) | Brown et al. (OpenAI) | 2020 | 175B参数大模型展示出强大的Few-Shot和In-Context Learning能力，开启LLM时代 | [arXiv](https://arxiv.org/abs/2005.14165) |
| A-06 | Deep Learning (Nature 2015) | LeCun, Bengio & Hinton | 2015 | 深度学习三巨头联合综述，系统定义了深度学习的研究框架和核心方法 | [Nature](https://www.nature.com/articles/nature14539) |
| A-07 | Mastering the Game of Go with Deep Neural Networks (AlphaGo) | Silver et al. (DeepMind) | 2016 | DL+RL+MCTS融合，在围棋上击败人类冠军，引爆全球AI热潮 | [Nature](https://www.nature.com/articles/nature16961) |
| A-08 | Generative Adversarial Nets (GAN) | Goodfellow et al. | 2014 | 提出生成器+判别器的对抗训练框架，开创生成式AI研究范式 | [arXiv](https://arxiv.org/abs/1406.2661) |
| A-09 | Training language models to follow instructions (InstructGPT/RLHF) | Ouyang et al. (OpenAI) | 2022 | 提出RLHF对齐技术（SFT+RM+PPO），让LLM变得"听话"，ChatGPT的核心技术基础 | [arXiv](https://arxiv.org/abs/2203.02155) |
| A-10 | Learning Transferable Visual Models From Natural Language Supervision (CLIP) | Radford et al. (OpenAI) | 2021 | 4亿图文对的对比学习模型，实现零样本视觉分类，开创多模态AI产品范式 | [arXiv](https://arxiv.org/abs/2103.00020) |

---

## 📘 B级推荐（20篇）

### 基础模型与深度学习

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-01 | Very Deep CNNs for Large-Scale Image Recognition (VGGNet) | Simonyan & Zisserman (Oxford) | 2014 | 小卷积核(3×3)深度堆叠的有效性 | [arXiv](https://arxiv.org/abs/1409.1556) |
| B-02 | Going Deeper with Convolutions (GoogLeNet/Inception) | Szegedy et al. (Google) | 2014 | Inception模块，多尺寸卷积核并行 | [arXiv](https://arxiv.org/abs/1409.4842) |
| B-03 | Gradient-Based Learning Applied to Doc Recognition (LeNet-5) | LeCun et al. | 1998 | 第一个成功应用的CNN | [IEEE](http://yann.lecun.com/exdb/publis/pdf/lecun-01a.pdf) |

### 多模态与生成

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-04 | Denoising Diffusion Probabilistic Models (DDPM) | Ho et al. (UC Berkeley) | 2020 | 去噪扩散概率模型，Stable Diffusion理论基础 | [arXiv](https://arxiv.org/abs/2006.11239) |
| B-05 | An Image is Worth 16x16 Words (ViT) | Dosovitskiy et al. (Google) | 2020 | Transformer应用于图像分类 | [arXiv](https://arxiv.org/abs/2010.11929) |
| B-06 | Segment Anything (SAM) | Kirillov et al. (Meta) | 2023 | 通用图像分割模型，零样本分割 | [arXiv](https://arxiv.org/abs/2304.02643) |

### NLP与大模型

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-07 | Scaling Laws for Neural Language Models | Kaplan et al. (OpenAI) | 2020 | 性能随参数量/数据量的幂律关系 | [arXiv](https://arxiv.org/abs/2001.08361) |
| B-08 | Constitutional AI: Harmlessness from AI Feedback | Bai et al. (Anthropic) | 2022 | 宪法AI方法，减少RLHF人工标注依赖 | [arXiv](https://arxiv.org/abs/2212.08073) |
| B-09 | Llama 2: Open Foundation and Fine-Tuned Chat Models | Touvron et al. (Meta) | 2023 | 开源大模型，推动LLM私有化部署 | [arXiv](https://arxiv.org/abs/2307.09288) |
| B-10 | Chain-of-Thought Prompting Elicits Reasoning in LLMs | Wei et al. (Google) | 2022 | 思维链推理，提升LLM复杂推理能力 | [arXiv](https://arxiv.org/abs/2201.11903) |

### 强化学习与具身智能

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-11 | Human-level control through deep RL (DQN) | Mnih et al. (DeepMind) | 2015 | DL+RL首次结合，Atari游戏人类水平 | [Nature](https://www.nature.com/articles/nature14236) |
| B-12 | Mastering Go without Human Knowledge (AlphaGo Zero) | Silver et al. (DeepMind) | 2017 | 完全自我对弈学习 | [Nature](https://www.nature.com/articles/nature24270) |
| B-13 | RT-2: Vision-Language-Action Models | Brohan et al. (Google DeepMind) | 2023 | VLM改造为机器人控制策略 | [arXiv](https://arxiv.org/abs/2307.15818) |
| B-14 | Decision Transformer: RL via Sequence Modeling | Chen et al. (UC Berkeley) | 2021 | Transformer做强化学习 | [arXiv](https://arxiv.org/abs/2106.01345) |

### AI系统与工程

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-15 | RAG: Retrieval-Augmented Generation | Lewis et al. (Facebook) | 2020 | 检索增强生成，减少LLM幻觉 | [arXiv](https://arxiv.org/abs/2005.11401) |
| B-16 | LoRA: Low-Rank Adaptation of LLMs | Hu et al. (Microsoft) | 2021 | 参数高效微调，降低大模型微调成本 | [arXiv](https://arxiv.org/abs/2106.09685) |
| B-17 | Megatron-LM: Training Multi-Billion Parameter LMs | Shoeybi et al. (NVIDIA) | 2019 | 大规模分布式训练框架 | [arXiv](https://arxiv.org/abs/1909.08053) |
| B-18 | FlashAttention: Fast Memory-Efficient Attention | Dao et al. (Stanford) | 2022 | IO感知注意力算法 | [arXiv](https://arxiv.org/abs/2205.14135) |

### AI芯片与硬件

| 编号 | 标题 | 作者/来源 | 年份 | 核心主题 | 链接 |
|------|------|-----------|------|----------|------|
| B-19 | AI Accelerators Survey (Reuther, 2022) | MIT | 2022 | AI加速器全景综述 | [arXiv](https://arxiv.org/abs/2209.11137) |
| B-20 | MLPerf Inference Benchmark | MLCommons | 2020 | AI芯片性能评测标准 | [MLCommons](https://mlcommons.org/benchmarks/) |

---

## 📖 C级参考（70篇）

### 基础模型与深度学习

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-01 | Network In Network (Min Lin, ICLR 2014) | MLP卷积核+全局平均池化 | [arXiv](https://arxiv.org/abs/1312.4400) |
| C-02 | Batch Normalization (Ioffe, ICML 2015) | 批归一化加速训练 | [arXiv](https://arxiv.org/abs/1502.03167) |
| C-03 | Dropout (Srivastava, JMLR 2014) | Dropout正则化 | [JMLR](https://jmlr.org/papers/v15/srivastava14a.html) |
| C-04 | MobileNets (Howard, 2017) | 深度可分离卷积 | [arXiv](https://arxiv.org/abs/1704.04861) |
| C-05 | EfficientNet (Tan, ICML 2019) | 复合缩放 | [arXiv](https://arxiv.org/abs/1905.11946) |
| C-06 | SqueezeNet (Iandola, 2016) | 极小模型设计 | [arXiv](https://arxiv.org/abs/1602.07360) |
| C-07 | YOLOv4 (Bochkovskiy, 2020) | 实时目标检测 | [arXiv](https://arxiv.org/abs/2004.10934) |
| C-08 | Mask R-CNN (He, ICCV 2017) | 实例分割 | [arXiv](https://arxiv.org/abs/1703.06870) |

### NLP与大模型

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-09 | GPT-1 (Radford, 2018) | 生成式预训练 | [OpenAI](https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf) |
| C-10 | GPT-2 (Radford, 2019) | 大规模生成式预训练 | [OpenAI](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) |
| C-11 | PaLM (Chowdhery, Google, 2022) | 540B大模型 | [arXiv](https://arxiv.org/abs/2204.02311) |
| C-12 | LLaMA (Touvron, Meta, 2023) | 开源大模型 | [arXiv](https://arxiv.org/abs/2302.13971) |
| C-13 | Chinchilla (Hoffmann, DeepMind, 2022) | 最优计算量模型 | [arXiv](https://arxiv.org/abs/2203.15556) |
| C-14 | RAG Survey (Gao, 2023) | RAG技术综述 | [arXiv](https://arxiv.org/abs/2312.10997) |
| C-15 | PEFT Survey (Lialin, 2023) | 参数高效微调综述 | [arXiv](https://arxiv.org/abs/2303.15647) |
| C-16 | A Survey of LLMs (Zhao, 2023) | 大模型综述 | [arXiv](https://arxiv.org/abs/2303.18223) |
| C-17 | Direct Preference Optimization (DPO) (Rafailov, 2023) | 直接偏好优化 | [arXiv](https://arxiv.org/abs/2305.18290) |
| C-18 | ReAct (Yao, ICLR 2023) | CoT+Action结合 | [arXiv](https://arxiv.org/abs/2210.03629) |
| C-19 | Toolformer (Schick, 2023) | LLM使用工具 | [arXiv](https://arxiv.org/abs/2302.04761) |

### 计算机视觉

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-20 | Faster R-CNN (Ren, NeurIPS 2015) | 区域提议网络 | [arXiv](https://arxiv.org/abs/1506.01497) |
| C-21 | YOLO (Redmon, 2016) | 单阶段检测 | [arXiv](https://arxiv.org/abs/1506.02640) |
| C-22 | DETR (Carion, 2020) | Transformer检测 | [arXiv](https://arxiv.org/abs/2005.12872) |
| C-23 | NeRF (Mildenhall, ECCV 2020) | 神经辐射场 | [arXiv](https://arxiv.org/abs/2003.08934) |
| C-24 | 3D Gaussian Splatting (Kerbl, 2023) | 实时辐射场渲染 | [arXiv](https://arxiv.org/abs/2308.04079) |
| C-25 | StyleGAN (Karras, CVPR 2019) | 高质量图像生成 | [arXiv](https://arxiv.org/abs/1812.04948) |
| C-26 | DDIM (Song, 2020) | 加速扩散模型 | [arXiv](https://arxiv.org/abs/2010.02502) |
| C-27 | Latent Diffusion Models (Rombach, CVPR 2022) | 潜在空间扩散 | [arXiv](https://arxiv.org/abs/2112.10752) |

### 多模态

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-28 | LLaVA (Liu, 2023) | 视觉指令微调 | [arXiv](https://arxiv.org/abs/2304.08485) |
| C-29 | BLIP-2 (Li, 2023) | 高效多模态预训练 | [arXiv](https://arxiv.org/abs/2301.12597) |
| C-30 | Flamingo (Alayrac, DeepMind, 2022) | 少样本视觉语言 | [arXiv](https://arxiv.org/abs/2204.14198) |
| C-31 | GPT-4 Technical Report (OpenAI, 2023) | GPT-4技术报告 | [arXiv](https://arxiv.org/abs/2303.08774) |
| C-32 | GPT-4V System Card (OpenAI, 2023) | GPT-4视觉能力 | [OpenAI](https://cdn.openai.com/papers/GPTV_System_Card.pdf) |
| C-33 | Gemini (Google, 2023) | 多模态大模型 | [arXiv](https://arxiv.org/abs/2312.11805) |

### 强化学习与具身智能

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-34 | MuZero (DeepMind, 2020) | 无模型规划算法 | [Nature](https://www.nature.com/articles/s41586-020-03051-4) |
| C-35 | DreamerV3 (DeepMind, 2023) | 世界模型RL | [arXiv](https://arxiv.org/abs/2301.04104) |
| C-36 | Imitation Learning Survey (2023) | 模仿学习综述 | [arXiv](https://arxiv.org/abs/2308.08512) |
| C-37 | Offline RL Tutorial (Levine, 2020) | 离线RL教程 | [arXiv](https://arxiv.org/abs/2005.01643) |
| C-38 | Open X-Embodiment (2023) | 跨机器人数据集 | [arXiv](https://arxiv.org/abs/2310.08864) |
| C-39 | Diffusion Policy (Chi, RSS 2023) | 扩散模型机器人策略 | [arXiv](https://arxiv.org/abs/2303.04137) |
| C-40 | Mobile ALOHA (2023) | 低成本双臂操作 | [arXiv](https://arxiv.org/abs/2401.02117) |
| C-41 | Octo: Generalist Robot Policy (2023) | 开源通用策略 | [arXiv](https://arxiv.org/abs/2405.12213) |

### AI系统与工程

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-42 | Distributed Training Survey (2023) | 分布式训练综述 | [arXiv](https://arxiv.org/abs/2310.03379) |
| C-43 | vLLM: PagedAttention (2023) | LLM推理服务 | [arXiv](https://arxiv.org/abs/2309.06180) |
| C-44 | TensorRT (NVIDIA) | 推理优化 | [NVIDIA](https://developer.nvidia.com/tensorrt) |
| C-45 | ONNX Exchange Format | 模型交换格式 | [ONNX](https://onnx.ai/) |
| C-46 | MLOps Maturity Model (Google) | MLOps成熟度 | [Google Cloud](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) |
| C-47 | Hidden Tech Debt in ML (Sculley, NIPS 2015) | ML系统技术债务 | [NIPS](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) |
| C-48 | PyTorch Distributed (Li, VLDB 2020) | PyTorch分布式 | [arXiv](https://arxiv.org/abs/2006.15704) |
| C-49 | Quantization Survey (Gholami, 2021) | 模型量化综述 | [arXiv](https://arxiv.org/abs/2103.13630) |
| C-50 | LLM Inference Survey (2023) | LLM推理综述 | [arXiv](https://arxiv.org/abs/2311.07231) |

### AI芯片与硬件

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-51 | Eyeriss v2 (Chen, JSSC 2019) | 可重构DNN加速器 | [arXiv](https://arxiv.org/abs/1807.07928) |
| C-52 | NVIDIA DGX SuperPOD | AI基础设施 | [NVIDIA](https://www.nvidia.com/en-us/data-center/dgx-superpod/) |
| C-53 | TPU v4 (Google, ISCA 2022) | TPU v4架构 | [arXiv](https://arxiv.org/abs/2204.09726) |
| C-54 | Edge AI Chip Guide (2023) | 边缘AI芯片评估 | [Edge AI](https://www.edge-ai-vision.com/) |

### 商业化与产品策略

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-55 | Business of AI (HBR, 2017) | AI商业价值 | [HBR](https://hbr.org/2017/07/the-business-of-artificial-intelligence) |
| C-56 | AI PM Guide (2023) | AI产品经理指南 | [Product School](https://productschool.com/blog/ai-product-management) |
| C-57 | Economics of AI (Agrawal, 2018) | AI经济学 | [Book](https://www.predictionmachines.ai/) |
| C-58 | AI Startup Landscape (CB Insights) | AI创业地图 | [CB Insights](https://www.cbinsights.com/research/artificial-intelligence-startup-market-map/) |
| C-59 | LLM Pricing Comparison (2024) | LLM定价对比 | [GitHub](https://github.com/ray-project/llm-budget) |

### AI安全与对齐

| 编号 | 标题 | 核心主题 | 链接 |
|------|------|----------|------|
| C-60 | Concrete Problems in AI Safety (Amodei, 2016) | AI安全问题 | [arXiv](https://arxiv.org/abs/1606.06565) |
| C-61 | AI Alignment Survey (Ji, 2023) | 对齐问题综述 | [arXiv](https://arxiv.org/abs/2310.19852) |
| C-62 | The Alignment Problem (Christian, 2020) | 对齐问题解读 | [Book](https://brianchristian.org/the-alignment-problem/) |

---

> 最后更新：2026-06-30
