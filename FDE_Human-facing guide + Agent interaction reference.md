# Universal FDE Training — Human-facing guide + Agent interaction reference

> Version: 0.2-rc2

> 给业务负责人、运营人员、实施人员、产品经理、工程师，以及第一次用 Agent 学习、辅助或自动化企业业务的人。

本文件解决两个问题：

1. **人应该怎样把一个真实业务交给 Agent 学习。**
2. **怎样启动一个会按阶段逐步推进，而不是一上来就乱写自动化方案的 FDE 会话。**

核心 Skill 位于：`FDE_TRAINING_SKILL.md`

---

# 1. 这套方案是干什么的

如果你有一项真实工作，例如：

- 员工每天从邮件里读取信息再录入 ERP；
- 客服从聊天记录整理数据到 CRM；
- 财务人员根据多个文件核对数据；
- 运营人员在多个后台之间复制、判断、提交；
- 工程人员从报告、图片、表格或设备状态中提取信息；
- 团队每天执行一套重复但存在少量判断的业务流程；
- 专家长期使用 AI 做辅助分析，但最终决定必须由人做；
- 一次性的企业数据迁移、尽调、盘点或流程重构项目；
- 设备/现场业务中，AI 只能观察或建议，实体动作永久由人执行；

你通常不能只对 AI 说：

> “帮我把它自动化。”

原因很简单：

AI 不知道员工脑子里那些没有写下来的经验，也不知道真实系统里哪些按钮只是查看、哪些按钮会改变生产状态，更不知道什么情况算真正成功。

因此，本项目采用一种**逐步训练法**：

> 人先把 AI 带进真实业务；  
> AI 通过提问、观察、Demo、跟做和真实案例理解流程；  
> 然后把稳定能力逐步转换成程序、脚本、规则、状态机和验证机制。

最终目标不是“做一个什么都自己判断的 Agent”，而是得到一个：

**AI / Program / Human 按任务需要组合：稳定确定的部分尽量工程化，真正需要推理的部分可以保留 Agent，人类可以永久保留在正常主路径和授权边界。**

这套方法不要求最终一定“全自动”。

---

# 2. 这不是传统模型训练

这里的“训练”通常不是：

- 微调基础模型
- LoRA
- 重新训练参数

而是：

1. 训练 AI 理解真实业务。
2. 训练 AI 知道什么时候不能继续。
3. 训练出稳定的业务规则。
4. 训练出可靠的工具和控制方式。
5. 把反复验证成功的经验编译成程序。
6. 用生产失败继续完善规则、状态和验证。

可以理解为：

> **把人的生产经验逐步编译成可验证的 AI + Software 工作流。**

---

# 3. 你需要准备什么

不需要在开始前写一份完美需求文档。

最好准备以下内容中的一部分：

- 一位真正做这项业务的人；
- 1–5 个普通真实案例；
- 1–3 个出过问题或比较特殊的案例；
- 能打开真实生产系统的环境（如果该业务有这样的系统且允许访问）；
- 现有 SOP、Excel、PDF、截图、聊天记录、表单或内部说明；
- 如果有测试环境或测试账号，更好；
- 能说明“什么叫完成”的人。

最重要的不是文档，而是：

> **真实生产轨迹。**

如果一个熟练员工愿意从头演示一单，通常比十页抽象需求更有价值。

---

# 4. 正确的使用方式

本项目核心只有两个大文件：

- `FDE_TRAINING_SKILL.md`：给 Agent 使用。
- `FDE_Human-facing guide + Agent interaction reference.md`：给人使用。

运行时不要把整个 Skill 一次塞给模型。

正确方式是：

```text
打开一个新会话
↓
粘贴本文件中的 Start Prompt
↓
Agent 初始化 FDE_PROJECT_STATE
↓
Agent 读取 GLOBAL CONSTITUTION
↓
Agent 只检索当前 Stage
+ 该 Stage 明确要求的历史产物
↓
Agent 向你提问 / 做当前阶段测试
↓
当前 Stage 通过 Gate
↓
Agent 更新 Project State
↓
只检索下一 Stage
↓
继续
```

这样做有三个好处：

1. 上下文更干净。
2. Agent 不容易跳步骤。
3. 每一步都有明确目标和停止条件。

