# 《The Hitchhiker's Guide to Agentic AI》— 整车PM视角精读系列

> 面向整车三电产品经理的精读翻译：用三电系统和整车的语言翻译每一个AI/算法/计算机概念。

---

## 精读清单（17篇）

| 编号 | 标题 | 来源章节 | 核心主题 |
|------|------|---------|---------|
| A00 | [全书导论](./精读A00_全书导论_整车PM视角.md) | Preface + Introduction | 全书架构、AI里程碑、PM阅读路线图 |
| A01 | [LLM架构：Tokenization与Transformer](./精读A01_LLM架构_Tokenization与Transformer.md) | Ch1 (part 1) | 分词、自注意力、位置编码、FFN |
| A02 | [LLM训练优化：FlashAttention与LoRA](./精读A02_LLM训练优化_FlashAttention与LoRA.md) | Ch1 (part 2) | AdamW、FlashAttention、LoRA、MoE、量化、投机解码 |
| A02 | [GPU系统基础与分布式训练](./精读A02_GPU系统基础与分布式训练.md) | Ch2 | GPU架构、内存层次、NVLink、vLLM |
| A03 | [强化学习入门](./精读A03_强化学习入门.md) | Ch3 | MDP、Q-Learning、策略梯度、Actor-Critic |
| A04 | [RLHF与PPO](./精读A04_RLHF与PPO.md) | Ch4-5 | 三阶段流程、奖励模型、KL散度 |
| A05 | [DPO与直接偏好优化](./精读A05_DPO与直接偏好优化.md) | Ch6 | Bradley-Terry、Online DPO、KTO |
| A06 | [推理模型：o1/R1与思维链](./精读A06_推理模型与思维链.md) | Ch13 | 思维链、Test-Time Scaling、MCTS |
| A07 | [LLM评估体系](./精读A07_LLM评估体系.md) | Ch14 | ELO排名、LLM-as-Judge、基准泄露 |
| A08 | [RAG检索增强与智能体导论](./精读A08_RAG检索增强与智能体导论.md) | Ch15-16 | 检索方法、Chunking、Agentic RAG |
| A09 | [GRPO与高级强化学习](./精读A09_GRPO与高级强化学习.md) | Ch7 | 组相对策略优化、DeepSeek-R1 |
| A10 | [智能体记忆系统与编排](./精读A10_智能体记忆系统与编排.md) | Ch17-18 | 四种记忆、Agent Harness、ReAct |
| A11 | [MCP协议与A2A通信](./精读A11_MCP协议与A2A通信.md) | Ch21 + 23 | 工具标准协议、智能体通信 |
| A12 | [多智能体系统与开发框架](./精读A12_多智能体系统与开发框架.md) | Ch24-25 | 架构类型、LangGraph、AutoGen |
| A13 | [智能体设计模式与环境](./精读A13_智能体设计模式与环境.md) | Ch19-20 | 工作流模式、OpenEnv |
| A14 | [智能体UI框架与未来展望](./精读A14_智能体UI框架与未来展望.md) | Ch26 + 29 | Generative UI、流式渲染、开放挑战 |
| A15 | [Agent技能体系与设计原则](./精读A15_Agent技能与设计原则.md) | Ch22 + 19 | Skill生命周期、vs Fine-Tuning |

---

## 三电类比速查表

每个精读文档都用三电/整车类比翻译技术概念，以下是一些贯穿全书的比喻：

| AI概念 | 三电类比 |
|--------|---------|
| Transformer架构 | 电驱系统设计 |
| GPU硬件 | 芯片/IGBT |
| Tokenization | 传感器采样+ADC转换 |
| Self-Attention | 团队会议——每个人都能听到所有人 |
| Embedding | BMS特性曲线查表 |
| LoRA | OTA刷写——不改硬件只改标定 |
| MoE | 混动系统——总功率大但实际只用部分 |
| FlashAttention | 优化数据搬运而不是计算速度 |
| RLHF | 路试验证+OTA优化 |
| DPO | 简化版OTA |
| RAG | 查手册而不是全靠记忆 |
| MCP | CAN/LIN总线协议 |
| A2A | V2X车联网通信 |
| Agent Harness | VCU整车控制器 |
| Multi-Agent | 车队协同调度 |
| Quantization | RAW图压成清晰JPG |

---

*生成日期：2026-07-01*
