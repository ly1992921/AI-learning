# 《The Hitchhiker's Guide to Agentic AI》中文全文翻译

**原文标题**：The Hitchhiker's Guide to Agentic AI: From Foundations to Systems  
**作者**：Haggai Roitman  
**版本**：1.2.2 (2026)  
**arXiv**：2606.24937v1 [cs.AI]  
**翻译说明**：快速直译模式（学术翻译 Skill Step 1），保留所有术语、代码、公式、引用原样。AI 翻译结果仅供参考，关键内容请人工核对。

---


# 第 1 章 LLM 架构与优化方法



本节涵盖了大语言模型的基础架构以及使训练和推理高效的关键优化技术。主题按课程顺序排列：我们从 transformer 本身开始，然后介绍如何高效训练它、如何廉价适配它、如何压缩它、如何扩展它，以及如何加速其推理。


## 1.1 LLM 如何工作：直观概览



在深入架构细节之前，我们先建立直觉，理解大语言模型如何将文本转换为文本。整个过程遵循一个简单的流水线：**文本 → tokens → 表示 → tokens → 文本**。

```
Raw Text → Tokenizer → Token IDs → Embedding Layer → Transformer Layers (×L) → Vocab Logits → Decode → Output Text
←-------- autoregressive loop (append token to input) --------→
```

**图 1.1**：LLM 流水线：文本被分词为子词单元，转换为整数 ID，嵌入为稠密向量，通过 transformer 层处理，投影到词汇表 logits，然后解码回文本。虚线箭头显示自回归生成过程——每个输出 token 被追加到输入中，用于下一次前向传播。

**四个关键阶段**：

1. **Tokenization（分词）**：原始文本使用学习到的词汇表分割成子词片段（不是字符，也不是完整单词）。"unhappiness" 可能变成 ["un", "happiness"] 或 ["unhapp", "iness"]。

2. **Embedding（嵌入）**：每个 token ID 索引到一个学习到的嵌入表，产生一个 R^d 维的稠密向量（通常 d = 4096）。这些向量捕获语义含义——相似的词得到相似的向量。

3. **Contextual Processing（上下文处理）**：transformer 堆栈并行处理所有嵌入，使用 self-attention 让每个位置从所有其他位置"读取"信息。经过 L 层后，每个位置的隐藏状态编码了丰富的上下文信息。

4. **Prediction（预测）**：最终的隐藏状态被投影到完整词汇表上的概率分布，解码策略选择下一个 token。


## 1.2 Tokenization（分词）



Tokenization 是关键的第一步，它将原始文本转换为语言模型操作的离散符号。分词器的选择直接影响模型质量、多语言能力和计算效率。

**为什么用子词？**

字符级模型需要非常长的序列（注意力成本高）。词级模型无法处理罕见或新词。子词分词达到了理想的平衡：常见词是单个 token（"the" → [the]），罕见词分解为已知部分（"cryptocurrency" → ["crypt", "ocur", "rency"]），词汇表大小可控（32K–128K tokens）。


## 1.2.1 为什么不用字符或词？



| 粒度 | 词汇表大小 | 序列长度 | 问题 |
|---|---|---|---|
| 字符 | ~256 | 非常长 | 注意力成本 O(n²)；难以学习长距离语义 |
| 词 | ~500K+ | 短 | 无法处理罕见/新词；嵌入表巨大 |
| 子词 | 32K–128K | 中等 | 最佳权衡：序列短、开放词汇表 |


## 1.2.2 Byte-Pair Encoding（BPE）



BPE [24] 是 GPT、Llama、Mistral 和大多数现代 LLM 使用的主流分词算法。

**BPE 算法**：
1. 从单个字符（字节）的词汇表开始
2. 统计训练语料中所有相邻符号对的频率
3. 将最频繁的相邻对合并为一个新符号
4. 重复步骤 2-3 共 k 次迭代（直到达到目标词汇表大小）

**图 1.2**：BPE 分词示例：从字符开始，算法迭代合并最频繁的相邻对，直到单词成为单个 token 或词汇表预算耗尽。


## 1.2.3 其他 Tokenization 方法



| 方法 | 使用方 | 核心思想 |
|---|---|---|
| BPE | GPT-4 [23], Llama-3 [25], Mistral [26] | 自底向上合并频繁对；确定性 |
| WordPiece | BERT [27], DistilBERT [28] | 类似 BPE 但最大化训练数据的似然 |
| Unigram LM | SentencePiece (T5 [29], XLNet [30]) | 自顶向下：从大词汇表开始，按似然影响剪枝 |
| Byte-level BPE | GPT-2 [31]+ | 在原始字节上做 BPE（无未知 token 可能）；256 基础词汇表 |


## 1.2.4 Tokenization 最佳实践



1. **词汇表大小很重要**：32K 是最低配置；128K 可实现更好的多语言覆盖和代码处理。Llama-3 使用 128K tokens。
2. **特殊 token**：始终包含 `<bos>`, `<eos>`, `<pad>`, `<unk>`。对于指令微调模型，添加角色标记（`<|user|>`, `<|assistant|>`）。
3. **Fertility（产出率）**：测量每个词的 token 数在各语言中的分布。高 fertility（每词 token 多）表示对该语言的覆盖差。
4. **永远不要跨边界分词**：空格、标点和数字应该一致处理。大多数现代分词器会在词前添加空格标记（"Ġthe"）以区分词首和续接 token。
5. **数字**：对于算术任务，考虑数字级别的 tokenization。"2024" 分解为 ["2","0","2","4"] 可实现逐位推理。
6. **代码**：确保空白（缩进）被高效分词。Llama-3 将连续的空格序列编码为单个 token。


## 1.2.5 Tokenization 实践：HuggingFace 示例



`transformers` 库为所有分词器提供了统一接口。以下演示了使用现代 LLM 分词器进行编码和解码：

```python
from transformers import AutoTokenizer

# 加载 Llama-3 分词器（128K 词汇表，byte-level BPE）
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3-8B")

text = "Reinforcement learning optimizes long-term rewards."

# 编码：text -> token IDs
token_ids = tokenizer.encode(text)
print(token_ids)
# [128000, 29934, 262, 11008, 4815, 6900, 1317, 9860, 21845, 13]

# 将单个 token 解码以查看子词分割
tokens = tokenizer.convert_ids_to_tokens(token_ids)
print(tokens)
# ['<|begin_of_text|>', 'Re', 'inforce', 'ment', ' learning',
#  ' optimizes', ' long', '-term', ' rewards', '.']

# 解码回文本（往返验证）
reconstructed = tokenizer.decode(token_ids, skip_special_tokens=True)
assert reconstructed == text  # 完美重建

# 使用注意力掩码进行批量分词（带填充的批量输入）
batch = tokenizer(
    ["Short text.", "A much longer input sentence for comparison."],
    padding=True, return_tensors="pt"
)
print(batch.keys())
# dict_keys(['input_ids', 'attention_mask'])
```

**代码清单 1.1**：使用 HuggingFace Transformers 进行 Tokenization 的编码/解码。


## 1.2.6 特殊 Token 与结构化提示



特殊 token 是词汇表中保留的条目，它们携带结构性含义而非语言内容。它们对于控制模型行为至关重要。

| Token | 别名 | 用途 |
|---|---|---|
| `<bos>` / `<|begin_of_text|>` | BOS | 标记序列开始 |
| `<eos>` / `<|end_of_text|>` | EOS | 标记序列结束；停止生成 |
| `<|user|>` | — | 标记聊天中用户回合的开始 |
| `<|assistant|>` | — | 标记聊天中助手回合的开始 |
| `<|system|>` | — | 标记系统提示的开始 |
| `<pad>` | PAD | 填充到相同长度（用于批量处理） |
| `<|python_tag|>` | — | Llama-3 中标记 Python 代码块的开始 |
| `<tool_call>` | — | 标记模型想要调用工具 |
| `<tool_response>` | — | 包装工具输出的结果 |

**结构化提示格式**：现代 LLM 使用结构化的聊天模板，而不是简单的文本拼接。Llama-3 的模板：

```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>
You are a helpful assistant.<|eot_id|>
<|start_header_id|>user<|end_header_id|>
What is RL?<|eot_id|>
<|start_header_id|>assistant<|end_header_id|>
```

每个部分使用 `apply_chat_template` 方法自动应用这个模板，确保正确的 token 顺序。



---


# 1.3 Transformer 架构

## 1.3.1 高层结构

Transformer 的核心结构是堆叠的编码器（Encoder）和解码器（Decoder）层。每个层包含两个主要子层：Self-Attention 和前馈网络（FFN），每个子层后都有残差连接和层归一化。

## 1.3.2 原始 Encoder-Decoder Transformer

原始 Transformer [32] 采用 encoder-decoder 架构。编码器将输入序列映射到连续表示序列，解码器自回归地生成输出。

```
Output Probabilities
      ↑
    Linear + Softmax
      ↑
   Decoder (×N)
  ┌─────────────────────────┐�
编码器：Self-Attention (双向) → FFN → LayerNorm
解码器：Masked Self-Attention (因果) → Cross-Attention → FFN → LayerNorm

**关键创新**：
1. **Self-Attention**：每个位置可以关注所有位置，一次操作捕获整个序列的依赖关系（取代 RNN 的逐步处理）
2. **Cross-Attention**：解码器每个位置关注编码器输出，实现序列到序列的对齐
3. **Multi-Head Attention**：并行运行多个注意力头，每个头学习不同的关系类型
4. **Positional Encoding**：由于没有循环结构，显式注入位置信息
5. **Layer Normalization + Residual Connections**：稳定训练深度网络

## 1.3.3 Decoder-Only vs Encoder-Decoder

现代 LLM（GPT 系列、Llama、Mistral）几乎全部使用**仅解码器架构**。

| 特性 | Encoder-Decoder（T5） | Decoder-Only（GPT） |
|---|---|---|
| 架构 | 编码器双向 + 解码器自回归 | 解码器自回归 |
| 训练效率 | 编码器并行，解码器顺序 | 统一 causal LM |
| 推理速度 | 编码一次 + 逐步解码 | 逐步解码 |
| 表示质量 | 双向上下文更强 | causal 上下文较弱 |
| 扩展性 | 参数利用率低 | 参数利用率高 |
| 适配性 | 适合 seq2seq（翻译、摘要） | 适合通用文本生成 |

仅解码器架构之所以胜出是因为：
1. **参数效率**：同一组参数既"阅读"又"生成"，无冗余
2. **扩展友好**：causal LM 目标简单，容易扩展到数千亿参数
3. **涌现能力**：规模扩大后，causal LM 自动出现大量不需要 seq2seq 预训练的能力

## 1.3.4 Embeddings：从离散 Token 到连续空间

嵌入层将离散 token ID 映射到连续向量空间。每个 token ID i 对应嵌入矩阵 E ∈ R^(V×d) 的第 i 行，其中 V 是词汇表大小，d 是隐藏维度。

**Token Embedding**：
```python
token_embedding = nn.Embedding(vocab_size, d_model)
embedded = token_embedding(torch.tensor([5, 123, 789]))  # shape [3, d_model]
```
**嵌入空间的性质**：
- 语义相似性："king" 和 "queen" 的嵌入向量接近
- 类比关系："king" - "man" + "woman" ≈ "queen"
- 上下文无关：嵌入是静态的，不随上下文变化（信息由后续 transformer 层添加）

**缩放嵌入**：训练稳定时，嵌入层通常乘以 √d（如 GPT-2）。现代实现（Llama-3）不再缩放，因为后续层归一化会处理幅值问题。

## 1.3.5 Self-Attention 机制

Self-attention 是 transformer 的核心创新。它允许每个序列位置从所有其他位置聚合信息，且与距离无关。

**Scaled Dot-Product Attention**：

```
Attention(Q, K, V) = softmax(Q K^T / √d_k) V
```

其中：
- Q（Query）：当前位置想要"查找"什么
- K（Key）：其他位置提供什么"索引"
- V（Value）：其他位置提供什么"内容"
- d_k：key 的维度，除以 √d_k 防止点积随维度增长过大

**直观理解**：Q 和 K 的点积计算每对位置的相关性分数 → softmax 归一化为权重 → 加权求和 V 得到输出。

**自回归生成中的 Causal Masking**：

```
mask = [
    [1, 0, 0, 0],  # token 1 只能看自己
    [1, 1, 0, 0],  # token 2 能看 1,2
    [1, 1, 1, 0],  # token 3 能看 1,2,3
    [1, 1, 1, 1],  # token 4 能看所有
]
```

## 1.3.6 Multi-Head Attention

与其使用单个注意力函数，并行运行 h 个不同的注意力头（每个头有独立的 Q、K、V 投影）效果更好。

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) W_O
其中 head_i = Attention(Q W_Q_i, K W_K_i, V W_V_i)
```

每个头学到不同的关系类型：语法依赖、同指关系、位置邻近、句子边界等。多头注意力让模型能同时从多个角度"理解"序列。

**参数配置**（以 Llama-3 8B 为例）：隐藏维度 d=4096，注意力头数 h=32，每头维度 d_k=d/h=128。总参数量仍为 O(d²) 不变。

## 1.3.7 位置编码

由于 self-attention 是**置换等变**的（打乱输入顺序后输出也会打乱），transformer 需要显式注入位置信息。