需要注意：**“每次只检索当前 Stage”并不等于不能读取历史产物。** 当前 Stage 可以读取它自己列出的 `REQUIRED_ARTIFACTS`（例如前一次 Golden Trace、SOP、失败日志），但不应提前读取未来 Stage。

阶段结果只有四种：

- `PASS`
- `BLOCKED`
- `BACKTRACK`
- Skill 明确允许时的 `NOT_APPLICABLE`

---

# 5. Start Prompt — 直接复制到新会话

下面内容是整个项目的人类入口。

你可以直接复制给支持文件检索 / 本地仓库 / GitHub / Tool Use 的 Agent、Clawbot、Codex 或类似系统。

```text
你现在进入 Universal FDE Progressive Training 模式。

你的任务不是立刻替我写一套完整自动化，而是把我的真实企业业务逐步训练成一个可验证、可执行的运行体系。AI、Program、Human 按业务需要组合；不要预设最终必须全自动。

你必须使用仓库中的：
FDE_Human-facing guide + Agent interaction reference.md
FDE_TRAINING_SKILL.md

其中：
- FDE_Human-facing guide + Agent interaction reference.md 定义 Human–Agent collaboration / interaction protocol。
- FDE_TRAINING_SKILL.md 是 Stage / Gate / FDE_PROJECT_STATE / Evidence / Runtime Rules 的权威来源；如果两者在执行协议上出现冲突，以 FDE_TRAINING_SKILL.md 为准。

运行规则：

1. 首先读取并长期遵守 FDE_TRAINING_SKILL.md 中的 GLOBAL CONSTITUTION。
2. 初始化并持续维护 FDE_PROJECT_STATE。
3. 从 FDE_STAGE_00_INIT 开始。
4. 每次只检索当前 Stage，以及该 Stage 明确列出的 REQUIRED_ARTIFACTS；不要一次性读取整个 Skill，也不要预读未来 Stage。
5. 当前 Stage 只有 PASS，或 Skill 明确允许且有证据的 NOT_APPLICABLE，才可以离开；否则不得进入下一 Stage。
6. 你需要主动向我提问，但不要一次丢给我几十个问题；按真实业务逐步追问。
7. 优先让我提供真实案例、真实界面、真实文件和实际操作，而不是让我写抽象需求文档。
8. 不知道的内容必须标记 UNKNOWN；推断只能标记 ASSUMPTION；未经验证不能变成 VALIDATED_RULE。
9. 如果存在真实系统，你必须尽可能用观察、测试和结果证明“能不能做”，不能只口头说可行。
10. 在早期真实生产探索中，默认不得执行最终提交、外部发送、审批、删除、付款或其他不可逆动作；到权限边界应暂停并说明。
11. 每个阶段结束时，向我给出：
   - 本阶段确认的事实
   - 仍然未知的内容
   - 新增产物
   - 暴露的问题
   - Gate 是否通过
   - 下一步为什么要做
12. 达到 Gate 后，更新 FDE_PROJECT_STATE 与 Gate Evidence，然后只读取下一 Stage。
13. 如果后续真实生产事实推翻了前面的结论，回退到最早产生错误假设或失效产物的 Stage；修复后重新经过所有受影响的 Gate。
14. 不要把所有问题都通过增加 Prompt 解决。稳定、重复、确定性的能力，应逐步下沉为规则、程序、脚本、状态机、Schema 和验证机制。
15. 你的目标不是最大化“全自动覆盖率”，而是找到正确的目标运行模式：该自动的自动，该由受约束 Agent 处理的保留 Agent，该永久由人决定的保留 Human Gate。
16. 每个 Stage 结束只能报告 PASS / BLOCKED / BACKTRACK / 合法的 NOT_APPLICABLE。NOT_APPLICABLE 只能在 Skill 明确允许且有证据时使用，不能拿来跳关。
17. 从第一次真实执行开始，不得只靠聊天记忆判断任务状态，必须维护最小执行 ledger。

现在不要给我完整架构方案。

先执行 FDE_STAGE_00_INIT，并从最少但最关键的问题开始了解我的真实业务。
```

---

# 6. 如果 Agent 没有自动找到 Skill

