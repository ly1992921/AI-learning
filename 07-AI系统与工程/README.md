# 07 - AI系统与工程

> 大模型的训练和推理是系统工程问题。理解系统层面的挑战，是AI PM的必备能力。

## 📚 本章节学习目标

- [ ] 理解分布式训练的基本原理
- [ ] 掌握模型推理优化的核心技术
- [ ] 了解MLOps和模型生命周期管理
- [ ] 能评估AI系统的工程方案

## 🎯 核心知识点

### 7.1 分布式训练
- 数据并行 vs 模型并行
- 流水线并行
- ZeRO优化
- Megatron-LM
- **PM视角**: 训练成本和时间的估算

### 7.2 推理优化
- KV Cache管理
- 量化(INT8/INT4/AWQ/GPTQ)
- 连续批处理(Continuous Batching)
- 投机解码(Speculative Decoding)
- **PM视角**: 推理成本直接决定产品经济性

### 7.3 推理服务系统
- vLLM: PagedAttention
- TensorRT-LLM
- TGI (Text Generation Inference)
- **PM视角**: 服务性能的关键指标

### 7.4 MLOps
- 模型版本管理
- A/B测试与实验追踪
- 监控与可观测性
- 模型降级与回滚
- **PM视角**: AI产品的持续交付

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 论文 | FlashAttention | [arXiv](https://arxiv.org/abs/2205.14135) | ⭐⭐⭐ |
| 论文 | vLLM | [arXiv](https://arxiv.org/abs/2309.06180) | ⭐⭐⭐ |
| 论文 | Megatron-LM | [arXiv](https://arxiv.org/abs/2104.04473) | ⭐⭐⭐ |
| 博客 | LLM推理优化指南 | [Anyscale](https://www.anyscale.com/blog/continuous-batching-llm-inference) | ⭐⭐ |
| 工具 | vLLM文档 | [vLLM](https://docs.vllm.ai/) | ⭐⭐ |
| 课程 | MLSys课程 | [Stanford](https://mlsys.stanford.edu/) | ⭐⭐⭐ |

## ✅ 检查清单

- [ ] 理解数据并行和模型并行的区别
- [ ] 理解KV Cache的作用
- [ ] 理解量化的基本原理和精度影响
- [ ] 了解主流推理框架(vLLM/TensorRT-LLM)
- [ ] 理解MLOps的核心流程