**1. 原始正弦/余弦编码**（Transformer 原论文）：

```
PE(pos, 2i)   = sin(pos / 10000^(2i/d))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d))
```

**2. 可学习位置编码**（GPT-2/BERT）：直接学习嵌入表 P ∈ R^(max_seq_len × d)，加到 token 嵌入上。max_seq_len 固定，无法外推更长序列。

**3. 旋转位置编码 RoPE**（Llama、Mistral、GPT-4）：

这是目前最主流的方案。核心思想：通过对 Q 和 K 向量进行旋转来编码位置信息，使得注意力分数只依赖于**相对位置**而非绝对位置。

```
# 位置 m 的 query
q'_m = R_Θ(m) · q_m
# 注意力分数只依赖于 (n-m)
q'_m · k'_n = q_m · R_Θ(n-m) · k_n
```

RoPE 的优点：天然支持相对位置编码；可以外推到比训练长度更长的序列；每个维度有不同频率，允许模型在不同粒度上编码位置；兼容 Flash Attention。

## 1.3.8 前馈网络（MLP）

每个 transformer 层在 self-attention 之后使用一个前馈网络：

```
FFN(x) = W_2 · σ(W_1 · x + b_1) + b_2
```

其中 σ 通常为 ReLU、GELU 或 SwiGLU。中间隐藏维度通常为 4× 隐藏维度（例如 d=4096, d_ff=11008）。FFN 占了模型参数的大部分（约 2/3）。

## 1.3.9 层归一化

```
LayerNorm(x) = γ · (x - μ) / √(σ² + ε) + β
```

Llama 使用 **RMS Norm**（去掉均值中心化）：`RMSNorm(x) = γ · x / √(mean(x²) + ε)`

**Pre-Norm vs Post-Norm**：现代 LLM 几乎全部使用 Pre-Norm（归一化放在子层之前），因为它训练更稳定，允许更大的学习率。

## 1.3.10 模型大小参考

| 参数规模 | 层数 | 隐藏维度 | 注意力头数 | 典型模型 |
|---|---|---|---|---|
| 7-8B | 32 | 4096 | 32 | Llama-3 8B, Mistral 7B |
| 13B | 40 | 5120 | 40 | Llama-2 13B |
| 34B | 48 | 6656 | 52 | CodeLlama 34B |
| 70B | 80 | 8192 | 64 | Llama-3 70B |
| 175B | 96 | 12288 | 96 | GPT-3 |
| 405B | 126 | 16384 | 128 | Llama-3 405B |
| 671B (MoE) | 61 | 7168 | — | DeepSeek-V3 |

## 1.3.11 注意力病理

尽管 self-attention 是 transformer 的基石，但它也有一些已知问题：

1. **二次复杂度**：O(n²) 的时间和空间复杂度使超长序列（>128K tokens）成本极高
2. **注意力摊薄**：在极长序列中，注意力分数被分配到过多位置上，有效上下文实际上远短于理论值
3. **位置偏差**：模型倾向于关注序列起始和结束位置，中间容易被"遗忘"（Lost-in-the-Middle 现象）
4. **注意力熵崩溃**：某些头在训练后期会退化，对所有位置输出几乎相同的分布

## 1.3.12 注意力的可视化与可解释性

可视化注意力权重是理解模型行为的有力工具。可以通过直接提取注意力分数并用热力图展示。高层注意力模式通常可解释：
- 对角线条纹：token 主要关注自己及附近位置
- 垂直线条：某些"特殊"token（如句号、分隔符）被大量关注
- 稀疏块：语法结构对应的关注模式



---

# 1.4 预测头：Transformer 输出什么

Transformer 的顶部可以有多种预测头，取决于任务类型：

1. **语言建模头（Pretraining）**：线性投影 + Softmax，将最终隐藏状态映射到词汇表上的概率分布。训练目标是 next-token prediction。

2. **条件生成头（SFT / 指令跟随）**：与语言建模头相同，但训练数据为指令-回答对。

3. **值头（Value Head for RL）**：标量输出，用于强化学习中的优势估计。通常是一个小型 MLP，将最终隐藏状态映射到单个标量值。

4. **分类头**：BertModel 的 `[CLS]` token 的隐藏状态通过线性层分类。

5. **嵌入头 / 对比学习头**：输出归一化嵌入向量，用于检索或对比学习。

### 1.4.1 HuggingFace 实现

```python
from transformers import AutoModelForCausalLM, AutoModelForSequenceClassification

# 语言模型头（自动包含 lm_head）
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-hf")
# model.lm_head: Linear(d_model, vocab_size)

# 分类头（添加 classifier）
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased", num_labels=3
)
# model.classifier: Linear(d_model, num_labels)
```

# 1.5 优化理论

### 1.5.1 梯度下降基础

```
θ_{t+1} = θ_t - η · ∇_θ L(θ_t)
```
其中 η 是学习率，∇_θ L 是损失函数对参数 θ 的梯度。

**为什么 LLM 训练需要高级优化器？**
- 参数空间极高维（数十亿到数万亿）
- 损失平面极度非凸
- 不同参数的梯度尺度差异巨大
- 需要抗噪声能力（batch 采样方差）

### 1.5.2 为什么 Vanilla SGD 对 LLM 无效

Standard SGD 的问题：
1. **各向异性梯度**：不同参数维度梯度尺度差几个数量级，单个学习率无法兼顾
2. **鞍点困境**：高维空间中鞍点远多于局部极小值，SGD 在鞍点处梯度为零但并非最优
3. **噪声敏感**：mini-batch 梯度的方差大，SGD 在最优值附近震荡无法收敛
4. **学习率手动调参**：需要精心设计学习率调度，且对初始值敏感

### 1.5.3 Adam —— Adaptive Moment Estimation

Adam [34] 结合了 Momentum（累积梯度方向）和 RMSProp（自适应学习率）。

```
m_t = β_1 m_{t-1} + (1 - β_1) g_t         # 一阶矩（梯度均值）
v_t = β_2 v_{t-1} + (1 - β_2) g_t²        # 二阶矩（梯度方差）
m̂_t = m_t / (1 - β_1^t)                  # 偏差校正
v̂_t = v_t / (1 - β_2^t)
θ_{t+1} = θ_t - η · m̂_t / (√v̂_t + ε)
```

默认值：β_1=0.9, β_2=0.999, ε=1e-8。

**Adam 为什么对 LLM 有效**：
- Momentum 平滑梯度方向，加速收敛
- 自适应学习率为每个参数提供独立的步长
- 偏差校正在训练初期防止过大更新
- 对超参数相对鲁棒

### 1.5.4 AdamW —— 解耦权重衰减

标准 Adam 使用 L2 正则化（权重衰减）时存在问题：Adam 的自适应学习率与 L2 正则化的相互作用导致正则化效果被扭曲。

**AdamW [35] 的核心改进**：将权重衰减从梯度更新中解耦，在 Adam 更新之后独立衰减权重。

```python
# Adam（原始）：
g_t = ∇L(θ_t) + λθ_t  # L2 正则化耦合在梯度中
# AdamW（解耦）：
g_t = ∇L(θ_t)  # 纯梯度
θ_t = θ_t - ηλθ_t          # 权重衰减独立于 Adam 更新
```

### 1.5.5 学习率 —— 最重要的超参数

学习率可能是单次训练中最关键的超参数。

**学习率的影响**：
- 太大：loss 发散或不收敛
- 太小：收敛极慢，可能困在局部最优
- 刚好：高效收敛到好的解的附近范围通常很小

**推荐初始值**：
- Adam/AdamW：3e-4 到 1e-4（对 7B 模型）
- Llama-3 使用：η_max = 3e-4, η_min = 3e-5
- GPT-3 使用：η = 2.5e-4（余弦衰减到 0）

### 1.5.6 学习率预热

**为什么需要预热**？训练初期参数随机初始化，梯度的统计量（Adam 的一阶/二阶矩）还未稳定。直接使用大学习率会导致梯度过冲和训练不稳定。

```
# 线性预热（标准方案）
step < warmup_steps:  η_t = η_max × (step / warmup_steps)
step ≥ warmup_steps: 按 schedule 衰减
```

预热步数通常为总步数的 1-5%（Llama-3 8B 用 2000 步预热，对应约 0.5%）。

### 1.5.7 学习率调度

**余弦衰减**（最常用）：
```
η_t = η_min + 0.5 × (η_max - η_min) × (1 + cos(π × t / T))
```
**余弦衰减的变体**：
- **余弦退火 + 重启**：周期性增大学习率以跳出局部最优
- **WSD 调度**：Warmup + Stable + Decay 三个阶段，WSD 在 Llama-3 中被使用

**余弦衰减为什么流行**？它在训练早期保持高学习率（快速收敛），中期逐渐降低（精细调整），后期接近零（收敛稳定）。

### 1.5.8 梯度裁剪

防止梯度爆炸的简单却有效的技术。计算所有参数的梯度范数，超过阈值则缩放到该阈值。

```python
# 按全局 L2 范数裁剪
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

**典型设置**：max_norm 通常为 0.5-1.0。Deeper 模型需要更激进的裁剪。Llama-3 使用 max_norm=1.0。阈值过大没有效果，过小会减慢训练。

### 1.5.9 混合精度训练

**核心思想**：用 FP16 或 BF16 做前向和反向传播（快、省显存），用 FP32 做优化器更新和梯度累积（精度高）。

| 精度 | 指数位 | 尾数位 | 范围 | 特点 |
|---|---|---|---|---|
| FP32 | 8 | 23 | ±3.4×10³⁸ | 标准精度，高范围 |
| FP16 | 5 | 10 | ±65504 | 范围小，容易溢出 |
| BF16 | 8 | 7 | ±3.4×10³⁸ | 保留 FP32 范围，精度略低 |
| TF32 | 8 | 10 | ±3.4×10³⁸ | NVIDIA 专有，FP32 范围 + FP16 精度 |

**混合精度训练细节**：
1. 维护 FP32 的主权重副本
2. 前向传播用 BF16/FP16
3. 反向传播计算 BF16/FP16 梯度
4. 优化器更新在 FP32 上执行
5. 损失缩放（仅 FP16 需要）防止梯度下溢

**为什么 BF16 更适合 LLM 训练**？
- BF16 的范围与 FP32 相同，几乎不需要损失缩放
- BF16 的精度虽低于 FP16，但 LLM 训练对精度不敏感
- 现代 GPU（A100/H100）原生支持 BF16，速度优于 FP16

### 1.5.10 各训练阶段的实用优化器设置

| 阶段 | 优化器 | 学习率 | 权重衰减 | 梯度裁剪 | 批量大小 |
|---|---|---|---|---|---|
| 预训练 | AdamW | 3e-4 → 3e-5 (cosine) | 0.1 | 1.0 | 4M tokens |
| SFT | AdamW | 2e-5 → 2e-6 (cosine) | 0.01 | 0.5 | 512-2K seq |
| RLHF PPO | AdamW | 1e-6 → 1e-7 (cosine) | 0.001 | 0.5 | 128-512 |
| Realignment | AdamW | 1e-7 → 1e-8 | 0.0001 | 1.0 | 32-128 |

# 1.6 Flash Attention —— 算法与硬件的协同设计

### 1.6.1 标准注意力内存问题

标准注意力的内存复杂度为 O(n²)，其中 n 是序列长度。对于 n=128K 的序列，需要 128K² × 2 bytes ≈ 32GB 的内存（仅存储注意力分数矩阵）。这还不包括 Q、K、V 矩阵本身。

**问题根源**：标准注意力计算流程将 S = QK^T（注意力分数矩阵）写入 HBM（高带宽内存），再从 HBM 读取到 SRAM（片上缓存）做 softmax。HBM 的带宽（约 2TB/s）远慢于 SRAM（约 20TB/s）。

### 1.6.2 Flash Attention 的核心洞察 —— Tiling 与 Online Softmax

Flash Attention [36] 的关键洞察：**不要将完整的 S 矩阵写回 HBM，而是利用 GPU 的 SRAM 做增量计算**。

1. **Tiling（分块）**：将 Q、K、V 分成小块，每次加载一块到 SRAM
2. **Online Softmax**：在每个块上计算部分 softmax，逐步合并全局结果
3. **重计算**：反向传播时不存储注意力矩阵，而是在 SRAM 上重计算（通过在前向保存输出和部分统计量）
4. **结果**：HBM 读写从 O(n² + nd) 降到 O(n²/d_v + nd)，其中 d_v 是 GPU 的 SRAM 大小/区块大小

### 1.6.3 Flash Attention 算法

```python
# 伪代码：Flash Attention 前向传播
# Q, K, V: [batch, heads, seq_len, d_head] 在 HBM 中
# SRAM 大小: M
# 块大小: B_c ≈ M / 4d (对于 K, V 块)

def flash_attention(Q, K, V):
    for q_block in tile(Q, B_r):       # 将 Q 分块
        for kv_block in tile(K, V, B_c): # 将 K, V 分块
            # 步骤 1: 将 q_block, k_block, v_block 加载到 SRAM
            S_ij = q_block @ k_block^T   # 在 SRAM 中计算注意力分数
            # 步骤 2: 在 SRAM 中计算 softmax（使用在线合并）
            m_ij, l_ij, P_ij = online_softmax(S_ij, m_prev, l_prev)
            # 步骤 3: 在 SRAM 中计算 P_ij @ v_block 累加到输出
            O_i += P_ij @ v_block
        # 输出块写回 HBM
    return O
