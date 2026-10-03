# Universal FDE skill

> **Stop automating from requirements. Train on reality.**  
> **别再从需求文档开始猜业务。让 AI 直接从真实生产中学习。**

---

Most enterprise automation starts with a document.

Universal FDE starts with **one real task**.

大多数企业自动化，从一份需求文档开始。

Universal FDE 从**一笔真实业务**开始。

```text
Real Work / 真实业务
        ↓
AI Observes / AI 观察
        ↓
AI Learns / AI 学习
        ↓
Human-Guided Execution / 人带 AI 实际执行
        ↓
Production Evidence / 生产证据
        ↓
Extract Stable Knowledge / 提取稳定能力
        ↓
Rules · Tools · Scripts · State Machines
规则 · 工具 · 脚本 · 状态机
        ↓
Production Workflow / 生产工作流
```

### The unusual idea / 一个反常规的核心思想

**The goal is not to make the Agent more autonomous forever.**

**目标不是让 Agent 永远获得越来越大的自由。**

Instead:

> Let AI explore broadly while the workflow is unknown.  
> Once something becomes stable, take it away from the AI and compile it into software.

换句话说：

> **业务还未知时，让 AI 广泛探索。**  
> **一旦某项能力被证明稳定，就把它从 AI 手里拿走，固化成程序。**

Eventually:

```text
AI      → uncertainty
AI      → 处理不确定性

Program → determinism
程序     → 处理确定性

Human   → exceptions + authority
人       → 处理异常与授权边界
```

---

# What is Universal FDE?
# Universal FDE 是什么？

Universal FDE is a **progressive training protocol for enterprise automation**.

It helps an Agent enter an unfamiliar real-world business process, learn how the work actually happens, verify what is true, discover exceptions, perform guarded real executions, and progressively convert stable knowledge into deterministic software.

Universal FDE 是一套面向企业自动化的**渐进式训练协议**。

它让 Agent 进入一个完全陌生的真实业务，通过真实案例、真实系统和真实操作逐步学习：

- 业务实际上怎么运行；
- 人到底在看什么；
- 哪些步骤只是机械操作；
- 哪些步骤需要判断；
- 什么才算真正成功；
- 哪些情况必须停止；
- 哪些经验已经稳定到可以写成程序。

最终，把人工经验逐步转化为：

```text
SOP
Rules
Schemas
Parsers
Tools
Scripts
Workflows
State Machines
Verification
Regression Tests
```

也就是：

**把人的生产经验，逐步编译成可验证的软件系统。**

---

# Why?
# 为什么需要它？

Because the real workflow is almost never fully written down.

因为真正的企业业务，几乎从来没有被完整写进 SOP。

A skilled employee may know:

一个熟练员工可能知道：

- which field actually matters / 哪个字段才真正重要；
- which system is the real source of truth / 哪个系统才是真实数据源；
- when the normal rule should not be used / 什么情况下不能走正常规则；
- how to recognize a silent failure / 怎么发现“看似成功，实际上失败”；
- when to retry / 什么时候重试；
- when to stop / 什么时候必须停止；
- what newcomers usually get wrong / 新人最容易在哪一步犯错。

Traditional automation tries to describe all of this first.

Universal FDE says:

传统自动化试图先把这些全部写成需求。

Universal FDE 的做法是：

> **Show the Agent one real task.**
>
> **先给 AI 看一笔真实业务。**

Then another.

再来一笔。

Then an exception.

再给它看一个异常。

Then let production evidence decide what becomes a rule.

最后，让真实生产证据决定什么可以成为规则。

---

# Not another giant prompt.
# 它不是另一个“超级 Prompt”。

Universal FDE is not about putting your entire company into one massive system prompt.

Universal FDE 不是把整个公司的规则塞进一个无限增长的 Prompt。

It uses an ordered training process:

它使用一套严格顺序推进的训练流程：

```text
00  Mission Initialization
    任务初始化

01  Production Discovery
    真实生产发现

02  Manual Workflow Reconstruction
    人工流程重建

03  Risk & Delegation Boundary
    风险与委托边界

04  Success / Failure / Verification Model
    成功、失败与验证模型

05  Technical Feasibility
    技术可行性验证

06  Minimal Safe Prototype
    最小安全原型

07  Production Shadow Learning
    真实生产影子学习

08  Human-Guided First Real Execution
    人带 AI 完成第一次真实执行

09  Guarded Autonomous Trial
    受控自主运行

10  Error Boundary & Negative Cases
    错误边界与负面案例

11  Capability Extraction
    稳定能力提取

12  Workflow Compilation
    工作流编译

13  State, Data & Audit
    状态、数据与审计

14  Production Hardening
    生产强化

15  Controlled Autonomy Expansion
    受控扩大自主权限

16  Performance & Cost Compression
    性能与成本压缩

17  Continuous Production Learning
    持续生产学习
```

