# 05 - 自然语言处理与大模型

> 大语言模型(LLM)是当前AI浪潮的核心。作为PM，你需要深入理解LLM的能力边界和产品化方法。

## 📚 本章节学习目标

- [ ] 深入理解Transformer架构的每个组件
- [ ] 掌握大语言模型的训练流程（预训练→SFT→RLHF）
- [ ] 理解Prompt工程、RAG、Agent等技术
- [ ] 能评估和对比不同LLM产品

## 🎯 核心知识点

### 5.1 Transformer深度解析
- Self-Attention机制详解
- 多头注意力
- 位置编码（绝对 vs 相对）
- Layer Normalization
- 前馈网络
- **PM视角**: 理解为什么Transformer如此强大

### 5.2 大语言模型训练
- 预训练：自回归 vs 掩码语言建模
- 指令微调(SFT)
- RLHF（人类反馈强化学习）
- 缩放定律(Scaling Laws)
- **PM视角**: 理解模型能力如何随规模增长

### 5.3 Prompt工程与优化
- Zero-shot / Few-shot / Chain-of-Thought
- Prompt设计模式
- 结构化输出
- **PM视角**: 优化产品中的Prompt以提升用户体验

### 5.4 RAG与知识增强
- 检索增强生成架构
- 向量数据库与嵌入
- 知识图谱结合
- **PM视角**: 如何让LLM回答更专业、更准确

### 5.5 Agent与工具使用
- ReAct模式
- Function Calling / Tool Use
- 多Agent协作
- **PM视角**: AI Agent的产品化机会

## 🔗 推荐资源

| 资源类型 | 名称 | 链接 | 难度 |
|----------|------|------|------|
| 论文 | Attention Is All You Need | [arXiv](https://arxiv.org/abs/1706.03762) | ⭐⭐⭐ |
| 论文 | Scaling Laws for Neural LMs | [arXiv](https://arxiv.org/abs/2001.08361) | ⭐⭐⭐ |
| 论文 | InstructGPT / RLHF | [arXiv](https://arxiv.org/abs/2203.02155) | ⭐⭐⭐ |
| 课程 | Stanford CS224N | [斯坦福](https://web.stanford.edu/class/cs224n/) | ⭐⭐⭐ |
| 书籍 | 《Build LLM From Scratch》 | [Raschka](https://www.manning.com/books/build-a-large-language-model-from-scratch) | ⭐⭐⭐ |
| 实践 | Prompt Engineering Guide | [指南](https://www.promptingguide.ai/) | ⭐⭐ |
| 实践 | LangChain文档 | [LangChain](https://python.langchain.com/) | ⭐⭐ |

## 📝 学习产出模板

```markdown
## 学习笔记: [LLM相关主题]

**日期**: YYYY-MM-DD

### 核心理解
[用自己的话总结]

### 产品应用场景
- 场景1: [描述]
- 场景2: [描述]

### 竞品分析
| 产品 | 模型 | 特点 | 优缺点 |
|------|------|------|--------|
| ChatGPT | GPT-4 | ... | ... |
| Claude | Claude 3 | ... | ... |

### PM思考
[从产品角度，这个技术的商业化机会在哪里？]
```

## ✅ 检查清单

- [ ] 精读Attention论文并理解每个组件
- [ ] 理解预训练→SFT→RLHF完整流程
- [ ] 掌握Prompt Engineering核心技巧
- [ ] 搭建一个RAG Demo
- [ ] 体验并对比3个以上LLM产品
- [ ] 理解Agent架构和Tool Use模式
