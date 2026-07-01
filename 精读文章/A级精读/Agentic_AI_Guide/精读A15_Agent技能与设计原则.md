# 精读A15：Agent技能体系与设计原则

## 0. 一句话定位

**Agent Skill（技能）是Agent的能力模块化单元——就像整车EE架构从单体ECU演进到面向服务的SOA架构，Agent也从"一个Prompt包打天下"走向"按需组合的Skill生态"。**

本文覆盖《Agentic AI Guide》第22章（Agent Skills：技能的架构模式、生命周期、注册市场、与Fine-Tuning的对比）和第19章（Agent Design Patterns：工作流模式、自主Agent模式、设计原则与模式选择指南）。

---

## 1. 什么是Agent Skill

### 1.1 定义

Skill是Agent的一个**自包含的能力模块**，它让Agent具备在特定领域或任务中的专业知识。与一个原始Tool（单个函数）不同，Skill封装了五个维度：

| 维度 | 内容 | 三电类比 |
|------|------|----------|
| **System Prompt（系统提示）** | 领域特定的指令、约束、人设 | 就像每个ADAS功能有自己的控制策略MAP |
| **Tool Bindings（工具绑定）** | 所需的API、MCP服务器、本地命令 | 相当于调用传感器（雷达、摄像头、IMU）获取数据 |
| **Knowledge（知识）** | 参考材料、示例、Few-shot演示 | 类似标定参数表，告诉系统"什么样的工况该怎么处理" |
| **Workflow Logic（工作流逻辑）** | 多步流程、决策树、条件分支 | 类似自动泊车的完整逻辑链：寻位→路径规划→执行→纠偏 |
| **Guardrails（护栏）** | 安全约束、输出格式验证 | 像扭矩安全监控（Torque Security Monitor）限制最大输出 |

### 1.2 Skill vs Tool vs Agent

原文用一个精妙的比喻总结了三者的层次关系：

> **Tool是锤子，Skill是知道怎么盖房子，Agent是选择用哪个Skill的工匠。**

用三电语言说得更直白：

| 概念 | 三电类比 | 说明 |
|------|----------|------|
| **Tool** | 一个传感器（单目摄像头） | 单一功能函数，如 `web_search(query)` |
| **Skill** | 一个功能模块（自动泊车APA） | 封装了提示词+工具+知识+工作流：`"泊车技能"` |
| **Agent** | 整车域控制器（VCU/HPC） | 自主实体，根据当前任务选择并编排多个Skill |

---

## 2. Skill架构模式

### 2.1 静态加载（Static Loading）

Agent初始化时根据配置一次性加载所有Skill。

- **优点**：简单、可预测、延迟低
- **缺点**：不用的Skill浪费上下文窗口；无法扩展到数百个Skill
- **三电类比**：==出厂预装的所有功能==——APA、ACC、LKA等全部在固件里，不管你用不用，它们都占用Flash和RAM

### 2.2 动态发现（Dynamic Discovery）

Agent根据当前任务实时选择激活哪些Skill。一个**Skill Router**（轻量分类器或Embedding匹配器）确定相关性。

- **优点**：支持大规模Skill库；上下文高效
- **缺点**：路由错误可能遗漏相关Skill；增加延迟
- **三电类比**：==OTA后新功能上线==——你开车回家时系统只激活导航和泊车技能，不需要加载越野模式；路由判断当前"场景"是什么

### 2.3 层级组合（Hierarchical Composition）

Skill可以依赖其他Skill，形成**有向无环图（DAG）**。高层Skill调用子Skill，子Skill可以被多个父Skill共享。

- **三电类比**：==整车功能层级树==——"自动泊车"高层Skill下面包含"车位识别"、"路径规划"、"执行控制"三个子Skill，而"路径规划"又可以同时被"自动泊车"和"记忆泊车"复用

原文给出的例子：`"Deploy Application"` 依赖 `"Run Tests"` → `"Build Docker Image"` → `"Update DNS"`。

