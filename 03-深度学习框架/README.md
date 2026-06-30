# 03 - 深度学习框架

> 深度学习是现代AI的核心引擎。理解神经网络的基本原理，是理解大模型的前提。

## 📚 本章节学习目标

- [ ] 理解神经网络的基本结构和工作原理
- [ ] 掌握反向传播算法的直觉
- [ ] 熟悉PyTorch/TensorFlow基本使用
- [ ] 理解激活函数、损失函数、优化器的作用

## 🎯 核心知识点

### 3.1 神经网络基础
- 感知机与多层感知机(MLP)
- 前向传播与反向传播
- 激活函数：ReLU、Sigmoid、Tanh、Softmax
- 损失函数：MSE、Cross-Entropy
- **PM视角**: 理解模型为什么能"学习"

### 3.2 训练技巧
- 权重初始化
- 批归一化(Batch Normalization)
- Dropout正则化
- 学习率调度
- 梯度消失/爆炸
- **PM视角**: 理解训练不收敛的常见原因

### 3.3 深度学习框架
- PyTorch vs TensorFlow vs JAX
- 自动微分机制
- 模型定义、训练、评估流程
- **PM视角**: 了解框架选择对工程效率的影响

### 3.4 模型部署基础
- 模型导出格式(ONNX, TorchScript)
- 推理优化基础
- **PM视角**: 理解从训练到部署的完整链路

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 教程 | PyTorch官方教程 | [PyTorch](https://pytorch.org/tutorials/) | ⭐⭐ |
| 教程 | Fast.ai课程 | [Fast.ai](https://www.fast.ai/) | ⭐⭐ |
| 课程 | Stanford CS231n | [斯坦福](http://cs231n.stanford.edu/) | ⭐⭐⭐ |
| 书籍 | 《动手学深度学习》 | [李沐](https://zh.d2l.ai/) | ⭐⭐ |
| 可视化 | Neural Network Playground | [Google](https://playground.tensorflow.org/) | ⭐ |

## 📝 学习产出模板

```markdown
## 学习笔记: [主题]

**日期**: YYYY-MM-DD

### 核心概念
[用自己的话解释]

### 可视化理解
[如果可以，画一个简图]

### PM视角
[这个概念对产品设计有什么影响？]
```

## ✅ 检查清单

- [ ] 理解MLP结构和反向传播
- [ ] 掌握至少一种深度学习框架
- [ ] 理解各种激活函数的适用场景
- [ ] 能训练一个简单的神经网络
- [ ] 理解模型部署的基本流程