如果你的工具没有自动加载仓库，可以再补一句：

```text
请先在当前仓库中搜索 FDE_TRAINING_SKILL.md，读取 GLOBAL CONSTITUTION 和 FDE_STAGE_00_INIT。不要读取后续 Stage。
```

如果是 Codex / Coding Agent，可以要求它先把仓库加入当前工作区，再按上面流程运行。

---

# 7. 人类在训练中的真正角色

你不是给 AI 写一份完整需求然后离开。

你主要做五件事。

## 7.1 提供真实事实

AI 不知道真实生产环境。

你需要提供：

- 一笔真实案例
- 一个真实页面
- 一个真实文件
- 一段真实操作
- 一个真实异常

越接近现实越好。

## 7.2 纠正 AI 的理解

AI 可能会把“通常”理解成“必须”。

例如你说：

> “一般这个客户都是 A 类。”

如果它写成：

> “客户类型 = A。”

你需要立刻纠正。

因为 FDE 最重要的事情之一就是：

**把事实、经验、默认值和硬规则分开。**

## 7.3 在第一次真实执行时监督

第一次真实任务不要追求快。

目标是暴露：

- 哪一步 AI 不理解；
- 哪一步工具不好用；
- 哪一步业务规则没写；
- 哪一步人平时靠经验；
- 哪一步系统结果不能确认。

## 7.4 确认新的业务规则

AI 可以提出：

> “我观察到三次都是这样，是否可以把它写成规则？”

只有你确认或系统证据充分后，它才能成为正式规则。

## 7.5 决定什么时候扩大权限

Agent 不应该自己宣布：

> “我已经很熟了，所以现在可以全自动提交。”

权限扩张应该由真实运行证据支持，并由人明确允许。

---

# 8. 整套训练流程，你会经历什么

下面是人类视角的简化版。

## Stage 00 — 初始化

AI 先搞清楚：你到底想完成什么业务结果。

它不会立刻给你技术方案。

## Stage 01 — 看真实业务基线

有现成流程时，AI 优先看一笔真实任务从开始到结束；如果是 Greenfield 新流程，则看真实输入、真实约束、周边系统和人类现在如何决策，并把尚不存在的步骤明确标成设计候选。

重点是区分：**现实已经存在什么，哪些只是准备设计。**

## Stage 02 — 把业务流程写清楚

AI 把工作拆成：

`输入 → 动作 → 判断 → 输出 → 成功/失败 → 下一步`

形成第一版操作指南。

## Stage 03 — 定义谁可以做什么

开始划分：

- AI 做什么
- 程序做什么
- 人做什么
- 哪些动作必须停下来确认

## Stage 04 — 先定义“需要怎样证明做对了”

这里先设计成功、失败和不确定状态的判据。

例如：

不是“提交按钮点了就算成功”，而是：

> 理想情况下，提交后应重新读取目标记录，确认状态真的改变。

注意：Stage 04 只定义**验证要求**，不假装技术上已经实现。

## Stage 05 — 实际证明技术可行性

AI/工程 Agent 开始试：

- 能不能读取系统；
- 能不能控制；
- 能不能确认结果；
- 有没有 API / DOM / CLI / 文件接口；
- 实在没有，才考虑视觉操作。

这一阶段真正测试 Stage 04 设计的读取、控制和验证方式，把它们标成“已证明可用 / 只能人工确认 / 当前阻断”。只证明“能不能做”，不追求漂亮。

## Stage 06 — 做最小 Demo

先跑一条最普通的测试路径。没有测试环境时，可以用 dry-run、历史 replay、模拟、数字孪生或人工接力证明最短闭环，不要求冒险碰真实不可逆动作。

可能很慢、很贵、很笨。

没关系。

只要它把真实问题暴露出来。

## Stage 07 — 用真实生产事实校准

优先现场 Shadow；Greenfield 可以用 controlled pilot；如果业务无法安全或及时现场观察，也可以用真实历史记录、录屏、日志、审计轨迹或专家逐步复盘。

默认不允许未授权的最终动作。

目标是验证 SOP 是否真的符合业务，而不是为了形式强求“现场盯着人做”。

## Stage 08 — 人带 AI 做第一笔真实任务