---

## 3. Skill生命周期

| 阶段 | 说明 | 三电类比 |
|------|------|----------|
| **① Discovery（发现）** | 系统识别可用Skill（注册中心、市场、本地定义） | 整车诊断发现所有可用的功能模块 |
| **② Selection（选择）** | 根据用户请求匹配并加载相关Skill | 检测到"我要停车"，系统筛选出APA/AVP相关模块 |
| **③ Activation（激活）** | 将Skill的Prompt、工具、知识注入Agent上下文 | 把APA控制策略载入到域控制器的运行时内存 |
| **④ Execution（执行）** | Agent使用Skill完成任务 | APA执行：寻位→路径→控制→完成 |
| **⑤ Deactivation（停用）** | 移除Skill上下文以释放窗口空间 | 退出泊车模式，释放内存给下一个功能 |
| **⑥ Learning（学习）** | 执行结果更新Skill的Few-shot示例或路由模型 | 系统学习该车主更偏好"倒车入库"而非"侧方停车" |

Skill生命周期的核心洞察：**Skill不是一次性部署的，而是持续演化的版本化组件**。原文特别强调了**Skill Manifest（清单）**的作用：

```json
{
  "name": "code-review",
  "version": "2.1.0",
  "requires": {
    "tools": ["file_read", "grep", "git_diff"],
    "mcp_servers": ["github"],
    "models": ["claude-sonnet-4-20250514"]
  },
  "prompts": ["skills/code-review/system.md"],
  "knowledge": ["skills/code-review/style-guide.md"]
}
```

这个Manifest就像车辆的**功能描述文件（如ARXML中的SWC描述）**——定义了功能名称、版本、依赖的硬件资源（传感器）、使用的算法模型。

### 注册中心与市场

生产级Skill系统需要的基础设施：
- **Skill清单**：结构化描述（名称、能力、所需工具、输入输出Schema）
- **版本控制**：Agent需要锁定特定版本以获得可复现性
- **依赖解析**：Skill可能依赖特定的MCP Server、API Key或其他Skill
- **权限模型**：不是所有Agent都能访问所有Skill（安全、成本、能力边界）
- **市场（Marketplace）**：组织可以发布、分享和安装Skill，===类似应用商店下载新功能===

---

## 4. Skills vs Fine-Tuning

这是一个核心决策问题：为什么用运行时Skill注入，而不是直接Fine-Tuning模型？

| 维度 | Skill（上下文注入） | Fine-Tuning | 三电类比 |
|------|--------------------|-------------|----------|
| **部署速度** | 瞬时 | 数小时到数天 | 在线OTA刷写功能（秒级） vs 产线刷写整版固件（小时级） |
| **灵活性** | 运行时自由切换/组合 | 训练时固定 | 按需开启功能 vs 重新标定整个电驱系统 |
| **上下文成本** | 消耗上下文窗口 | 运行时零成本 | 占用实时内存 vs 固化在Flash中，不占运行内存 |
| **深度行为改变** | 受限于上下文长度 | 深度参数改变 | 查表映射修改 vs 重新训练神经网络权重 |
| **多租户** | 不同用户不同Skill | 所有用户同一模型 | 每辆车可单独选装配置 vs 统一硬件平台 |
| **维护** | 更新文本文件 | 重新训练新数据 | 更新配置文件即可 vs 需要重新做台架标定 |

### 两者是互补的

核心结论：**Fine-Tuning提供基础能力（指令遵循、工具调用格式、推理能力），Skill在运行时提供任务特定的专业知识层叠在上面。**

==Fine-Tuning = 标定电驱系统的基础MAP（扭矩计算的基础公式）==
==Skill = 按需启动的专项功能（雪地模式、运动模式、经济模式）==

---

## 5. 设计原则

原文第19章和第22章共同提炼出以下**六个跨模式的设计原则**：

### 原则1：保持简单（Keep It Simple）