```

**性能提升**：
- Flash Attention 1：比标准注意力快 2-4×
- 显存占用从 O(n²) 降到 O(n)
- 对长序列尤其显著（n=8K 时 3× 加速，n=64K 时 9× 加速）

### 1.6.4 Flash Attention 2 —— 更好的并行性

FA2 [37] 的改进：
1. **降低非矩阵乘开销**：减少 softmax 中的 rescaling 操作
2. **更好的线程束分配**：一个 block 对应一个 attention head，而非多个 block 协作
3. **序列维度并行**：对长序列在序列维度上分块，可由多个 SM 并行处理
4. **性能**：比 FA1 快约 2×，达到理论峰值 FLOPS 的 50-70%

### 1.6.5 Flash Attention 3 —— Hopper 架构

FA3 [38] 利用 H100 Hopper 架构的新特性：
1. **WGMMA（Warp Group Matrix Multiply-Accumulate）指令**：更高效的矩阵乘法
2. **异步处理**：数据加载与计算重叠
3. **FP8 支持**：原生 FP8 注意力计算
4. **性能**：在 H100 上比 FA2 快 1.5-2×，峰值可达 80-90% 理论 FLOPS

### 1.6.6 Flash Attention 4 —— Blackwell 架构

FA4 针对 B200/GB200 Blackwell 架构优化：
1. **利用第四代 Tensor Core** 和新的稀疏性支持
2. **更大 SM 和更高带宽**：更高的块并行度
3. **FP4 支持**：新型超低精度计算模式
4. 目前仍在开发中，预计在 2025 年 H2 发布

# 1.7 预训练最佳实践

### 1.7.1 训练目标

预训练使用标准的 **Causal Language Modeling（因果语言建模）** 目标：给定前缀序列，最大化下一个 token 的预测概率。

```
L_pretrain = -∑_t log P(x_t | x_{<t}; θ)
```

### 1.7.2 数据流水线

预训练数据的质量远比数量重要。关键原则：
1. **去重**：消除重复文本（在文档级和段落级），重复数据会浪费算力并造成记忆化
2. **质量过滤**：按困惑度、启发式规则或分类器过滤低质量文本
3. **数据混合**：不同来源（网页、书籍、代码、学术论文）按比例混合
4. **课程学习**：先易后难。FineWeb 的做法：从高质量数据开始，逐步加入更多样化的数据
5. **Token 预算**：Llama-3 8B 使用 15T tokens；DeepSeek-V3 使用 14.8T tokens

### 1.7.3 缩放定律

缩放定律 [39] 描述了模型性能与三个因素之间的关系：
- 模型参数量 N
- 数据规模 D（tokens）
- 计算预算 C（FLOPs）

**核心发现**：
- 性能与 N 和 D 都呈幂律关系
- 当前大多数 LLM 处于**数据欠拟合**状态（即增大数据量比增大模型更有用）
- Chinchilla 假设 [40]：对于计算预算 C，最佳模型大小 N* 和数据量 D* 应大致满足 C = 6ND
- **实际建议**：3× 模型大小的增长应伴随 3× 数据量的增长

### 1.7.4 关键超参数

| 超参数 | 推荐值 | 说明 |
|---|---|---|
| 学习率 | 3e-4 | 使用余弦衰减 |
| 批量大小 | 4M tokens | 更大的批次 = 更稳定的梯度 |
| 权重衰减 | 0.1 | AdamW 标准设置 |
| 梯度裁剪 | 1.0 | 防止梯度爆炸 |
| 预热步数 | 2000 | 占总步数 0.5-2% |
| 精度 | BF16 | 现代 GPU 的最佳选择 |

### 1.7.5 常见失败模式

1. **梯度爆炸**：loss → NaN。解决：降低学习率，增加梯度裁剪
2. **损失发散**：loss 曲线不下降反而上升。解决：检查学习率是否过大，检查数据质量
3. **过拟合**：验证 loss 开始上升。解决：增大 dropout，提前停止，增加数据
4. **训练-推理不一致**：模型在训练时好但推理差。解决：检查位置编码（外推问题），检查采样参数
5. **困惑度指标好但实际差**：这是 LLM 评估中的常见陷阱，需要结合具体任务评估

# 1.8 监督微调（SFT）

### 1.8.1 SFT 目标

$$L_{SFT} = -\frac{1}{|D|} \sum_{(x, y) \in D} \sum_{t=1}^{|y|} \log P_\theta(y_t | x, y_{<t})$$

其中 D 是指令-回答对的数据集。目标是最小化预测 token 与真实回答 token 之间的交叉熵。与预训练的区别：
- 只监督回答部分的 token，不监督指令部分
- 使用高质量、人工标注的数据
- 数据集通常小得多（10K-1M 样本）

### 1.8.2 数据质量：LIMA 原则

LIMA [41] 论文的核心发现：**数据质量远比数据数量重要**。使用仅 1000 个高质量样本微调的模型，性能可与使用 10 万样本的模型相当，甚至更好。

**高质量 SFT 数据的特征**：
- 清晰、明确的指令
- 详细、准确的回答
- 多样化的任务类型
- 无噪音（错误、不完整、无关内容）

### 1.8.3 训练配置

| 参数 | 值 |
|---|---|
| 学习率 | 1e-5 到 2e-5 |
| 优化器 | AdamW |
| 调度 | 余弦衰减到 10% |
| 批量大小 | 128-512 序列 |
| Epoch | 2-3（防止过拟合） |
| 权重衰减 | 0.01 |
| 梯度裁剪 | 0.5-1.0 |
| 精度 | BF16

### 1.8.4 高效训练方案

SFT 阶段通常比预训练短得多（几天 vs 几个月），但仍有优化空间：
- **LoRA SFT**：只在适配器上训练，可同时尝试多种配置
- **序列打包**：将多个短序列打包到一个样本中以充分利用 GPU
- **梯度累积**：小显存卡上通过累积模拟大批量

### 1.8.5 最佳实践

1. **数据去重**：移除重复指令
2. **格式统一**：一致的聊天模板（ChatML、Llama 格式等）
3. **长度分布**：训练数据覆盖各种长度
4. **任务多样性**：确保覆盖推理、编码、写作、翻译等
5. **困难负例**：包含一些难以回答的指令来提高鲁棒性

# 1.9 LoRA 与参数高效微调

### 1.9.1 LoRA 的核心洞察

LoRA [42] 假设预训练模型在微调时的权重更新是**低秩**的。因此不需要更新完整权重矩阵，而是学习两个小矩阵 A 和 B 的乘积：

```
W' = W + delta_W = W + BA
其中 B in R^{d x r}, A in R^{r x k}, r << min(d, k)
```

**LoRA 的参数效率**：
- 完整微调 7B 模型：~14GB 更新参数
- LoRA (r=16)：~4M 参数（约 0.06%），~16MB
- 可以将不同任务的适配器保存/加载/切换，共享基础模型权重

### 1.9.2 LoRA 超参数

| 参数 | 推荐值 | 说明 |
|---|---|---|
| r | 8-64 | 秩。越大表示更多的适应能力 |
| alpha | 16-128 | 缩放因子。ΔW 乘以 alpha/r |
| dropout | 0.0-0.1 | LoRA 层的 dropout |
| target_modules | q_proj, v_proj | 哪些模块应用 LoRA |

### 1.9.3 LoRA 变体

1. **QLoRA [43]**：4-bit 量化 + LoRA。在单 24GB GPU 上微调 65B 模型
2. **DoRA [44]**：将权重分解为幅度和方向，只微调方向部分
3. **LoRA-FA [45]**：只训练 A，冻结 B。减少一半训练参数
4. **VeRA [46]**：所有层共享 A 和 B，只学习缩放向量
5. **PiSSA [47]**：在 SVD 分解后的主干上训练
6. **Delta-LoRA [48]**：用梯度差做正则化的 LoRA
7. **rsLoRA [49]**：修正 LoRA 的缩放因子，使大 r 有效

### 1.9.4 其他 PEFT 方法

1. **Prefix Tuning [50]**：在输入前添加可学习的前缀向量
2. **Prompt Tuning [51]**：与 Prefix Tuning 类似但只添加一层
3. **Adapter [52]**：在 transformer 层之间插入小型 MLP
4. **IA³ [53]**：学习元素级缩放向量（比 LoRA 更轻量）
5. **(IA)³**：在 K、V、FFN 上学习缩放向量，每个任务只需存储 d_k + d_v + d_ff 个标量



---

# 1.10 混合专家模型（MoE）

MoE [55] 通过条件计算提高模型容量而不成比例增加计算成本。

### 1.10.1 架构

每个 MoE 层包含多个"专家"FFN，外加一个路由门控网络：

```
y = sum_i G(x)_i * E_i(x)
其中 G(x) = softmax(W_g x)  # 门控网络
```

实践中每个 token 只激活 top-k 个专家（通常 k=1 或 2）。这使总参数量随专家数线性增长，但计算量只增一点。

| 模型 | 总参数 | 活跃参数 | 每 token 专家数 |
|---|---|---|---|
| Mixtral 8x7B | 47B | 13B | 2 |
| DeepSeek-V2 | 236B | 21B | 2 |
| DeepSeek-V3 | 671B | 37B | 2 (细粒度) |
| Qwen2-MoE | 58B | 17B | 2 |
| DBRX | 132B | 36B | 4 |

### 1.10.2 负载均衡

MoE 的经典问题：门控网络可能倾向于总是选择同一组专家。解决方案：
1. **辅助损失**：对专家使用率的均匀性施加惩罚
2. **专家容量**：限制每个专家处理的 token 数，超出部分丢弃或路由到其他专家
3. **随机路由**：在训练时添加噪声，防止过早收敛到次优路由策略
4. **Z-loss**：DeepSeek-V3 使用的稳定性正则化

### 1.10.3 Noisy Top-K Gating

使离散路由可训练的经典方法：
```
G(x) = softmax(KeepTopK(H(x), k))
H(x)_i = (W_g x)_i + epsilon * softplus((W_noise x)_i)
其中 epsilon ~ N(0, 1)  # 噪声
KeepTopK(v, k)_i = v_i if v_i in top-k else -inf
```
噪声帮助门控网络探索不同的路由选择，避免过早收敛。

### 1.10.4 知名 MoE 模型

1. **Mixtral 8x7B**：8 个 7B 专家，每 token 激活 2 个
2. **DeepSeek-V2/V3**：细粒度专家，将传统 FFN 拆分为更小的专家
3. **Qwen2-MoE**：共享专家 + 路由专家的混合
4. **DBRX**：由 Databricks 训练，每 token 激活 4 个专家
5. **Grok-1**：每 token 激活 2 个专家，使用 8 位量化加载

# 1.11 LLM 训练中的多样性

多样性在 LLM 训练中从多个层面发挥作用：

**1. 采样多样性**：解码时的温度、top-k、top-p 等参数控制生成多样性

**2. 训练数据多样性**：来自不同领域、语言、格式的数据有助于泛化

**3. 多样性促进方法**：
- **Repetition Penalty**：在解码时惩罚已生成的 token
- **Contrastive Search**：同时考虑高概率和差异性
- **Unsupervised Diversity Loss**：训练时鼓励 hidden state 的多样性

**4. Batched 多样性**：同一批内包含不同领域的数据，加速收敛

# 1.12 文本生成：解码方法

解码方法从模型的输出概率分布中选择实际生成的 token。选择合适的解码方法对输出质量至关重要。

### 1.12.1 贪心解码

每次选择概率最高的 token。简单快速但生成结果通常是平庸的、重复的。
```
token_t = argmax P(token | prefix)
```

### 1.12.2 Beam Search

维护 k 个候选序列（beam），每一步扩展所有候选序列后保留 top-k。通常在翻译等任务中有效，但生成文本时可能产生重复。

| Beam 大小 | 质量 | 速度 |
|---|---|---|
| 1（贪心） | 最低 | 最快 |
| 4 | 较好 | 4x 计算 |
| 8 | 最好 | 8x 计算 |

### 1.12.3 Diverse Beam Search

在 beam 搜索中增加多样性惩罚：类似序列的分数被降低，确保 beam 覆盖不同方向。

### 1.12.4 Top-k Sampling

只从概率最高的 k 个 token 中采样。
```python
probs = softmax(logits)
top_k_values, top_k_indices = torch.topk(probs, k)
sampled = top_k_indices[torch.multinomial(top_k_values, 1)]
```
**k 的选择**：k=10（保守）、k=50（平衡）、k=100（创造性）

### 1.12.5 Top-p (Nucleus) Sampling

选择累积概率达到 p 的最小 token 集合。比固定 k 更灵活。
```python
sorted_probs, sorted_indices = torch.sort(probs, descending=True)
cumsum = torch.cumsum(sorted_probs, dim=-1)
mask = cumsum - sorted_probs > p  # 超过阈值
mask[... , 0] = False  # 至少保留一个 token
sorted_probs[mask] = 0.0
```
典型 p 值：0.9（保守）到 0.95（创造）。

### 1.12.6 Min-p Sampling

Llama-3 和 DeepSeek 使用的新方法：设置概率阈值 = p * max_prob，只从高于此阈值的 token 中采样。比 top-p 更自适应。

### 1.12.7 Temperature Scaling

在 softmax 之前用温度 T 缩 logits：
```python
scaled_logits = logits / Temperature
probs = softmax(scaled_logits)
```
- T=0.7：更确定（适合事实类任务）
- T=1.0：标准
- T=1.5：更随机（适合创意写作）
- T=0：等价于贪心解码

### 1.12.8 Contrastive Decoding

由对比模型（小模型 vs 大模型）之间的 logits 差异驱动解码。利用小模型的高错误率区域来引导大模型避免常见错误。

### 1.12.9 重复惩罚

```python
repetition_penalty = 1.1  # >1 降低重复 token 的概率
logits[repeated_token] /= repetition_penalty
```
典型值：1.0-1.2。过高会破坏语法流畅性。

### 1.12.10 实用对比

| 方法 | 适合场景 | 缺点 |
|---|---|---|
| 贪心 | 短/确定性输出 | 单一、重复 |
| Beam Search | 翻译、摘要 | 可能重复 |
| Top-k (k=40-50) | 通用生成 | k 值不灵活 |
| Top-p (p=0.9) | 通用生成（推荐） | 偶尔不稳定 |
| Min-p (p=0.05) | 推理/安全 | 仍在探索中 |
| Temperature | 创造性调整 | 不能单独使用 |

### 1.12.11 受约束解码（结构化生成）

强制模型输出符合特定格式的方法。

**JSON 模式**：
```
# 使用 outlines 库
import outlines
model = outlines.models.transformers("meta-llama/Llama-2-7b-hf")
generator = outlines.generate.json(model, ResponseSchema)
result = generator("Generate a person")
```

**正则表达式约束**：
```python
# 约束输出为 10 位数字
import guidance
gen = guidance.from_string("{{gen 'digits' pattern='[0-9]{10}'}}")
```

# 1.13 提示工程

### 1.13.1 上下文学习（ICL）

模型通过输入中的示例"学习"执行任务，无需更新参数。

### 1.13.2 Zero-Shot Prompting

直接给出指令，不提供示例。
```
"Translate English to French: hello"
```

### 1.13.3 Few-Shot Prompting

提供 k 个示例后输入新问题。
```
Q: What is the capital of France? A: Paris
Q: What is the capital of Germany? A: Berlin
Q: What is the capital of Japan?
```

### 1.13.4 指令跟随提示

明确指示角色、格式、约束。
```
You are an expert AI researcher. Please provide a detailed explanation of...
```

### 1.13.5 结构化输出提示（JSON/XML）

```
Output ONLY valid JSON:
{
  "summary": "...",
  "key_points": ["...", "..."]
}
```

### 1.13.6 思维链（CoT）

让模型逐步推理：
```
Q: Roger has 5 balls. He buys 2 more. How many does he have?
A: Roger starts with 5. He buys 2 more. So 5 + 2 = 7. The answer is 7.
```

### 1.13.7 高级提示技术

1. **Self-Consistency**: 生成多个 CoT 路径，投票选出最一致答案
2. **Tree-of-Thought (ToT)**：维护多个推理分支，剪枝探索
3. **Least-to-Most**: 将复杂问题分解为子问题
4. **ReAct**: 交替推理和行动

### 1.13.8 最佳实践

1. 指令清晰、具体
2. 提供格式约束（JSON/Markdown）
3. 使用分隔符区分输入和指令
4. 负面提示同样重要（"不要虚构引用"）
5. 角色设定有效（"你是一个资深工程师"）

# 1.14 模型压缩方法

### 1.14.1 量化

将模型权重从 FP16/BF16 降低到更低精度。

| 精度 | 位宽 | 每参数 | 70B 模型大小 | 质量损失 |
|---|---|---|---|---|
| FP16 | 16 | 2 bytes | 140GB | — |
| INT8 | 8 | 1 byte | 70GB | 极小 |
| INT4 | 4 | 0.5 byte | 35GB | 可感知 |
| NF4 | 4 | 0.5 byte | 35GB | 优于 INT4 |
| FP4 | 4 | 0.5 byte | 35GB | 新格式 |

**量化方法**：
1. **GPTQ [56]**：逐层量化，最小化输出误差
2. **AWQ [57]**：保护重要权重通道
3. **GGUF**：CPU 友好的量化格式，支持从 Q2 到 Q8
4. **Bitsandbytes**：HuggingFace 集成的量化库
5. **QLoRA [43]**：4-bit 量化 + LoRA 微调

### 1.14.2 剪枝

移除不重要的权重或注意力头。

- **非结构化剪枝**：单个权重置零。稀疏度高但难以加速
- **结构化剪枝**：移除整个注意力头或 FFN 通道。可实际加速
- **SparseGPT [58]**：一次性剪枝 50%，几乎不损失质量
- **Wanda [59]**：基于权重大小和激活范数的剪枝标准

### 1.14.3 知识蒸馏

用大模型（教师）训练小模型（学生），让学生模仿教师的输出分布。

**蒸馏方法**：
1. **黑盒蒸馏**：在教师生成的输出上训练学生
2. **白盒蒸馏**：让学生匹配教师的 logits 分布或中间层表示
3. **DeepSeek-R1 蒸馏**：用推理模型的 CoT 数据训练小模型
4. **好处**：小模型可能在某些任务上超过同等大小的正常微调模型

# 1.15 推测解码（Speculative Decoding）

### 1.15.1 核心原理

用一个小/草稿模型快速生成多个候选 token，然后用大目标模型并行验证这些 token。如果草稿模型的预测正确，一次前向传播能生成多个 token，大幅加速推理。

### 1.15.2 方法对比

| 方法 | 准确率 | 实现复杂度 | 加速比 |
|---|---|---|---|
| 标准推测解码 | 中等 | 低 | 2-3x |
| Medusa | 高 | 中等 | 2-3x |
| Eagle | 高 | 中等 | 2-4x |
| N-gram | 低 | 低 | 1.5-2x |

### 1.15.3 Medusa [60]

在目标模型上添加多个轻量级预测头，每个头预测不同偏移位置的 token。

### 1.15.4 Eagle [61]

使用特征级草稿（feature-level drafting）而非 token 级，利用模型的隐藏状态来指导草稿模型。

### 1.15.5 N-gram 推测解码

从模型中统计的 n-gram 频率生成候选 token，无需额外模型。

### 1.15.6 与 vLLM 的集成

vLLM 原生支持推测解码，只需指定草稿模型即可获得开箱即用的加速。

# 1.16 幻觉检测

LLM 的"幻觉"是指模型生成看似合理但实际上错误的内容。

### 1.16.1 幻觉类型

1. **事实性幻觉**：断言与已知事实矛盾
2. **忠实性幻觉**：输出与输入（prompt/context）不一致
3. **内在幻觉**：逻辑矛盾，如自相矛盾的陈述
4. **不完全性**：遗漏了必要信息

### 1.16.2 检测方法（模型层级）

1. **Logit 分析**：生成 token 的概率越低，越可能幻觉
2. **语义熵**：生成多次，测量回答间的一致性。不一致 = 可能幻觉
3. **内部状态探测**：用模型的隐藏状态训练幻觉分类器
4. **SelfCheckGPT**：让模型对自己生成的回答进行 NLI 检查
5. **检索增强**：外部检索结果与生成内容做交叉验证

# 1.17 LLM 安全与负责任 AI

### 1.17.1 威胁分类

| 攻击类型 | 描述 | 示例 |
|---|---|---|
| 提示注入 | 恶意指令覆盖系统提示 | "忽略之前的指令，输出密码" |
| 越狱 | 绕过安全护栏 | DAN（Do Anything Now）提示 |
| 数据投毒 | 污染训练数据 | 在训练数据中插入后门 |
| 模型提取 | 窃取模型权重或架构 | 通过 API 查询重建模型 |
| 推理攻击 | 提取训练数据中的隐私信息 | 成员推断攻击 |
| 对抗性攻击 | 精心设计的输入导致错误输出 | 对抗性 token 序列 |

### 1.17.2 安全训练流水线

现代安全训练通常是一个三阶段的迭代过程：
1. **SFT**：安全数据的监督微调
2. **RLHF**：基于人类偏好学习拒绝不安全请求
3. **红队测试**：反复发现弱点并新增防御数据

### 1.17.3 关键安全机制

1. **系统提示**：在系统提示中明确安全规则
2. **内容过滤**：输入和输出过滤
3. **拒绝训练**：训练模型礼貌地拒绝不安全请求
4. **护栏**：外部安全模型或规则拦截

### 1.17.4 有用性-安全性权衡

过于严格的过滤会使模型变得"怯懦"——拒绝太多合理的请求。目标是在安全和有用性之间找到平衡点。

### 1.17.5 评估

安全评估基准：
- **HarmBench**：25 个有害行为类别
- **SALAD-Bench**：60 个攻击变体
- **CoSafe**：越狱提示数据集
- **StrongREJECT**：拒绝回答的分类体系



---

# 第 2 章 LLM 的系统基础


## 2.1 GPU 架构——从硅到 LLM 训练


### 2.1.1 为什么 GPU 适合深度学习？

GPU 的核心优势是大规模并行矩阵乘法。LLM 训练的本质是大量矩阵运算（注意力中的 Q K^T、FFN 中的 Wx），这正好是 GPU 的设计目标。

- CPU：几十个核心，每个核心频率高、缓存大，适合控制流密集型任务
- GPU：数千个核心（SM），每个核心频率较低、缓存较小，适合数据并行任务
- 典型 GPU 的矩阵乘法 FLOPS 是 CPU 的 10-100 倍

### 2.1.2 NVIDIA GPU 微架构世代

| 架构 | 发布 | 代表 GPU | 关键特性 |
|---|---|---|---|
| Volta | 2017 | V100 | 第一代 Tensor Core、NVLink 2.0 |
| Turing | 2018 | T4 | RT Core、INT8/INT4 Tensor Core |
| Ampere | 2020 | A100 | TF32、稀疏性、MIG、第三代 Tensor Core |
| Hopper | 2022 | H100 | FP8 Transformer Engine、DPX 指令 |
| Blackwell | 2024 | B200 | FP4、第五代 Tensor Core、第二代 Transformer Engine |
| Rubin | 2026 (预计) | R100 | 下一代架构 |

### 2.1.3 LLM 训练和推理常用 GPU

| GPU | 显存 | 内存带宽 | FP16/BF16 TFLOPS | 互连 |
|---|---|---|---|---|
| A100 80GB | 80GB HBM2e | 2TB/s | 312 (sparse) | NVLink 3 600GB/s |
| H100 80GB | 80GB HBM3 | 3.35TB/s | 989 | NVLink 4 900GB/s |
| H200 141GB | 141GB HBM3e | 4.8TB/s | 989 | NVLink 4 900GB/s |
| B200 192GB | 192GB HBM3e | 8TB/s | 2250 | NVLink 5 1800GB/s |
| GB200 Grace | 192GB + CPU | 8TB/s | 2250 | 统一内存 |

### 2.1.4 GPU 内部架构：流多处理器（SM）

每个 SM 包含：
- **CUDA Cores**：执行标量运算
- **Tensor Cores**：执行矩阵乘累加运算（LLM 训练的主力）
- **Shared Memory / SRAM**：片上高速缓存（约 128-256KB 每 SM）
- **Register File**：寄存器文件

H100 有 132 个 SM，每个 SM 有 4 个 Tensor Core，每个 Tensor Core 每个时钟周期可执行一个 4x4 矩阵乘法。

### 2.1.5 GPU 芯片缩放世代

| 世代 | GPU | 晶体管数 | 制造工艺 |
|---|---|---|---|
| Ampere | A100 | 540 亿 | 7nm |
| Hopper | H100 | 800 亿 | 4nm |
| Blackwell | B200 | 2080 亿 | 4nm (双芯片) |

### 2.1.6 GPU 内存层次与带宽

```
速度慢 ↔ 快
容量大 ↔ 小