真实写操作开始前，先建立两样最小基础设施：

1. **Minimal Execution Ledger**：至少知道这条任务是谁、做到哪、外部动作有没有发生、验证结果是什么。
2. **Minimum Error Boundary**：关键 Unknown、冲突、验证失败、越权和超过 Retry 上限都必须停止。

然后由人监督，AI 一步一步做。

遇到卡住，不绕过，现场分析原因。

这条成功轨迹会成为第一条 Golden Trace。

## Stage 09 — AI 第一次受控独立跑

人不逐步告诉它该做什么。

AI 自己推进，但仍使用 Stage 08 建立的最低错误边界和执行 ledger，并在危险动作前暂停。

这里主要看：

> 它是不是真的掌握了任务状态。

如果目标运行模式本来就是永久 `READ_ONLY / HUMAN_GUIDED`，且人必须在每个物质性决策前介入，那么“独立推进”可以合法 N/A；不能为了追求自动化率强行做 Stage 09。

## Stage 10 — 专门训练“什么时候不能做”

主动制造或收集：

- 缺数据
- 冲突
- 模糊
- 重复
- 状态不一致
- 网络失败
- 页面变化

系统必须知道什么时候停止。

## Stage 11 — 把值得工程化的稳定能力从 AI 手里拿走

如果 AI 每次都在重新找同一个按钮，就做 Selector。

如果 AI 每次都在重新判断固定规则，就做 Rule。

如果 AI 每次都把同一种文本转成同一个格式，就做 Parser。

如果 AI 每次都在重复同一组动作，就做 Script。

## Stage 12 — 合并成受约束 Runtime Workflow

不是强迫整个业务变成固定脚本，而是把流程拆成三种 Segment：

- `DETERMINISTIC`：程序/规则稳定执行；
- `AGENTIC_BOUNDED`：AI 仍需要推理，但有输入输出、允许工具、停止、验证和人工升级契约；
- `HUMAN`：业务本来就要求人判断或授权。

输入本来就是结构化的，可以完全没有 AI；高度语义化的知识工作也可以保留 Agentic Segment。

目标不是“去掉 AI”，而是**去掉没有必要的自由度**。

## Stage 13 — 把最小状态升级为正式持久化和审计

Stage 08 已经要求有最小执行 ledger；这里把它升级成与项目匹配的状态和审计架构。可以是数据库，也可以是文件/日志、外部 Source of Truth、工作流引擎或仍然足够的轻量 ledger，不强制为了形式建数据库。

系统开始正式、持久地知道：

- 这条任务处理过没有；
- 现在进行到哪里；
- 为什么失败；
- 人改过什么；
- 用的是哪个版本规则。

## Stage 14 — 主动攻击系统

测试：

- 重复执行
- 重启
- 断网
- 超时
- 并发
- 登录失效
- 历史数据再次出现

把 Demo 变成生产系统。

## Stage 15 — 扩大自主权限

不是一次性“打开全自动”。

而是按动作一类一类开放。

如果项目根本没有要扩大自主权限的外部动作，这一阶段可以按 Skill 的规则记录为 `NOT_APPLICABLE`，但不能无理由跳过。

## Stage 16 — 再优化速度和成本

现在才系统性优化：

- 少调用模型
- 批处理
- 缓存
- 小模型
- 程序化
- 缩短上下文

但不能为了便宜删掉验证。

## Stage 17 — 按项目生命周期收尾

如果是持续业务，进入生产学习循环：每个错误都判断该改 Prompt、Rule、Tool、State Machine 还是 Human escalation。

如果是一次性/阶段性项目，则做 Finite Closeout：冻结最终版本、归档证据和回归用例、记录限制与 reopen triggers，然后可以正式 `PROJECT_COMPLETE`。

Universal FDE 不强迫一次性项目永远运行。

---

# 9. 一个最小真实例子

假设你说：

> “我们每天收到 Excel，然后有人把数据录进内部 ERP，我想自动化。”

错误的开局是：

> “可以，用 Python + Playwright。”

FDE 会这样开始。

### 第一步

AI 问：

> “先拿今天刚处理完的一条普通记录。Excel 是从哪里来的？员工看到它之后第一步实际做什么？”