> 用最简单的架构开始。只有在证明必要后才增加复杂度。**一个能解决问题的Prompt Chain，永远好过一个"可能"更好但更复杂的多Agent系统。**

**三电注解**：不要为普通定速巡航去设计一个L4自动驾驶系统。一个PID控制器能解决的问题，不需要上深度强化学习。

### 原则2：透明胜过聪明（Transparency Over Cleverness）

> 每一步都应该是可检查的。避免隐藏状态或隐式推理。当Agent失败时，你需要知道为什么——不透明的架构让调试不可能。

**三电注解**：诊断协议（UDS）的每个DTC都要能追溯到根因。如果"电机过热"报警你不知道是温度传感器故障还是冷却泵失效，那就没法做根因分析。

### 原则3：提供好的工具（Provide Good Tools）

> 文档清晰、类型完备、错误信息明确的工具是力量倍增器。一个模糊描述的工具会被误用；一个精确Schema的工具会被正确选用。

**三电注解**：每个传感器接口必须有明确的信号范围、精度、故障值定义。一个说"IMU数据"的接口和说"IMU三轴加速度，±4g范围，16位分辨率，0xFFFE表示故障"的接口，可用性天差地别。

### 原则4：为失败做规划（Plan for Failure）

> 每次Tool调用都可能失败。在引擎层（Harness）构建重试逻辑、回退和优雅降级，这样模型不需要推理基础设施故障。

**三电注解**：功能安全ISO 26262的核心思想——每个功能必须有故障处理路径。毫米波雷达失效→降级到超声波+视觉融合；再失效→提示驾驶员接管。

### 原则5：使用结构化输出（Use Structured Outputs）

> 约束生成（JSON Schema、Function Calling）防止解析失败。一个产生自由文本需要正则解析的Agent是脆弱的；一个产生已验证JSON的Agent是鲁棒的。

**三电注解**：CAN/LIN总线上的信号是严格定义的DBC——每个信号的位置、长度、取值范围都是确定的。Agent的输出也应该像CAN帧一样可解析。

### 原则6：用多样化的输入测试（Test with Diverse Inputs）

> Agent的行为比单轮Chat变化更大。同一个Prompt在不同运行时可能产生不同的Tool调用序列。用边缘案例、模糊请求和异常输入进行对抗性测试。

**三电注解**：标定测试需要覆盖全工况（高温、高寒、高海拔、坏路）。Agent测试也一样，需要覆盖各种输入模式。

### Anthropic的核心理念：Augmented LLM

> **Augmented LLM = Model + Retrieval + Tools + Memory**

这是Agent设计的基本原子单元——不再是裸模型，而是模型+检索源+可调用工具+持久记忆的组合体。这个单元实质上就是一个**Skill赋能的模型**。原文强调：

> 最有效的Agent不是最复杂的Agent。它们是带有好工具的简单循环。

```python
while not done:
    action = llm.decide(context, tools)
    result = execute(action)
    context.append(result)
    done = llm.should_stop(context)
```

---

## 6. 关键概念速查表

### Workflow Patterns（第19章）

| 模式 | 复杂度 | LLM调用次数 | 最佳场景 |
|------|--------|-------------|----------|
| **Prompt Chaining** | 低 | N（固定） | 顺序任务，内容Pipeline |
| **Routing** | 低 | 1 + 1 | 多类型输入、分流 |
| **Parallelization** | 低 | N（并行） | 独立子任务、投票 |
| **Orchestrator-Workers** | 中 | 可变 | 未知分解方式的复杂任务 |
| **Evaluator-Optimizer** | 中 | 2–10（循环） | 质量关键型输出 |

### Autonomous Agent Patterns（第19章）

| 模式 | 复杂度 | LLM调用次数 | 最佳场景 |
|------|--------|-------------|----------|
| **ReAct** | 中 | 3–25（循环） | 通用工具使用、探索 |
| **Planning Agent** | 高 | 5–50+ | 长周期、多步骤任务 |
| **Reflection** | 高 | +50%开销 | 首次尝试常失败的任务 |
| **Multi-Agent** | 高 | 很多 | 复杂领域、专业化分工 |

