# 06 - 强化学习与具身智能

> 强化学习是AI走向自主决策的核心。具身智能(Embodied AI)是RL在物理世界的延伸。
>
> **与LinuxHermes的衔接**: 本章是AI基础向具身智能的桥梁，LinuxHermes提供更深入的机器人学习资料。

## 📚 本章节学习目标

- [ ] 理解MDP和强化学习的基本框架
- [ ] 掌握Q-Learning、Policy Gradient等算法
- [ ] 理解VLA (Vision-Language-Action)模型
- [ ] 能分析具身智能产品的技术方案

## 🎯 核心知识点

### 6.1 强化学习基础
- 马尔可夫决策过程(MDP)
- 值函数与策略
- Q-Learning与SARSA
- 策略梯度(Policy Gradient)
- Actor-Critic架构
- **PM视角**: RL适合什么类型的问题？

### 6.2 深度强化学习
- DQN与经验回放
- A3C/A2C异步训练
- PPO: 近端策略优化
- SAC: 软 Actor-Critic
- **PM视角**: 训练不稳定的挑战

### 6.3 具身智能
- Sim-to-Real迁移
- 模仿学习(Imitation Learning)
- VLA模型 (RT-1/RT-2/OpenVLA)
- 世界模型(World Models)
- Diffusion Policy
- **PM视角**: 具身智能产品的独特挑战

### 6.4 机器人学习系统
- 数据收集与标注
- 仿真环境 (MuJoCo, Isaac Sim)
- 多模态感知融合
- 安全与可靠性
- **PM视角**: 机器人PM的核心关注点

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 论文 | DQN (Nature) | [Nature](https://www.nature.com/articles/nature14236) | ⭐⭐⭐ |
| 论文 | PPO | [arXiv](https://arxiv.org/abs/1707.06347) | ⭐⭐⭐ |
| 论文 | RT-1 | [arXiv](https://arxiv.org/abs/2212.06817) | ⭐⭐⭐ |
| 论文 | RT-2 | [arXiv](https://arxiv.org/abs/2307.15818) | ⭐⭐⭐ |
| 课程 | UC Berkeley CS285 | [课程](http://rail.eecs.berkeley.edu/deeprlcourse/) | ⭐⭐⭐ |
| 仓库 | LinuxHermes | [GitHub](https://github.com/ly1992921/LinuxHermes) | ⭐⭐⭐ |

## ✅ 检查清单

- [ ] 理解MDP框架
- [ ] 理解DQN和Policy Gradient
- [ ] 理解Sim-to-Real问题
- [ ] 理解VLA模型架构
- [ ] 阅读LinuxHermes中的机器人论文