你回答：

> “先看客户编号，然后去 ERP 搜客户。”

AI 继续：

> “搜到以后，你怎么确认是同一个客户？只看编号，还是还会核对名称、公司或其他字段？”

继续追问，直到一笔业务走完。

然后 AI 才开始测试：

- Excel 能不能结构化读取；
- ERP 有没有 API；
- 没有 API 是否能读取 DOM；
- 搜索结果是否有唯一 ID；
- 写入后怎样确认保存成功。

最终系统可能变成：

```text
Excel parser
↓
Program validation
↓
ERP lookup API
↓
AI only handles ambiguous customer matching
↓
Program writes deterministic fields
↓
Program re-reads record
↓
Verify
↓
Audit log
```

此时 AI 并不是“变得更会点 ERP”。

相反，它需要亲自处理的事情变少了。

这通常才是正确方向。

---

# 10. 什么情况下应该停下来，而不是继续自动化

出现以下情况之一时，应该停在当前 Stage：

- 根本不知道什么叫任务成功；
- 没有定义任何可靠的权威来源、来源优先级或冲突裁决策略；
- 写入后无法验证是否成功；
- 真实业务人员自己也说不清关键判断；
- 必须靠猜测填关键数据；
- 系统没有任何稳定控制方式；
- 大量异常尚未理解；
- Agent 需要频繁“猜现在是什么状态”；
- 同一任务可能被重复执行而无法识别；
- 失败后不知道到底有没有已经产生外部结果。

停下来不代表项目失败。

有时正确结果不是继续向前，而是：

- `BLOCKED`：当前缺必要事实/环境/证据；
- `BACKTRACK`：后面的真实事实证明前面理解错了，需要退回最早受影响阶段；
- `NOT_APPLICABLE`：仅当 Skill 明确允许，而且这一阶段对当前业务确实不存在时使用。

它意味着：

> 当前缺的是业务信息、系统控制面或验证机制，而不是更多 Prompt。

---

# 11. Prompt 应该怎么写

不要把整个企业规则塞进一个无限增长的超级 Prompt。

一个生产 Prompt 最好只负责一个任务。

推荐结构：

```text
ROLE
TASK
INPUT
SOURCE OF TRUTH / AUTHORITY POLICY
OUTPUT SCHEMA
RULES
FORBIDDEN
FAILURE BEHAVIOR
EXAMPLES
```

例如，数据提取任务应该允许：

- `unknown`
- `null`
- `conflict`
- `error`

而不是逼模型每个字段都给出答案。

如果一条规则已经非常稳定，例如：

> “金额必须保留两位小数。”

它通常更适合程序校验，而不是每次提醒模型。

---

# 12. AI、程序、人怎么分工

一个简单判断法：

## 交给 AI 的问题

> “需要理解含义吗？”

例如：

- 读自然语言
- 看复杂图片
- 判断意图
- 理解异常说明
- 处理低频长尾输入

## 交给程序的问题

> “答案是否确定，而且可以写成规则？”

例如：

- 日期格式
- 数值计算
- 去重
- 数据库查询
- 状态转换
- API 调用
- 页面固定字段
- 重试和超时
- 日志

## 留给人的问题

> “如果做错，系统现在还没有可靠办法证明和阻断吗？”

例如：

- 新异常
- 高影响批准
- 多个关键来源冲突
- 业务政策变化
- 无法自动验证的决定

---

# 13. 如何判断项目正在往正确方向发展

一个常见的健康轨迹是：

### 早期

AI 做很多事情。

因为系统还不知道真正需要什么能力。

### 中期

开始出现：

- 页面地图
- 小工具
- Schema
- Rule
- Parser
- 错误边界

### 后期

普通路径越来越少依赖模型逐步判断。

结构变成：

```text
AI 处理输入中的不确定性
↓
Program 校验
↓
Workflow 执行
↓
Program 验证
↓
异常才回 Agent / Human
```

如果上线很久以后，Agent 仍然每处理一单都需要重新：

- 看整个屏幕
- 找按钮
- 猜状态
- 重新决定固定流程

通常说明还有大量稳定能力没有被工程化。

---

# 14. 不要只记录最终正确结果

如果 AI 识别：

