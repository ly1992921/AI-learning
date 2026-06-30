# 08 - AI芯片与硬件

> AI芯片是AI产品的算力底座。作为PM，理解硬件约束是设计可行产品方案的前提。
>
> **与Chip-Learning仓库的衔接**: 本章聚焦AI PM视角的芯片认知，Chip-Learning提供更深入的芯片设计知识。

## 📚 本章节学习目标

- [ ] 理解GPU/TPU/NPU的架构差异
- [ ] 掌握AI推理的性能指标（Latency/Throughput/Power）
- [ ] 理解模型量化和部署优化
- [ ] 能根据产品需求选择合适的硬件平台

## 🎯 核心知识点

### 8.1 AI芯片架构概览
- GPU: CUDA Core, Tensor Core, NVLink
- TPU: Matrix Multiply Unit, HBM, MXU
- NPU/ASIC: 脉动阵列、专用加速器
- FPGA: 可重构计算
- **PM视角**: 不同硬件适合什么场景？

### 8.2 性能指标与评估
- FLOPS vs 实际吞吐量
- 内存带宽瓶颈
- 功耗与能效比(TOPS/W)
- 延迟 vs 吞吐量权衡
- **PM视角**: 如何给技术团队提合理的性能需求

### 8.3 模型优化技术
- 量化(INT8/INT4/FP16)
- 剪枝(Pruning)
- 知识蒸馏(Knowledge Distillation)
- 算子融合
- **PM视角**: 优化对精度的影响 vs 性能收益

### 8.4 部署平台对比
- 云端: NVIDIA A100/H100, AWS Trainium/Inferentia
- 边缘: NVIDIA Jetson, 高通骁龙, 地平线征程
- 端侧: Apple Neural Engine, 联发科APU
- **PM视角**: 成本、性能、功耗的三角权衡

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 论文 | TPU架构论文 | [arXiv](https://arxiv.org/abs/1704.04760) | ⭐⭐⭐ |
| 论文 | FlashAttention | [arXiv](https://arxiv.org/abs/2205.14135) | ⭐⭐⭐ |
| 报告 | MLPerf Benchmark | [MLPerf](https://mlcommons.org/benchmarks/) | ⭐⭐ |
| 教程 | NVIDIA深度学习性能指南 | [NVIDIA](https://docs.nvidia.com/deeplearning/performance/index.html) | ⭐⭐⭐ |
| 博客 | 量化推理入门 | [TensorFlow](https://www.tensorflow.org/lite/performance/post_training_quantization) | ⭐⭐ |
| 对比 | AI芯片性能对比 | [Hugging Face](https://huggingface.co/spaces/optimum/llm-perf-leaderboard) | ⭐ |

## 📝 学习产出模板

```markdown
## 学习笔记: [AI芯片主题]

**日期**: YYYY-MM-DD

### 硬件架构理解
[简述架构特点和适用场景]

### 性能分析
- 理论峰值: X TFLOPS
- 内存带宽: Y GB/s
- 典型功耗: Z W

### 产品选型思考
[如果我要做一个XX产品，应该选什么硬件？]
```

## ✅ 检查清单

- [ ] 理解GPU/TPU/NPU架构差异
- [ ] 掌握AI推理关键性能指标
- [ ] 理解量化/剪枝/蒸馏的基本原理
- [ ] 能对比至少5种边缘AI平台
- [ ] 阅读至少2篇AI芯片架构论文