Every stage has a Gate.

每个阶段都有 Gate。

**No evidence, no promotion.**  
**没有证据，就不能进入下一阶段。**

---

# The transformation
# 它真正做的事情

At the beginning:

一开始：

```text
Agent
↓
Look at screen
↓
Think
↓
Click
↓
Think again
↓
Click again
```

This is useful for learning.

这种方式适合学习。

But it should not remain this way forever.

但它不应该永远这样运行。

After enough real production evidence:

当真实生产证据足够之后：

```text
Unstructured Input
非结构化输入
        ↓
AI Interpretation
AI 语义理解
        ↓
Structured Output
结构化结果
        ↓
Program Validation
程序校验
        ↓
Deterministic Workflow
确定性工作流
        ↓
Independent Verification
独立验证
        ↓
Persistent State
持久状态
```

The AI returns only when something is genuinely uncertain.

只有真正遇到不确定情况时，才重新交给 AI。

---

# The principle
# 核心原则

> **Use AI to discover the system.**  
> **让 AI 帮你发现真实系统。**
>
> **Use production evidence to constrain it.**  
> **用真实生产证据约束 AI。**
>
> **Compile stable knowledge into software.**  
> **把稳定知识编译成软件。**

---

# Two files. One system.
# 两个文件，一套训练系统。

The core repository intentionally stays simple.

核心仓库故意保持清晰：

```text
Human-facing guide + Agent interaction reference
        ↓
Human ↔ Agent collaboration / interaction protocol

FDE_TRAINING_SKILL.md
        ↓
Authoritative Stage / Gate / State / execution protocol
```

[FDE_Human-facing guide + Agent interaction reference.md](<FDE_Human-facing guide + Agent interaction reference.md>) defines how the human and Agent collaborate. `FDE_TRAINING_SKILL.md` is authoritative for the staged workflow, Gates, project state, and execution rules.

[Human-facing guide + Agent interaction reference](<FDE_Human-facing guide + Agent interaction reference.md>) 说明人和 Agent 如何协作；`FDE_TRAINING_SKILL.md` 是阶段、Gate、项目状态和执行规则的权威来源。

This project is licensed for non-commercial use only. All commercial use requires prior written authorization; see [LICENSE](LICENSE).

本项目仅允许非商业用途。所有商业用途均须事先获得书面授权，详见 [LICENSE](LICENSE)。

The Codex skill entry point is `.codex/skills/universal-fde/SKILL.md`. When Universal FDE starts, the Agent reads the interaction guide, initializes `FDE_PROJECT_STATE`, retains the `GLOBAL CONSTITUTION`, retrieves only `FDE_STAGE_00_INIT`, and begins with the minimum critical questions needed to understand the real business. It does not design the final automation architecture or preload later stages. After a Gate passes, it updates project state before retrieving only the next Stage.

Codex skill 入口位于 `.codex/skills/universal-fde/SKILL.md`。Universal FDE 启动后，Agent 会先阅读交互指南、初始化 `FDE_PROJECT_STATE`、记住 `GLOBAL CONSTITUTION`，然后只读取 `FDE_STAGE_00_INIT`，并通过最少的关键问题了解真实业务。此时不会设计最终自动化架构，也不会预加载后续阶段。每个 Gate 通过后，先更新项目状态，再只读取下一阶段。

To begin, tell the Agent:

开始时，只需告诉 Agent：

```text
Use Universal FDE.
I want to train a new real business workflow.

使用 Universal FDE。
我要训练一个新的真实业务流程。
```

Then:

> **Show it one real task.**

然后：

> **给它看一笔真实业务。**

---
# From reality to software.
# 从真实生产，到软件系统。

**Observe reality. / 观察真实业务。**  
**Learn the workflow. / 学会业务流程。**  
**Verify what works. / 用证据验证。**  
**Compile what becomes stable. / 把稳定能力编译成程序。**

### Universal FDE

**Train on reality. Compile what works.**  
**从真实业务中学习，把有效经验编译成软件。**