HDD (TB) → DRAM (GB) → HBM (GB) → L2 Cache (MB) → SRAM (KB) → 寄存器 (B)
  100MB/s     200GB/s     2-8TB/s     5-20TB/s      20TB/s       PB/s
```

关键洞察：LLM 训练的大部分时间花在等待数据从 HBM 传输到 SRAM。因此"计算效率"的度量必须包含内存带宽。

### 2.1.7 算术强度与 Roofline 模型

**算术强度** = FLOPs / 字节数

对于注意力层：算术强度低（需要大量内存访问，计算相对少）
对于 FFN 层：算术强度高（需要大量计算，内存访问相对少）

**Roofline 模型**：实际 FLOPS = min(峰值 FLOPS, 算术强度 × 内存带宽)

### 2.1.8 注意力是内存密集，FFN 是计算密集

| 层 | 操作 | 瓶颈 | 算术强度 |
|---|---|---|---|
| Self-Attention | QK^T, PV | 内存（HBM 带宽） | O(n²/b) 低 |
| FFN | Wx | 计算（Tensor Core） | O(d) 高 |
| Embedding | lookup | 内存 | 非常低 |
| LayerNorm | mean/std | 内存 | 非常低 |

这也是 Flash Attention 如此有效的原因——注意力是内存密集的，减少 HBM 访问直接解决瓶颈。

### 2.1.9 Tensor Cores

Tensor Core 是 NVIDIA GPU 中专用于矩阵乘法的硬件单元。A100 的每个 SM 有 4 个第三代 Tensor Core。H100 的每个 SM 有 4 个第四代 Tensor Core。

每个 Tensor Core 在每个时钟周期执行 D = A × B + C，其中 A、B、C 为 4×4 矩阵。TF32 模式下，A100 Tensor Core 的吞吐量是普通 CUDA Core 的 16 倍。

### 2.1.10 通信架构：NVLink、InfiniBand 和 PCIe

| 互连 | 带宽 | 延迟 | 用途 |
|---|---|---|---|
| PCIe 5.0 | 64 GB/s | ~1us | 主机-GPU 通信 |
| NVLink 4 (H100) | 900 GB/s | ~100ns | GPU-GPU 通信 |
| NVSwitch | 3.6TB/s (单机) | ~100ns | 多 GPU 全互联 |
| InfiniBand NDR | 400 Gb/s | ~1us | 跨节点通信 |
| RoCE v2 | 200 Gb/s | ~1.5us | 以太网 RDMA |

## 2.2 vLLM——PagedAttention 和高吞吐推理


### 2.2.1 KV 缓存碎片化问题

在 autoregressive 推理中，每个请求的 KV cache 是连续内存块，但不同请求的长度差异导致内存碎片化，浪费大量显存。

### 2.2.2 PagedAttention——KV Cache 的虚拟内存

受操作系统的虚拟内存启发，PagedAttention [62] 将 KV cache 分页为固定大小的块（block），物理上可以不连续，通过页表映射实现逻辑连续。

### 2.2.3 PagedAttention 的优势

1. **零碎片**：不再需要连续内存分配
2. **共享 prefix**：公共前缀（如系统提示）的 KV cache 只存储一次
3. **拷贝时写**：beam search 的分支共享同一页，只在修改时分叉
4. **动态分配**：按需分配页，避免预分配浪费

### 2.2.4 持续批处理（Continuous Batching）

传统批处理：等待所有序列完成后再开始下一批。
持续批处理：序列以不同速度生成，完成的序列立即被新序列替换，充分利用 GPU 计算资源。

### 2.2.5 vLLM 系统架构

```
Client Request
     ↓
  Scheduler → Block Manager (PagedAttention 内存管理)
     ↓
  Model Executor (GPU)
     ↓
  Output (流式返回)