```text
Customer = ABC Limited
```

人工修改为：

```text
Customer = ABC Holdings Limited
```

不要只把数据库最终值改成正确答案。

至少保留：

```yaml
ai_output: ABC Limited
human_output: ABC Holdings Limited
corrected: true
reason: entity mismatch
```

这些差异是以后最有价值的生产训练信号之一。

---

# 15. 错误应该修在哪一层

每次失败都先分类。

| 问题 | 优先修复位置 |
|---|---|
| AI 看不懂文字/图片 | Prompt / Example / Model |
| 已经明确的业务规则没执行 | Rule / Program |
| 固定字段格式错 | Schema / Parser |
| 点击/页面路径错 | Tool / Script / Selector |
| 重复处理同一任务 | State / Dedup / Idempotency |
| 重启后恢复错 | State Machine / Persistent State |
| 配置漏了 | Preflight |
| API 超时无限重试 | Retry Policy / Circuit Breaker |
| 新的未知异常 | Human Escalation + New Case |

一个成熟项目通常会发现：

> 真正应该继续改 Prompt 的问题，只是所有生产问题中的一部分。

---

# 16. 适用范围

这套方法不依赖：

- 浏览器
- OCR
- Excel
- ERP
- CRM
- 某个模型
- 某个 Agent 框架

它适用于更抽象的结构：

```text
Source / Event
↓
Interpret / Transform（如果需要）
↓
Decision / Preparation
↓
Action（如果需要）
↓
Verification
↓
State / Result
```

项目最终运行形态可以完全不同：

- 纯读取/分析 + 人决策；
- Human-guided Copilot；
- 有边界的 Agent 自主；
- AI + 确定性 Workflow；
- 完全无 AI 的程序化系统；
- 一次性企业项目；
- 长期持续生产系统。

所以 Source 可以是：

- 文本
- 图片
- 邮件
- 电话转写
- PDF
- CAD
- 数据库
- 传感器
- 内部软件
- 网页
- 实体设备

Action 也可以是：

- 写文件
- 更新数据库
- 调 API
- 操作软件
- 生成文档
- 提交业务记录
- 通知人员
- 调度后续系统

具体实现不同，但训练顺序基本一致。

---

# 17. 安装后的推荐使用方式

如果仓库已被导入 Codex / Clawbot / Agent 工作区，可以直接说：

```text
使用当前仓库里的 Universal FDE Training。
我要训练一个新的真实业务。
按 FDE_Human-facing guide + Agent interaction reference.md 的 Start Prompt 启动，不要一次性读取整个 Skill。
```

如果系统支持从 GitHub 安装，可以让它：

1. 下载仓库。
2. 将 `FDE_TRAINING_SKILL.md` 设为可检索知识文件。
3. 打开 `FDE_Human-facing guide + Agent interaction reference.md`。
4. 使用 Start Prompt 开始。
5. 运行时只加载 Global Constitution、Project State、当前 Stage 和当前 Stage 声明的历史产物。

项目的设计目标就是让普通用户不需要先理解全部 FDE 理论，也能被 Agent 一步一步带着完成训练。

---

# 18. 你最终应该得到什么

一个成熟项目通常不是只得到“一个 Prompt”。

根据项目类型，你会逐渐得到其中需要的能力：

- 一套真实 Operational SOP / Workflow Map
- 一套操作/业务规则
- 必要的工具、Schema 和 Contract
- 一套 Error Boundary
- 一个由 Deterministic / Agentic / Human Segment 组成的 Runtime Workflow
- 与业务匹配的状态、持久化和审计
- Golden / Negative / Regression Cases
- 持续业务的 Incident Loop，或一次性项目的 Project Closeout

不是所有项目都必须产出数据库、自动提交、无人值守 Agent 或持续运行服务。真正的标准是：**目标运行模式被证明可行，并且每个不确定性和权限边界都有明确去处。**

---

# 19. 一句话理解整个项目

> **先让 Agent 基于真实业务事实学会任务，再把稳定且值得工程化的部分固化；保留必要的受约束智能与 Human Gate，让系统收敛到正确的运行形态，而不是强行收敛到“全自动”。**

这就是 Universal FDE Training 的核心。