### Tool Invocation Patterns（第19章）

| 模式 | 三电类比 |
|------|----------|
| **Single-turn**（单次Tool调用） | 一次CAN请求获取车速信号 |
| **Multi-tool**（并行多Tool调用） | 同时读取车速、转速、电池SOC |
| **Sequential**（顺序Pipeline） | SOC低→限制功率→提醒充电 |
| **Nested**（子Agent嵌Tool） | 域控调度MCU执行具体电机控制 |
| **Fallback**（回退降级） | 毫米波雷达→超声波→摄像头降级 |

---

## 7. PM视角

### 核心战略建议

**1. Skill体系就是Agent时代的能力复用架构**

正如整车从"一个ECU管一个功能"走向"面向服务的SOA架构"，Agent从"一个Prompt解决所有问题"走向"Skill市场按需组合"。如果团队正在构建Agent产品，想清楚Skill的粒度、注册标准和复用策略——这决定了后续你能多快响应新需求。

**2. 静态加载 → 动态发现 → 层级组合是渐进演进路径**

不要一开始就设计完美的动态路由系统。从静态加载开始（最简单），当Skill数量超过某个阈值（约10-15个）时引入动态发现，然后再考虑层级组合。这与整车EE架构的演进路径一致：先集中式→再域控→再准中央计算。

**3. Skill vs Fine-Tuning的决策框架**

- 需要**快速迭代、频繁变更**的任务 → 用Skill（像OTA频繁更新功能）
- 需要**深度行为改变、基础能力提升**的任务 → 用Fine-Tuning（像硬件平台升级）
- **最佳实践**：Fine-Tuning打底 + Skill按需叠加

**4. Skill市场是生态竞争的制高点**

如果一个平台能吸引第三方开发者发布Skill，它就拥有了类似App Store的网络效应。Skill Manifes的标准化是第一步——就像当年AUTOSAR标准化了SWC接口描述。

### 决策矩阵：选择Agent模式

原文给出清晰的决策路径：**从最简单的开始，只在更简单模式证明不够时才向下移动。**

```
任务结构已知？     →    Workflow（Chaining / Routing）
有独立子任务？     →    Parallelization
子任务不确定？     →    Orchestrator-Workers
质量需要迭代？     →    Evaluator-Optimizer
开放探索？         →    ReAct / Planning
需要自省纠错？     →    Reflection
需要专业分工？     →    Multi-Agent
```

---

## 8. 记忆钩子

这些类比帮你快速向同事讲清楚Agent Skills：

| 钩子 | 一句话 | 对应的原文概念 |
|------|--------|---------------|
| **"功能模块"** | Skill就像电动车的自动泊车功能——封装了所有需要的东西（感知→规划→控制→安全监控） | Skill的定义：Prompt + Tools + Knowledge + Workflow + Guardrails |
| **"出厂预装 vs OTA新增"** | 静态加载 = 出厂所有ADAS功能全激活；动态发现 = OTA后识别新功能 | Static Loading vs Dynamic Discovery |
| **"应用商店"** | Skill市场 = 从Vehicle App Store下载新功能 | Skill Marketplace |
| **"重标定 vs 开功能"** | Fine-Tuning = 重新标定整个电驱系统；Skill = 按需开启雪地模式 | Skills vs Fine-Tuning |
| **"传感器降级"** | 毫米波雷达坏了→超声波顶上；Tool失败也有Fallback | Plan for Failure / Graceful Degradation |
| **"CAN帧协议"** | Agent的输出要像CAN报文一样结构化可解析 | Structured Outputs / JSON Schema |

---

*本文基于《The Hitchhiker's Guide to Agentic AI》Chapter 22（Agent Skills）和Chapter 19（Agent Design Patterns）精读整理，聚焦原文核心论点，辅以整车三电领域类比，供产品与技术决策参考。*
