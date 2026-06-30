# 04 - 计算机视觉

> 计算机视觉是AI最成熟的应用领域之一。从图像分类到生成式AI，CV技术正在重塑无数行业。

## 📚 本章节学习目标

- [ ] 理解CNN架构演进史（LeNet → ResNet → EfficientNet）
- [ ] 掌握目标检测、图像分割、图像生成技术
- [ ] 理解Vision Transformer的原理
- [ ] 能评估CV产品技术方案

## 🎯 核心知识点

### 4.1 CNN架构演进
- LeNet (1998): 开创性工作
- AlexNet (2012): 深度学习复兴
- VGGNet (2014): 深度与规律
- ResNet (2015): 残差连接
- EfficientNet (2019): 复合缩放
- **PM视角**: 模型选型时的效率权衡

### 4.2 目标检测与分割
- R-CNN系列（两阶段）
- YOLO系列（单阶段）
- 语义分割 (FCN, U-Net)
- 实例分割 (Mask R-CNN)
- SAM (Segment Anything)
- **PM视角**: 精度 vs 速度的产品化取舍

### 4.3 图像生成
- GAN (StyleGAN, BigGAN)
- VAE与扩散模型
- Stable Diffusion架构
- ControlNet条件控制
- **PM视角**: 生成式AI的产品机会与风险

### 4.4 Vision Transformer
- ViT: Patch Embedding + Transformer
- Swin Transformer: 层次化设计
- MAE: 掩码自编码器
- **PM视角**: 何时用CNN，何时用ViT

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 论文 | AlexNet | [NeurIPS](https://papers.nips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html) | ⭐⭐ |
| 论文 | ResNet | [arXiv](https://arxiv.org/abs/1512.03385) | ⭐⭐ |
| 论文 | ViT | [arXiv](https://arxiv.org/abs/2010.11929) | ⭐⭐⭐ |
| 论文 | Stable Diffusion | [arXiv](https://arxiv.org/abs/2112.10752) | ⭐⭐⭐ |
| 课程 | Stanford CS231n | [斯坦福](http://cs231n.stanford.edu/) | ⭐⭐⭐ |
| 教程 | YOLO官方文档 | [Ultralytics](https://docs.ultralytics.com/) | ⭐⭐ |

## ✅ 检查清单

- [ ] 理解CNN架构演进的关键节点
- [ ] 理解目标检测两阶段 vs 单阶段的区别
- [ ] 理解扩散模型的基本原理
- [ ] 理解ViT的设计思想
- [ ] 体验至少3个CV产品（图像生成、检测等）