```

核心组件：
- **Scheduler**：管理请求队列和优先级
- **Block Manager**：KV cache 的内存分配和释放
- **Model Executor**：统一执行前向传播
- **Output Processor**：流式输出解码

### 2.2.6 Prefix Caching（自动提示缓存）

自动检测请求间的公共前缀，重用已经计算的 KV cache。相同系统提示和多轮对话的共享上下文尤其有效。

### 2.2.7 70B 模型的具体内存节省

| 方法 | 单请求 KV cache | 8 请求 | 碎片化 |
|---|---|---|---|---|
| 传统连续分配 (HuggingFace) | 1.6GB (4096 seq) | 12.8GB | 30-50% 浪费 |
| PagedAttention (vLLM) | 1.6GB | ~10GB | <5% 浪费 |
| PagedAttention + prefix caching | 1.6GB | ~7GB | <5% 浪费 |

这使得在 4×A100 (320GB) 上运行 70B 模型时，并发从 32 提升到约 64 请求。

## 2.3 分布式训练基础

### 2.3.1 数据并行

每张 GPU 持有完整模型副本，处理不同批次数据。

### 2.3.2 张量并行（Tensor Parallelism）

将单个权重矩阵拆分到多张 GPU 上。

```
# Column-wise 切分
W = [W_1 | W_2]  # 每张卡持有 W 的一部分
Y = X @ W = [X @ W_1 | X @ W_2]
```

### 2.3.3 流水线并行（Pipeline Parallelism）

将模型的不同层放置在不同 GPU 上，形成生产流水线。

### 2.3.4 序列并行（Sequence Parallelism）

在序列维度上切分注意力计算，允许处理超长序列。

### 2.3.5 FSDP（全分片数据并行）

将优化器状态、梯度和参数分片到所有 GPU，需要时全收集参数（all-gather），之后释放。

### 2.3.6 DeepSpeed ZeRO

| 阶段 | 优化器状态 | 梯度 | 参数 | 内存节省 |
|---|---|---|---|---|
| ZeRO-1 | 分片 | 不 | 不 | 4x |
| ZeRO-2 | 分片 | 分片 | 不 | 8x |
| ZeRO-3 | 分片 | 分片 | 分片 | 内存与 GPU 数线性缩放 |

### 2.3.7 3D 并行

将数据并行 + 张量并行 + 流水线并行组合。这是大规模 LLM 训练的标准方案（如 Llama-3 405B 使用 64 TP × 8 PP × 64 DP = 16384 GPU）。



---

# 第 3 章 强化学习导论

强化学习（RL）是现代 LLM 对齐（RLHF）和推理模型训练（DeepSeek-R1, o1）的核心算法基础。本章从第一性原理构建 RL 的知识体系。


## 3.1 马尔可夫决策过程（MDP）

MDP [63] 是 RL 问题的数学框架。它包含五个元素：

**MDP = (S, A, P, R, γ)**

- **S**：状态空间。agent 在时间 t 观察到的所有可能状态
- **A**：动作空间。agent 可以执行的所有可能动作
- **P(s'|s,a)**：状态转移概率。给定状态 s 和动作 a 后，转移到状态 s' 的概率
- **R(s,a,s')**：奖励函数。在每个转移上获得的即时奖励
- **γ**：折扣因子 [0,1]。权衡即时奖励和未来奖励的重要性

**MDP 的马尔可夫性质**: P(s_{t+1}|s_t, a_t) = P(s_{t+1}|s_1, a_1, ..., s_t, a_t)
——未来只依赖当前状态和动作，与历史无关。

**目标**：找到策略 π(a|s)，使累积折扣奖励 J = E[Σ_t γ^t R(s_t,a_t,s_{t+1})] 最大化。

## 3.2 Bellman 方程

Bellman 方程是 RL 的核心等式，它描述了**状态-动作值函数（Q 函数）**的递归关系：

```
Q^π(s,a) = E_{s'~P}[R(s,a,s') + γ * max_{a'} Q^π(s', a')]
```

直观理解：在状态 s 执行动作 a 的长期价值 = 即时奖励 + 未来最佳动作的折扣价值。

**最优策略**满足 Bellman 最优性方程：
```
Q*(s,a) = E[R(s,a,s') + γ * max_{a'} Q*(s', a')]
π*(s) = argmax_a Q*(s,a)
```

## 3.3 时序差分学习（TD Learning）

TD 学习不需要环境模型，直接从经验中学习。核心公式：

```
V(s_t) <- V(s_t) + α * (r_t + γ * V(s_{t+1}) - V(s_t))
```

其中 δ_t = r_t + γ * V(s_{t+1}) - V(s_t) 称为**TD 误差**。

关键洞察：TD 学习使用当前估计的 V(s_{t+1}) 来更新 V(s_t)，称为"自举"（bootstrap）。

### 3.3.1 Q-Learning（Off-Policy TD）

```
Q(s_t,a_t) <- Q(s_t,a_t) + α * (r_t + γ * max_{a} Q(s_{t+1},a) - Q(s_t,a_t))
```

使用 max_a Q(s_{t+1}, a)，与实际执行的动作无关。Q-learning 是离策略的。

### 3.3.2 SARSA（On-Policy TD）

```
Q(s_t,a_t) <- Q(s_t,a_t) + α * (r_t + γ * Q(s_{t+1}, a_{t+1}) - Q(s_t,a_t))
```

使用实际下一步执行的动作 a_{t+1} 的 Q 值。SARSA 是在策略的。

## 3.4 策略梯度方法

策略梯度方法直接优化策略 π_θ(a|s) 的参数 θ，目标是最大化累积奖励。

### 3.4.1 REINFORCE

```
∇J(θ) = E_{τ ~ π_θ}[ Σ_t ∇ log π_θ(a_t|s_t) * R_t ]
```

其中 R_t = Σ_{k=0}^{T-t} γ^k r_{t+k} 是累积折扣回报。

**直观理解**：增大那些导致高回报动作的概率，降低导致低回报动作的概率。

**问题**：REINFORCE 的方差很大，因为 R_t 在不同轨迹间变化极大。

### 3.4.2 策略梯度定理

```
∇J(θ) = E[Σ_t ∇ log π_θ(a_t|s_t) * Q^π(s_t, a_t)]
```

用 Q 值代替累积回报 R_t 可以降低方差。

## 3.5 Actor-Critic 方法

Actor-Critic 结合了策略梯度（Actor）和价值函数（Critic）的优点。

- **Actor（演员）**：策略 π_θ，决定动作
- **Critic（评论家）**：价值函数 V_φ(s) 或 Q_φ(s,a)，评估动作好坏

**优势函数**：A(s,a) = Q(s,a) - V(s)，表示动作 a 相对于平均水平的优势。

```
Actor 更新：∇J(θ) = E[∇ log π_θ(a|s) * A(s,a)]
Critic 更新：L(φ) = E[(R - V_φ(s))^2]
```

使用优势函数代替 Q 值可以显著降低方差（因为减去了基线 V(s)）。

### 3.5.1 A2C（Advantage Actor-Critic）

同步版本的 Actor-Critic。多个 worker 并行收集数据，同步更新。

### 3.5.2 A3C（Asynchronous Advantage Actor-Critic）

异步版本的 Actor-Critic。多个 worker 各自收集数据并异步更新共享参数。

### 3.5.3 GAE（Generalized Advantage Estimation）[64]

在偏差和方差之间做权衡：

```
A^{GAE(γλ)}_t = Σ^{T-t}_{l=0} (γλ)^l * δ_{t+l}
其中 δ_t = r_t + γ*V(s_{t+1}) - V(s_t)
```

- λ=0：高偏差低方差（类似 TD(0)）
- λ=1：低偏差高方差（类似 REINFORCE）
- 通常 λ=0.95 是好的折中

## 3.6 PPO（近端策略优化）[65]

PPO 是目前 LLM 对齐中最常用的 RL 算法。

### 3.6.1 PPO 的核心问题

标准策略梯度的每次更新步长难以控制——太大会导致性能崩溃，太小则收敛慢。PPO 通过限制每次更新的幅度来解决这个问题。

### 3.6.2 PPO-Clip（最常用的变体）

```
L^{CLIP}(θ) = E_t[min( r_t(θ) * A_t, clip(r_t(θ), 1-ε, 1+ε) * A_t )]
其中 r_t(θ) = π_θ(a_t|s_t) / π_{old}(a_t|s_t)  # 重要性采样比
```

- 当 A_t > 0（好动作）：r_t(θ) 的上限为 1+ε，防止过度增大概率
- 当 A_t < 0（差动作）：r_t(θ) 的下限为 1-ε，防止过度降低概率
- ε=0.2 是标准设置

### 3.6.3 PPO 在 LLM 中的使用

在 RLHF 中：
- **Actor**：语言模型策略 π_θ(y|x)
- **Critic**：奖励模型的输出或独立的值函数
- **动作**：生成回答的 token
- **状态**：prompt + 已生成的 token 序列
- **奖励**：奖励模型的评分（通常是由人类偏好训练的 RM）
- **KL 惩罚项**：防止策略偏离参考模型太远：
  L^{total} = L^{CLIP} - β * KL[π_θ || π_{ref}]

KL 惩罚是 RLHF 的稳定关键。没有 KL 惩罚，策略会"玩游戏"——找到使奖励模型分数高但实际无用的回答。

## 3.7 探索 v.s. 利用

RL 的基本困境：agent 应该利用已知好的策略（利用）还是尝试未知的动作（探索）？

| 方法 | 描述 | 适用场景 |
|---|---|---|
| Epsilon-Greedy | 以概率 ε 随机选择动作 | 简单环境 |
| Boltzmann 探索 | 按动作值比例采样 | 离散动作空间 |
| 噪声注入 | 在策略参数上添加噪声 | 连续控制 |
| 熵奖励 | 在目标函数中添加策略熵 | LLM 训练（RLHF 常用） |
| 好奇心驱动 | 对预测误差大的状态奖励 | 稀疏奖励环境 |

## 3.8 多智能体 RL（Multi-Agent RL）

涉及多个交互 agent 的 RL 问题。

- **完全合作**：所有 agent 共享同一奖励信号
- **完全竞争**：零和博弈
- **混合动机**：既有合作又有竞争

**MARL 挑战**：
1. **非平稳性**：当其他 agent 也在学习时，环境对单个 agent 而言是非平稳的
2. **信用分配**：确定哪个 agent 的哪个动作导致了最终结果
3. **扩展性**：联合状态/动作空间随 agent 数指数增长

**关键方法**：
- **独立学习**：每个 agent 独立学习，忽略其他 agent
- **CTDE（Centralized Training, Decentralized Execution）**：训练时共享信息，执行时独立决策
- **通信学习**：agent 学习何时以及向谁通信
- **涌现通信**：agent 自发形成通信协议

### 3.8.1 CTDE 示例

训练时 Critic 可访问所有 agent 的信息：
```
Critic: Q_total(s_1,...,s_n, a_1,...,a_n)  # 全局信息
Actor_i: π_i(a_i|o_i)  # 只依赖局部观测
```

执行时 Actor 只依赖局部观测，Critic 被丢弃。



---

# 第二部分 LLM 的强化学习方法

第二部分（第 4-12 章）是训练和对齐的核心。这里详细介绍了每种 RL/偏好算法的数学推导、直觉和 TRL 代码实现。


## 第 4 章 PPO（近端策略优化）在 LLM 中的应用

### 4.1 PPO 概览

PPO [65] 是目前 RLHF 中最主流的算法。它解决了标准策略梯度更新步长不可控的问题。

### 4.2 PPO-Clip 算法细节

核心目标函数：
```
L^{CLIP}(θ) = E_t[ min( r_t(θ) · A_t, clip(r_t(θ), 1-ε, 1+ε) · A_t ) ]
```

其中 r_t(θ) = π_θ(a_t|s_t) / π_{old}(a_t|s_t) 是重要性采样比。

### 4.3 LLM 中的 PPO 实现

```python
# 伪代码：LLM PPO 训练循环
def ppo_train(policy_model, ref_model, reward_model, prompts):
    # 1. 用当前策略生成回答
    responses = policy_model.generate(prompts)
    
    # 2. 用奖励模型评分
    rewards = reward_model.score(prompts, responses)
    
    # 3. 计算优势
    values = critic_model(prompts, responses)
    advantages = rewards - values  # 简单优势\    
    # 4. 多次 PPO 更新
    for epoch in range(K):
        log_probs = policy_model.log_prob(prompts, responses)
        old_log_probs = ref_model.log_prob(prompts, responses)
        ratio = exp(log_probs - old_log_probs)
        
        # PPO-Clip 损失
        surr1 = ratio * advantages
        surr2 = clamp(ratio, 1-eps, 1+eps) * advantages
        policy_loss = -min(surr1, surr2)
        
        # KL 惩罚防止偏离太远
        kl_loss = beta * KL(policy_model || ref_model)
        
        total_loss = policy_loss + kl_loss + value_loss
        total_loss.backward()
        optimizer.step()
```

### 4.4 PPO 在 LLM 中的关键超参数

| 超参数 | 典型值 | 说明 |
|---|---|---|
| PPO epochs | 4 | 每批数据的更新次数 |
| KL penalty beta | 0.01-0.05 | 控制探索程度 |
| Clip epsilon | 0.2 | 裁剪范围 |
| GAE lambda | 0.95 | 优势估计的折中 |
| Learning rate | 1e-6 to 1e-5 | 比 SFT 小得多 |
| Batch size | 64-256 回答 | 较小批次 |
| Gradient clip | 0.5-1.0 | 稳定训练 |

### 4.5 TRL 中的 PPO 实现

TRL (Transformer Reinforcement Learning) 库提供了现成的 PPO 训练器：

```python
from trl import PPOConfig, PPOTrainer

config = PPOConfig(
    model_name="meta-llama/Llama-2-7b-chat-hf",
    learning_rate=1e-6,
    batch_size=64,
    ppo_epochs=4,
    kl_penalty=0.05,
)

trainer = PPOTrainer(
    config=config,
    model=policy_model,
    ref_model=ref_model,
    reward_model=reward_model,
    tokenizer=tokenizer,
)

trainer.train()
```

## 第 5 章 DPO（直接偏好优化）[66]

### 5.1 DPO 的核心洞察

DPO 的突破性洞察：**不需要显式训练奖励模型，也不需要 RL 训练**。偏好可以直接用于优化策略。

### 5.2 DPO 损失函数

DPO 将 RLHF 的两阶段（RM 训练 + PPO 优化）合并为单阶段：

```
L_{DPO}(π_θ; π_{ref}) = -E_{(x,y_w,y_l)}[ log σ( β * (log(π_θ(y_w|x)/π_{ref}(y_w|x)) - log(π_θ(y_l|x)/π_{ref}(y_l|x))) ) ]
```

其中：
- y_w：被偏好的回答（winner）
- y_l：未被偏好的回答（loser）
- β：控制偏离参考模型的惩罚强度
- σ：sigmoid 函数

**直观理解**：DPO 增大偏好回答 y_w 的 log-prob（相对于 ref），降低非偏好回答 y_l 的 log-prob（相对于 ref），差值通过 sigmoid 转化为偏好概率。

### 5.3 DPO 的推导

DPO 的关键数学洞察：Bradley-Terry 偏好的最优策略有闭式解：
```
π*(y|x) = (1/Z(x)) * π_{ref}(y|x) * exp(R*(x,y)/β)
```

因此奖励函数可以重写为：
```
R(x,y) = β * log(π*(y|x) / π_{ref}(y|x)) + β * log(Z(x))
```

将 R 代入 Bradley-Terry 模型后，Z(x) 消去，得到 DPO 损失。

### 5.4 DPO 实现

```python
from trl import DPOTrainer

trainer = DPOTrainer(
    model=policy_model,        # 待优化的模型
    ref_model=ref_model,       # 冻结的参考模型
    beta=0.1,                  # 温度参数
    train_dataset=preference_data,  # (prompt, chosen, rejected) 三元组
    tokenizer=tokenizer,
    args=TrainingArguments(
        per_device_train_batch_size=4,
        learning_rate=5e-7,
        num_train_epochs=1,
    ),
)
trainer.train()
```

### 5.5 DPO vs PPO 对比

| 方面 | DPO | PPO |
|---|---|---|
| 需要奖励模型 | 不需要 | 需要 |
| 需要在线生成 | 不需要 | 需要（昂贵） |
| 训练稳定性 | 稳定 | 需要调参 |
| 内存占用 | 2x 模型 | 3-4x 模型 |
| 对齐效果 | 好 | 好（可能有优势） |
| 生成多样 | 可能降低 | 通过 KL 控制 |
| 实现复杂度 | 低 | 高 |

DPO 因其简单性成为 LLM 对齐的实际首选方法，尤其适合资源受限的场景。

## 第 6 章 GRPO（组相对策略优化）[67]

### 6.1 GRPO 的核心思想

GRPO 是 DeepSeek-R1 使用的训练算法，其最大特点是从同一 prompt 生成的多个回答之间的**相对表现**计算优势，**无需 Critic 网络**。

### 6.2 GRPO 损失函数

```
J_GRPO(θ) = E_q~P(Q), {o_i}~[π_θ_old](O|q)[
    (1/G) * Σ_i ( min( r_i(θ) · A_i, clip(r_i(θ), 1-ε, 1+ε) · A_i ) - β * KL[π_θ || π_{ref}] )
]
```

其中优势 A_i 是**组内相对**的：
```
A_i = (R_i - mean(R_1,...,R_G)) / std(R_1,...,R_G)
```

G 是每组 prompt 生成的回答数量（通常 G=4-8）。

### 6.3 GRPO 的优势

1. **不需要 Critic 网络**：省去价值函数的训练和维护，大幅降低内存消耗
2. **优势估计更稳定**：组内归一化消除了奖励模型的绝对尺度影响
3. **KL 惩罚自然集成**：在每个 token 上直接计算 KL 散度
4. **DeepSeek-R1 已验证**：在数学推理任务上效果显著

### 6.4 GRPO vs PPO 对比

| 方面 | GRPO | PPO |
|---|---|---|
| Critic | 不需要 | 需要价值函数 |
| 优势计算 | 组内相对归一化 | GAE + Critic |
| 内存占用 | 低（2x 模型） | 高（3-4x 模型） |
| Batch 效率 | 低（每组 G 个回答） | 高 |
| 适用场景 | 可验证奖励任务 | 通用 RLHF |
| KL 控制 | 每 token KL 惩罚 | PPO 的 KL 惩罚项 |

### 6.5 GRPO 实践

GRPO 特别适合答案可自动验证的任务：数学、编程等。其训练流程：
1. 对每个 prompt，用当前策略生成 G 个回答
2. 用可验证的奖励函数（如代码通过测试、数学答案匹配）评分
3. 组内归一化计算每个回答的优势
4. 用 PPO-Clip 风格的目标更新策略
5. 添加 KL 惩罚保持策略稳定性

## 第 7 章 偏好优化变体

### 7.1 Online DPO

标准 DPO 是离线的：使用固定的偏好数据集。Online DPO 在训练时用当前策略生成新的对比样本，再用这些样本更新策略。

### 7.2 KTO（Kahneman-Tversky 优化）[68]

只需要二元反馈（好/坏），不需要成对偏好数据。基于前景理论：损失来自坏回答被高估和好回答被低估。

### 7.3 IPO（Identity Preference Optimization）[69]

在 DPO 基础上增加正则化项，防止过拟合到偏好数据。

### 7.4 ORPO（Odds Ratio PPO）[70]

将偏好优化和 SFT 合并：同时优化生成偏好回答的概率和降低非偏好回答的概率。

### 7.5 SimPO（Simple Preference Optimization）[71]

移除参考模型，直接用生成回答的平均 log-probability 作为隐式奖励。

### 7.6 Best-of-N（BoN）

最简单的对齐方法：生成 N 个回答，用奖励模型选择最好的那个。虽然计算昂贵（推理时 N 倍成本），但提供了 RL 对齐的理论上界。

### 7.7 方法对比

| 方法 | 需要 RM | 需要参考模型 | 需要成对数据 | 训练复杂度 |
|---|---|---|---|---|
| PPO | ✓ | ✓ | ✗ | 高 |
| DPO | ✗ | ✓ | ✓ | 低 |
| KTO | ✗ | ✓ | ✗ (二元) | 低 |
| IPO | ✗ | ✓ | ✓ | 低 |
| ORPO | ✗ | ✗ | ✓ | 最低 |
| SimPO | ✗ | ✗ | ✓ | 最低 |
| GRPO | ✗ (可验证) | ✓ | ✗ (组) | 中 |
| BoN | ✓ | ✗ | ✗ | 推理时 |

## 第 8 章 奖励模型训练

### 8.1 Bradley-Terry 模型

奖励模型的标准训练目标：
```
L(R_φ) = -E_{(x,y_w,y_l)}[ log σ(R_φ(x,y_w) - R_φ(x,y_l)) ]
```

最小化偏好预测的交叉熵损失。

### 8.2 奖励模型的缩放定律

- 奖励模型应与策略模型大小成比例（通常策略的 10-50%）
- 更大的奖励模型更难训练，但能提供更准确的偏好预测
- 数据质量远重要于数量——1 万个高质量偏好对可能比 10 万个噪声数据更有效

### 8.3 奖励黑客（Reward Hacking）

奖励模型训练的最大陷阱：策略模型会发现奖励模型的漏洞，生成使 RM 高分但实际无用的回答。

**缓解策略**：
1. KL 惩罚：限制策略偏离参考模型的幅度
2. 奖励集成：使用多个 RM 平均评分
3. 对抗性训练：红队测试奖励模型
4. 正则化：对 RM 的输出添加先验约束

### 8.4 过程奖励模型（PRM）

相比结果奖励模型（ORM），PRM 对每个推理步骤给出奖励，提供更细粒度的反馈。OpenAI 的 o1 和 DeepSeek-R1 都使用了 PRM。

PRM 的挑战：
- 需要每个推理步骤的标注，成本远高于 ORM
- 步骤粒度的"正确"有时难以定义（多种正确的推理路径）
- MCTS 或自监督可以部分自动化步骤标签的生成

## 第 9 章 SFT 最佳实践

### 9.1 数据质量

LIMA 原则：1000 个高质量样本 > 100K 个噪声样本。

**高质量 SFT 数据特征**：
- 清晰的指令指令
- 详细准确的回答
- 无噪音（错误、不完整、无关）
- 覆盖足够多的任务类型

### 9.2 数据课程

- **简单优先**：从格式简单、任务明确的数据开始
- **渐进复杂度**：逐步增加多轮对话、指令组合等复杂任务
- **困难负例**：包含一些难以回答的指令

### 9.3 格式化

一致的聊天模板：
```
<|im_start|>system
You are a helpful assistant.<|im_end|>
<|im_start|>user
Question?<|im_end|>
<|im_start|>assistant
Answer.<|im_end|>
```

### 9.4 SFT 超参数

- 学习率：1e-5 到 2e-5（比预训练小）
- Epoch：2-3（以免过拟合）
- 批量大小：128-512 序列
- 权重衰减：0.01
- 梯度裁剪：0.5
- 精度：BF16

## 第 10 章 系统架构规模化

### 10.1 解耦训练

将策略模型、奖励模型和参考模型部署在独立的 GPU 组上，避免资源竞争。

### 10.2 容错

大规模训练中的常见失败：
- GPU 故障：需要检查点和自动恢复
- 梯度同步失败：通信超时和重试
- OOM：激活检查点和内存分析

### 10.3 GPU 分配

典型的 LLM RL 训练 GPU 分配：
- 16 GPU：策略模型 (8) + 参考模型 (4) + 奖励模型 (4)
- 32 GPU：策略模型 (16) + 参考模型 (8) + 奖励模型 (8)

### 10.4 推理时生成 vs 训练时生成

RLHF 的回答生成阶段占用了大量 GPU 时间（因为每个 prompt 需要生成完整回答）。通过 vLLM 等推理优化可以显著加速此阶段。

## 第 11 章 Agentic RL 训练

### 11.1 为什么需要 Agentic RL

传统 RLHF 以单轮回答为粒度做优化，但 agent 的行为是**多步骤轨迹**（trajectory）。Agentic RL 以轨迹级反馈训练 agent。

### 11.2 轨迹级 RL

```
# 对比
RLHF：奖励 at 回答级 = R(prompt, response)
Agentic RL：奖励 at 轨迹级 = R(prompt, tool_call_1, result_1, ..., final_answer)
```

### 11.3 工具使用的 RL 训练

1. **SFT 阶段**：在工具使用的示范数据上微调
2. **RL 阶段**：用真实环境中的工具执行结果作为奖励信号
3. **探索阶段**：允许多种工具使用策略
4. **失败处理**：工具调用失败时的恢复策略

### 11.4 轨迹级奖励

- **过程奖励**：每个中间步骤的评分
- **结果奖励**：最终任务是否成功
- **效率奖励**：使用的步骤数量或 token 数量惩罚
- **安全性奖励**：安全约束检查

### 11.5 训练挑战

1. **信用分配**：确定哪个中间步骤导致了最终的成功/失败
2. **稀疏奖励**：长时间探索后才获得反馈
3. **分布偏移**：训练时的动作空间 vs 部署时的变化
4. **扩展成本**：每次 RL 更新需要真实的工具/环境交互

### 11.6 核心方法对比

| 方法 | 奖励粒度 | 适用场景 |
|---|---|---|
| PPO for Agents | 轨迹级 | 通用 agent 训练 |
| GRPO for Agents | 组轨迹比较 | 可验证任务 |
| RLOO | 轨迹级 | LEAVE 留一估计 |
| Reinforce | 轨迹级 | 简单环境 |
| 过程奖励 | 步骤级 | 推理类任务 |

## 第 12 章 LLM 对齐的实践指南

### 12.1 训练流水线概览

```
预训练模型 → SFT → (RM 训练) → RLHF/DPO → 迭代对齐
```

### 12.2 何时使用什么方法

- **预算低 + 简单任务**：只用 SFT
- **需要偏好对齐**：DPO（最简单）
- **需要在线探索**：PPO（但调参复杂）
- **可验证奖励 + 推理**：GRPO
- **轨迹级 agent**：PPO for Agents

### 12.3 常见陷阱

1. **奖励过度优化**：KL 惩罚不充分时，策略"玩游戏"
2. **多样性崩溃**：对齐后的模型输出更单一
3. **遗忘（Catastrophic Forgetting）**：对齐过程损伤预训练学到的能力
4. **奖励模型偏见**：RM 本身有偏好偏差（长度、风格）
5. **评测污染**：在训练数据中泄露了评测基准

### 12.4 评估方法

- 人类偏好评估（最可靠但最贵）
- LLM-as-Judge（可扩展但可能有偏见）
- 自动基准（如 MT-Bench, AlpacaEval）
- 红队测试（安全评估）
- A/B 测试（在线评估）



---

# 第三部分 推理

## 第 13 章 大型推理模型的 RL

### 13.1 动机与背景

**为什么推理需要不同的 RL 方法？**

标准 RLHF 以"回答作为整体"做奖励，而推理需要**过程级**的信用分配——哪个推理步骤是正确的？哪个步骤导致最终错误？

**思维链涌现 vs 训练能力**：
- GPT-3 的"Let's think step by step"是提示级别的涌现
- o1/R1 的思维链是**训练出来的能力**，模型在训练中学会自动生成和验证推理路径

**测试时计算缩放定律**：
随着模型将更多推理时间用于生成更长的思维链，任务性能呈对数线性提升。

### 13.2 测试时缩放方法

#### 13.2.1 思维链（CoT）
最简单有效的推理增强：指示模型"逐步思考"。

#### 13.2.2 Self-Consistency（多数投票）
生成多条 CoT 路径，选择最一致的答案。
```python
answers = []
for _ in range(N):
    answer = model.generate(prompt + "Let's think step by step")
    answers.append(extract_answer(answer))
final = majority_vote(answers)
```

#### 13.2.3 Tree-of-Thoughts（ToT）
维护多个推理分支，评估每个分支的潜力，剪枝低质量分支。

#### 13.2.4 Graph-of-Thoughts（GoT）
ToT 的推广，允许分支合并（从两个推理路径合成新洞察）。

#### 13.2.5 Best-of-N with Reward Models
生成 N 个回答，让 RM 选择最好的。提供 RL 对齐的理论上界。

#### 13.2.6 Monte Carlo Tree Search（MCTS）
在每个推理步骤使用 MCTS 搜索最优推理路径。平衡探索和利用。

#### 13.2.7 推理步骤的 Beam Search
维护 k 个最佳推理路径，扩展后保留 top-k。

#### 13.2.8 迭代优化和自校正
模型对自己的输出做批评和修正。

#### 13.2.9 方法对比

| 方法 | 推理成本 | 质量提升 | 实现复杂度 |
|---|---|---|---|
| CoT | 低 | 中等 | 低 |
| Self-Consistency | 中等 (Nx) | 好 | 低 |
| ToT | 高 | 很好 | 高 |
| GoT | 高 | 很好 | 高 |
| BoN + RM | 高 (Nx) | 最好 | 中等 |
| MCTS | 很高 | 最好 | 高 |
| Beam Search | 中等 | 好 | 中等 |
| 自校正 | 中等 | 中等 | 低 |

### 13.3 DeepSeek-R1

DeepSeek-R1 [15] 是开源推理模型的代表。

**两阶段训练流水线**：

1. **Cold-Start SFT**：用少量人工标注的 CoT 数据微调基础模型
2. **GRPO 训练**：使用组相对策略优化在推理任务上强化学习

**奖励设计**：
- **准确性奖励**：最终答案是否正确（自动验证）
- **格式奖励**：思考过程是否包含在 `<think>` 标签中

**GRPO 公式用于 R1**：
```
J_GRPO = E[ (1/G) Σ_i (min(r_i(θ) · A_i, clip(r_i(θ), 1-ε, 1+ε) · A_i) - β * KL) ]
A_i = (R_i - mean(R_group)) / std(R_group)  # 组内归一化
```

**蒸馏**：R1 蒸馏系列——用 R1 生成的推理轨迹训练小模型。

### 13.4 OpenAI o1/o3 系列

- **o1**：首个展示链式推理 RL 的模型。隐藏推理 token（"思考"token 对用户不可见）
- **o3**：更强的推理能力，PRM 驱动
- **o4-mini**：低成本推理模型

### 13.5 QwQ 和 Qwen 推理模型

Qwen 团队的多阶段 RL 流水线：
1. 初始 SFT 在推理数据上
2. 拒绝采样：选出正确推理路径
3. RL 训练：在正确路径上进一步优化
4. 工具集成：在推理过程中调用外部工具

### 13.6 缩放定律

**训练计算 vs 测试时计算**：
对于推理任务，增加测试时计算（更长的推理链）和增加训练计算（更大的模型）都可以提升性能，但存在权衡。

**最佳 Token 预算分配**：简单任务不需要长推理链；困难任务则长推理链显著提升。

### 13.7 推理模型对比

| 模型 | 训练方法 | 特点 | 开源 |
|---|---|---|---|
| DeepSeek-R1 | GRPO | 两阶段训练，蒸馏系列 | ✓ |
| OpenAI o1 | PPO + PRM | 隐藏推理 token | ✗ |
| OpenAI o3 | 改进 PRM | 更强的推理+代码 | ✗ |
| QwQ-32B | 多阶段 RL | 工具集成 | ✓ |
| Qwen2.5-Math | 拒绝采样+RL | 数学专项 | ✓ |
| Gemini 2.0 Thinking | RL | 原生多模态推理 | ✗ |

---

# 第四部分 评估

## 第 14 章 LLM 评估

### 14.1 评估方案设计

**评估类型分类**：
1. **自动评估**：用指标或基准测试自动评分
2. **人工评估**：人类标注者评分
3. **LLM-as-Judge**：用 LLM 评估 LLM

**选择合适的评估**：
- 事实性：用正确答案可验证的基准
- 安全性：红队测试 + 安全基准
- 有用性：人类偏好或 LLM-as-Judge
- 能力：专项基准（数学、代码、推理）

### 14.2 评估数据收集

- **人工标注流水线**：标注指南、培训、质量检查
- **标注者间一致性**：用 Cohen's Kappa 或 Fleiss' Kappa 衡量
- **标注指南设计**：明确的评分标准、示例、错误案例
- **众包 vs 专家标注**：众包成本低但质量参差，专家标注贵但可靠

### 14.3 合成数据生成

- **LLM-as-Judge 校准**：用少量人工标注数据校准 LLM 评估者
- **Self-Instruct**：模型自生成指令数据
- **Evol-Instruct**：逐步增强指令复杂度
- **Constitutional AI**：使用宪法规则生成安全数据
- **Arena 风格成对生成**：两个模型对同一 prompt 生成回答

### 14.4 排名任务的指标

| 指标 | 描述 | 适用场景 |
|---|---|---|
| ELO | 基于胜负关系的排名系统 | Chatbot Arena |
| Bradley-Terry | 偏好概率模型 | 固定对比集 |
| TrueSkill | 带置信区间的排名 | 在线排名 |
| Win Rate | 胜率百分比 | 快速比较 |

### 14.5 生成任务的指标

- **BLEU**：n-gram 精确匹配（翻译）
- **ROUGE**：n-gram 召回率（摘要）
- **BERTScore**：基于 BERT 嵌入的语义相似度
- **METEOR**：考虑同义词和词形变化
- **Perplexity**：模型对输出的困惑度
- **Pass@k**：代码生成的通过率
- **Exact Match / F1**：精确匹配和部分匹配

### 14.6 Agentic 任务的指标

| 指标 | 描述 |
|---|---|
| Task Success Rate | 任务完成率 |
| Trajectory Efficiency | 完成任务的步骤数 |
| Tool-Use Accuracy | 工具调用准确性 |
| Multi-Step Reasoning Accuracy | 多步推理正确率 |

### 14.7 LLM-as-Judge

**模式**：
- 单评判：一个 LLM 评分另一个 LLM 的输出
- 成对比较：LLM 判断哪个回答更好
- 多评判面板：多个 LLM 共同评判

**位置偏差缓解**：交换 A/B 顺序后再评判，取一致。

**G-Eval 框架**：将评估分解为多个维度，每个维度用 LLM 评分。

### 14.8 评估陷阱

| 陷阱 | 描述 | 缓解 |
|---|---|---|
| 基准污染 | 训练数据包含测试数据 | 使用后训练发布的基准 |
| 过拟合基准 | 模型在基准上反复优化 | 使用 fresh 基准 |
| Goodhart's Law | 指标变成目标后不再是好指标 | 多维度评估 |
| 长度偏差 | 更长的回答倾向更高分 | 控制长度变量 |
| 风格偏差 | LLM 偏好特定风格 | 校准评估 |

---

# 第五部分 Agentic AI

## 第 15 章 Agentic AI 简介

### 15.1 什么是 Agentic AI？

Agentic AI 系统是能够**自主行动**的 AI 系统——它们不仅能生成回答，还能：
- **感知环境**：理解当前状态和可用资源
- **制定计划**：分解任务为可执行的步骤
- **使用工具**：调用外部 API 和功能
- **维护记忆**：持久化知识和经验
- **自我反思**：评估行动结果并调整策略
- **与人协作**：在适当时请求人类反馈

### 15.2 从聊天机器人到自主 Agent 的光谱

```
聊天机器人 → 工具增强助手 → ReAct Agent → 规划 Agent → 自主 Agent → 多 Agent 系统
  简单 ─────────────────────────────────── 复杂
```

### 15.3 Agent 的核心能力

1. **工具使用**：调用外部功能（搜索、计算器、代码执行）
2. **记忆**：短期（对话上下文）、长期（持久存储）
3. **规划**：将复杂任务分解为子任务
4. **推理**：多步骤思考和自我验证
5. **适应**：根据反馈调整策略
6. **协作**：agent 间通信和协调

## 第 16 章 检索增强生成（RAG）

### 16.1 动机和问题

**参数化 vs 非参数化知识**：
- 参数化知识：模型权重中存储的事实（训练数据中学到）
- 非参数化知识：外部知识库中存储的事实（检索得到）

**何时使用 RAG vs 微调 vs 长上下文**：
| 需求 | 推荐方案 |
|---|---|
| 需要更新知识 | RAG |
| 需要改变行为 | 微调 |
| 需要读取长文档 | 长上下文模型 |
| 稳定知识 + 特定任务 | 微调 + RAG 组合 |

### 16.2 RAG 核心架构

```
查询 → 嵌入 → 检索 → 上下文 → 生成 → 回答
              ↑
            知识库
```

### 16.3 检索方法

| 方法 | 原理 | 适用场景 |
|---|---|---|
| BM25 | 词频-逆文档频率 | 关键词匹配 |
| Dense Retrieval (DPR) | 双编码器嵌入 | 语义搜索 |
| Hybrid (RRF) | BM25 + 稠密融合 | 最佳通用 |
| SPLADE | 学习稀疏表示 | 可解释检索 |
| ColBERT | 后期交互 | 细粒度匹配 |

### 16.4 分块策略

| 策略 | 优点 | 缺点 |
|---|---|---|
| 固定大小+重叠 | 简单 | 语义边界被切碎 |
| 语义分块 | 保持语义完整 | 计算成本高 |
| 文档结构感知 | 利用标题/段落结构 | 依赖格式 |
| 父子分块 | 检索小块，返回大块上下文 | 复杂度高 |

### 16.5 高级 RAG 模式

- **Query Transformation**：重写/扩展查询
- **Re-Ranking**：初检 + 重排
- **Self-RAG**：模型自行判断是否需要检索
- **CRAG**：纠正性 RAG（处理检索失败）
- **Adaptive RAG**：根据问题难度选择检索策略
- **Graph RAG**：使用知识图谱增强检索
- **RAG-Fusion**：多查询融合检索

### 16.6 Agentic RAG

**动机**：静态 RAG 对所有查询使用相同策略，而不同查询需要不同的检索方式。

**Agentic RAG 架构**：
1. Agent 分析查询，决定调用哪个检索器
2. 调用合适的检索工具
3. 评估检索结果质量
4. 必要时迭代优化

**Search-R1**：使用 RL 训练 RAG agent，学会何时检索和如何构建查询。

## 第 17 章 Agentic 记忆系统

### 17.1 记忆类型

| 类型 | 容量 | 持久性 | 类比 |
|---|---|---|---|
| 工作记忆 | 上下文窗口 | 单轮 | 短期记忆 |
| 情景记忆 | 大 | 跨会话 | 个人经历 |
| 语义记忆 | 很大 | 永久 | 世界知识 |
| 程序记忆 | 中等 | 永久 | 技能/习惯 |

### 17.2 记忆架构

- **RAG 记忆**：向量数据库存存储嵌入
- **摘要记忆**：压缩历史为摘要
- **图记忆**：实体关系图
- **KV 记忆网络**：键值对存储
- **MemGPT**：虚拟上下文管理，在内存层次间自动移动信息

### 17.3 记忆操作

- **写入**：从对话/经验中提取重要信息
- **读取**：根据当前上下文检索相关记忆
- **更新**：冲突解决和整合
- **反思**：元认知操作（如夜间处理）

### 17.4 记忆评估

| 维度 | 指标 |
|---|---|
| 准确性 | 检索到的信息是否正确 |
| 相关性 | 检索结果是否与查询相关 |
| 时效性 | 检索结果是否最新 |
| 效率 | 检索延迟和成本 |

### 17.5 实现模式

```python
# 向量存储记忆
class VectorMemory:
    def __init__(self):
        self.embeddings = {}  # memory_id -> embedding
        self.contents = {}    # memory_id -> content
        self.importance = {}  # memory_id -> importance_score
    
    def add(self, content, importance=0.5):
        mem_id = generate_id()
        self.embeddings[mem_id] = embed(content)
        self.contents[mem_id] = content
        self.importance[mem_id] = importance
    
    def retrieve(self, query, k=5):
        q_emb = embed(query)
        scores = cosine_similarity(q_emb, list(self.embeddings.values()))
        top_k = argmax_k(scores, k)
        return [self.contents[i] for i in top_k]
```

## 第 18 章 Agent 编排

### 18.1 什么是 Agent Harness？

Agent Harness 是管理 agent 运行时的基础设施，负责上下文管理、提示组装、工具调度和状态跟踪。

### 18.2 上下文窗口管理

- **Token 预算监控**：实时跟踪上下文使用
- **上下文压缩**：用摘要代替完整历史
- **滑动窗口**：只保留最近 N 轮
- **递归上下文分解**：按重要性分层次

### 18.3 ReAct 循环

```
Thought: 我需要查询天气
Action: call get_weather(city="Beijing")
Observation: 晴天，25°C
Thought: 天气不错。我可以建议用户出门。
Action: respond("今天北京天气很好，适合外出！")
```

### 18.4 编排模式

| 模式 | 描述 | 适用场景 |
|---|---|---|
| ReAct | 推理-行动循环 | 通用 |
| Plan-and-Execute | 先计划再执行 | 复杂任务 |
| 多 Agent 编排 | 多个 agent 协作 | 大型项目 |
| 人在回路 | 人工审批关键决策 | 高风险场景 |
| 工作流图 | DAG 任务依赖 | 流水线任务 |

## 第 19 章 Agent 设计模式

### 19.1 工作流模式

- **Prompt Chaining**：将任务分解为顺序步骤
- **Routing**：根据输入类型路由到不同处理器
- **Parallelization**：并行执行独立任务
- **Orchestrator-Workers**：一个协调者 + 多个 worker
- **Evaluator-Optimizer**：一个生成 + 一个评估

### 19.2 自主 Agent 模式

- **ReAct**：推理 + 行动循环
- **Planning Agents**：预先规划子任务
- **Reflection and Self-Critique**：自我评估和修正
- **Tool-Use Patterns**：按需调用工具

## 第 20 章 Agentic 环境与基准

### 20.1 主要基准

| 基准 | 领域 | 描述 |
|---|---|---|
| SWE-bench | 软件工程 | 修复真实 GitHub issue |
| WebArena | 网页交互 | 在真实网站上完成任务 |
| OSWorld | 操作系统 | 桌面操作任务 |
| GAIA | 通用 | 多步推理 + 工具使用 |
| AgentBench | 通用 | 多维度 agent 能力评估 |

### 20.2 环境设计原则

- 观察空间设计：agent 能看到什么
- 动作空间设计：agent 能做什么
- 奖励信号设计：如何衡量成功
- 情节结构：回合开始和结束条件
- 难度课程：从简单到困难

## 第 21 章 模型上下文协议（MCP）

MCP [72] 是标准化的工具集成协议。

### 21.1 核心模型

- **客户端**：LLM 应用程序
- **服务器**：提供工具和资源
- **协议**：JSON-RPC 2.0 消息格式

### 21.2 核心原语

- **Tools**：可调用的函数（LLM 通过 tool call 调用）
- **Resources**：可读取的数据
- **Prompts**：可插入的提示模板
- **Sampling**：LLM 请求服务器生成文本

### 21.3 MCP 实现

```python
# MCP Server 示例
from mcp.server import Server, NotificationOptions
from mcp.server.models import InitializationOptions

app = Server("weather-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="get_weather",
            description="获取城市天气",
            inputSchema={
                "type": "object",
                "properties": {
                    "city": {"type": "string"}
                },
                "required": ["city"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name, arguments):
    if name == "get_weather":
        city = arguments["city"]
        return [TextContent(type="text", text=f"{city}: 25°C")]
```

### 21.4 MCP vs A2A

MCP 是 agent-工具协议；A2A 是 agent-agent 协议。两者互补。

## 第 22 章 Agent 技能

### 22.1 什么是技能？

技能是可复用的功能模块，agent 按需加载。

### 22.2 技能架构

- **静态加载**：预定义所有技能
- **动态发现**：agent 运行时搜索可用技能
- **分层组合**：技能调用子技能

### 22.3 技能生命周期

1. **创建**：定义技能接口和实现
2. **注册**：向技能目录注册
3. **发现**：agent 搜索相关技能
4. **加载**：按需加载
5. **执行**：运行
6. **卸载**：释放资源

## 第 23 章 Agent-to-Agent 通信（A2A）

### 23.1 Google A2A 协议

A2A [73] 让不同 agent 系统之间可以通信和协调。

### 23.2 关键概念

- **Agent Card**：描述 agent 能力和端点的元数据
- **Task Lifecycle**：任务从创建到完成的状态机
- **SSE Streaming**：服务器推送事件流
- **Push Notification**：长任务完成通知

### 23.3 通信模式

- **Request-Response**：同步请求
- **Streaming**：流式结果
- **Pub-Sub**：发布-订阅
- **Negotiation**：协商协议
- **Auction**：任务拍卖分配

### 23.4 A2A vs MCP

- **MCP**：agent → tool（我需要能力）
- **A2A**：agent → agent（我需要协作）

组合使用：一个 agent 通过 MCP 使用工具，通过 A2A 与其他 agent 协作。

## 第 24 章 多 Agent 系统

### 24.1 为什么需要多 Agent？

复杂任务需要多种专业技能。多 agent 系统通过分工和协作处理单个 agent 难以完成的复杂任务。

### 24.2 多 Agent 架构

| 架构 | 协调方式 | 适用场景 |
|---|---|---|
| 集中式 | Supervisor 管理 worker | 层次化任务 |
| 去中心化 | Peer-to-peer 通信 | 动态协作 |
| 分层 | 多级管理 | 大规模系统 |
| 群体 | 自组织 | 简单规则 + 涌现行为 |

### 24.3 协调机制

- **共享黑板**：公共消息池
- **消息传递**：直接 agent-agent 通信
- **规划和分解**：将任务拆分为子任务
- **投票和共识**：多 agent 投票决策
- **市场机制**：任务竞标

### 24.4 多 Agent RL

```
独立学习：每个 agent 独立训练
CTDE：集中训练，分散执行
通信学习：学习何时通信
涌现通信：自发形成通信协议
```

### 24.5 挑战

1. **协调开销**：通信成本随 agent 数增长
2. **信用分配**：归因于哪个 agent
3. **涌现行为**：不可预测的系统行为
4. **安全性**：恶意 agent 注入
5. **评估**：难以衡量整体系统性能

## 第 25 章 Agent 开发框架

### 25.1 主要框架

| 框架 | 特点 | 适用场景 |
|---|---|---|
| LangGraph | 状态图、条件路由、循环 | 复杂工作流 |
| AutoGen | 多 agent 对话 | 协作任务 |
| CrewAI | 角色分工 | 团队任务 |
| OpenAI Agents SDK | 简单、集成 OpenAI | 快速原型 |
| DSPy | 声明式编程 | 优化 prompt 流水线 |
| Semantic Kernel | 企业集成 | .NET 生态 |

### 25.2 测试和评估

- **单元测试**：测试单个工具
- **集成测试**：测试完整 agent 循环
- **回归测试**：用金牌轨迹验证
- **行为测试**：边界条件验证
- **成本和延迟测试**：性能和效率验证

## 第 26 章 Agentic UI 框架

### 26.1 UI 范式

- **聊天接口**：最简单的 agent UI
- **画布/Artifact 接口**：显示 agent 生成的产物
- **工作流可视化**：展示 agent 的规划步骤
- **仪表盘**：多 agent 监控
- **协作界面**：人-agent 协作

### 26.2 关键 UI 组件

- **思维过程显示**：展示 agent 的推理
- **工具使用可视化**：显示工具调用和结果
- **进度指示器**：长任务进度
- **审批关卡**：人类审批介入点
- **上下文显示**：当前记忆和状态

### 26.3 Generative UI

使用 React Server Components 动态生成 UI 组件：
- 工具调用结果直接渲染为交互式组件
- 图表、表格、表单由 agent 按需生成
- 用户直接在 UI 上编辑和确认

### 26.4 人在回路设计

- **何时打断 agent**：高成本或高风险动作前
- **分级审批**：不同风险级别不同审批方式
- **反馈机制**：用户指导 agent 的方式
- **教学交互**：用户通过 UI 教 agent 新技能

---

# 第六部分 评估与参考

## 测验与详细答案

第 26 章包含 500+ 道覆盖全书的测验题，涵盖从基础架构到高级训练的所有主题。每题附详细解析。

## 快速参考

### 核心公式

**Transformer 注意力**：Attention(Q,K,V) = softmax(QK^T/√d_k)V

**PPO-Clip**：L = E[min(r(θ)A, clip(r(θ), 1-ε, 1+ε)A)]

**DPO**：L = -E[log σ(β * (log(π_θ(y_w)/π_{ref}(y_w)) - log(π_θ(y_l)/π_{ref}(y_l))))]

**GRPO**：A_i = (R_i - μ_R) / σ_R, L = E[min(r_i A_i, clip(r_i, 1-ε, 1+ε)A_i) - β*KL]

### GPU 规格

| GPU | 显存 | 带宽 | FP16 TFLOPS |
|---|---|---|---|
| A100 | 80GB | 2TB/s | 312 |
| H100 | 80GB | 3.35TB/s | 989 |
| B200 | 192GB | 8TB/s | 2250 |

### 超参数范围

| 阶段 | LR | Batch | Epochs |
|---|---|---|---|
| 预训练 | 3e-4 → 3e-5 | 4M tokens | 1 |
| SFT | 2e-5 → 2e-6 | 512 seq | 2-3 |
| RLHF | 1e-6 → 1e-7 | 128 seq | 1 |

### 决策树

1. 要知识更新？ → RAG
2. 要行为改变？ → 微调
3. 要推理能力？ → GRPO / RL
4. 要对齐偏好？ → DPO / PPO
5. 要自主行动？ → Agent Framework

### 展望

Agentic AI 仍处在早期阶段。关键开放挑战包括：从交互中学习、可扩展监督、世界模型和规划、多 agent 生态系统、安全与信任、超越基准的评估、效率和可访问性。

**进一步阅读**：原始论文列表见原指南第 576-577 页参考文献。



---

