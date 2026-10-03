# Universal FDE Progressive Training Skill

> Version: 0.2-rc2  
> Purpose: 将一个陌生的真实 ToB / 企业业务流程，按受控顺序训练为可验证、可执行、可持续优化的运行体系；AI / Program / Human 按业务需要组合，不预设必须全自动化。  
> Intended users: Agent / Clawbot / Codex / Computer-Use Agent / Coding Agent / Tool-Using LLM  
> Runtime rule: **不要一次性把全文放入上下文。长期保留 Global Constitution + Project State；每次只检索当前 Stage 以及该 Stage 明确声明的 REQUIRED_ARTIFACTS，禁止预读未来 Stage。**

---

<a id="FDE_SKILL_RUNTIME"></a>
## 0. Runtime Contract

本文件不是普通教程，而是一个**顺序执行协议（Sequential Training Protocol）**。

任何使用本 Skill 的 Agent，必须遵循以下运行方式：

1. 初始化 `FDE_PROJECT_STATE`。
2. 读取并长期保留 `GLOBAL CONSTITUTION`。
3. 根据 `current_stage` 检索对应阶段，例如：`FDE_STAGE_05_FEASIBILITY`。
4. 同一时刻只允许读取：
   - `GLOBAL CONSTITUTION`
   - 当前 `FDE_PROJECT_STATE`
   - 当前 Stage
   - 当前 Stage 明确声明的 `REQUIRED_ARTIFACTS`
5. **禁止为了“提前规划”而读取未来 Stage。** 当前 Stage 如果需要过去产物，只读取所列产物，不重新加载整个 Skill。
6. 只执行当前阶段定义的动作，不预先执行后续阶段。
7. 当前阶段只有 `PASS` 或该 Stage 明确允许且证据充分的 `NOT_APPLICABLE` 才能离开；否则不得进入下一阶段。
8. 如果缺少信息，优先向人类询问真实生产事实，而不是自行补全。
9. 如果存在可验证的真实环境，应优先通过观察、测试、日志和结果证明，而不是“根据经验认为可行”。
10. 阶段结束时更新 `FDE_PROJECT_STATE`，记录事实、假设、未知、失败、产物、Gate Evidence 与下一动作。
11. 若生产事实推翻此前结论，必须回退到**最早产生错误假设或失效产物的 Stage**，修复后重新经过受影响的后续 Gate。
12. `NOT_APPLICABLE` 不是跳关工具。只有当前 Stage 明确允许 N/A，且理由和证据被记录时，才能以 N/A 离开该 Stage。

### Stage Result

每个 Stage 执行结束只能产生以下状态之一：

- `PASS`：Gate 全部满足，可以进入 `NEXT`。
- `BLOCKED`：缺少当前阶段必须的信息、权限、环境或证据；停留当前 Stage。
- `BACKTRACK`：当前发现推翻了过去结论；回到指定 Stage 修复。
- `NOT_APPLICABLE`：仅当该 Stage 明确允许 N/A，并且已记录不适用原因与证据。

### Stage 执行标准结构

每个 Stage 必须能够在**单独被检索**时正常执行，因此至少包含以下语义：

- `PURPOSE`：为什么存在。
- `ENTRY CONDITIONS`：上一阶段凭什么允许进入。
- `REQUIRED STATE`：进入时 Project State 中必须已经知道什么。
- `REQUIRED ARTIFACTS`：当前阶段允许按需读取哪些过去产物。
- `ACTIONS`：当前真正要做什么。
- `REQUIRED OUTPUT`：本阶段必须形成什么。
- `EVIDENCE`：用什么证明 Output 不是口头结论。
- `GATE`：哪些客观条件全部成立才能 `PASS`。
- `BACKTRACK`：什么情况必须退回哪一阶段。
- `NEXT`：通过后下一阶段。

如果 Gate 未通过：

1. 不得假装完成。
2. 将阻断原因写入 `open_questions` 或 `failures`。
3. 只做解决当前 Gate 所需的工作。
4. 若阻断来自先前错误理解，使用 `BACKTRACK`，不得在当前阶段打补丁掩盖过去错误。
5. 修复完成后，重新验证所有受影响的后续 Gate。

### 0.1 推荐检索键

每个阶段均有稳定检索键：

- `FDE_STAGE_00_INIT`
- `FDE_STAGE_01_DISCOVERY`
- `FDE_STAGE_02_MANUAL_WORKFLOW`
- `FDE_STAGE_03_DELEGATION_BOUNDARY`
- `FDE_STAGE_04_VERIFICATION_MODEL`
- `FDE_STAGE_05_FEASIBILITY`
- `FDE_STAGE_06_SAFE_PROTOTYPE`
- `FDE_STAGE_07_SHADOW_LEARNING`
- `FDE_STAGE_08_GUIDED_FIRST_RUN`
- `FDE_STAGE_09_GUARDED_AUTONOMY`
- `FDE_STAGE_10_ERROR_BOUNDARY`
- `FDE_STAGE_11_CAPABILITY_EXTRACTION`
- `FDE_STAGE_12_WORKFLOW_COMPILATION`
- `FDE_STAGE_13_STATE_DATA_AUDIT`
- `FDE_STAGE_14_PRODUCTION_HARDENING`
- `FDE_STAGE_15_AUTONOMY_EXPANSION`
- `FDE_STAGE_16_COST_COMPRESSION`
- `FDE_STAGE_17_CONTINUOUS_LEARNING`

---

<a id="FDE_GLOBAL_CONSTITUTION"></a>
# GLOBAL CONSTITUTION

以下规则在整个训练周期内持续生效。

## G1. 先理解业务，再自动化

第一次接触任务时，不直接写最终脚本、不直接设计完整系统、不根据一句需求就宣布“可以自动化”。

必须先还原真实生产流程和真实成功标准。

## G2. 事实、假设、未知、规则必须分离

所有项目知识至少分为四类：

- `FACT`：被人类确认或被真实系统证据证明。
- `ASSUMPTION`：暂时推断，尚未证明。
- `UNKNOWN`：当前不知道。
- `VALIDATED_RULE`：已经通过生产事实或测试验证，可以用于执行。

禁止将 `ASSUMPTION` 或 `UNKNOWN` 自动升级为 `VALIDATED_RULE`。

## G3. 学习权限与执行权限分离

Agent 可以较早观察真实生产，但不代表它可以较早改变生产状态。

权限等级可参考：

`READ → OBSERVE → SIMULATE → PREPARE → REVERSIBLE_WRITE → GUARDED_EXTERNAL_ACTION → CONTROLLED_AUTONOMY`

这是**权限上限阶梯，不是要求每个项目必须实际执行每一级动作**。如果业务不存在某类中间动作（例如没有可逆写入），不需要人为制造该动作；但授予更高权限前，必须满足该更高权限自己的验证、停止、授权和恢复前置，不能因为 Agent “已经理解”而直接放权。

## G4. 第一次真实探索默认 Fail-Closed

在未明确授权前，默认禁止执行会改变外部真实状态的动作，包括但不限于：

- Submit / Final Save
- Send / Publish
- Approve / Reject
- Delete
- Pay / Refund
- Change final status
- Issue irreversible command

到达这些动作前必须停下、显示拟执行内容、等待授权或由程序 Gate 决定。

## G5. 不可验证的“成功”不算成功

如果一个动作执行后，无法通过独立信号确认是否成功，则该动作尚未形成可靠自动化路径。

每个关键动作必须尽可能有：

`Action → Independent Verification → State Update`

## G6. AI 负责需要智能的不确定性，Program 负责适合确定化的部分，Human 负责必须由人完成的判断、授权与异常域

典型划分：

- AI：语义理解、非结构化解析、模糊分类、开放式判断、异常解释。
- Program：格式校验、计算、状态机、数据库、API、DOM、固定规则、去重、锁、重试、超时、日志、验证。
- Human：业务明确要求的人类判断/授权、规则确认、业务变更确认、未覆盖异常、无法证明正确的情况。

如果一个任务长期稳定、重复且可确定，应优先从 AI 下沉到 Program。

## G7. 稳定能力在值得工程化时应被“编译掉”

对重复出现、输入输出确定、无需语义推理，且从风险、成本、延迟或维护角度值得工程化的步骤，不应长期消耗模型推理。

优先转换为：

`API / Function / Script / Parser / Schema / Rule / State Machine / Selector / Deterministic Workflow`

对于一次性、极低频或确定化成本明显高于收益的步骤，可以继续保留 `AGENTIC_BOUNDED` 或 Human 处理，但必须记录原因。

## G8. 优先寻找更确定的控制面

当存在多个操作通道时，优先寻找：

1. 官方 API / SDK
2. CLI / Application Interface
3. Database / File Interface（仅在架构允许时）
4. DOM / Accessibility Tree / Structured UI
5. Scriptable application interface
6. RPA / Keyboard-Mouse automation
7. Vision-based interaction

不是固定排序，而是优先选择**更稳定、可验证、可维护、可回滚**的通道。

## G9. 每次失败先问“该判断是否还应该交给模型”

生产失败后，不默认通过“把 Prompt 写得更长”修复。

先分类：

- 语义理解问题 → Prompt / Example / Model
- 明确业务规则 → Rule
- 状态问题 → State Machine
- 页面操作问题 → Tool / Script / Selector
- 数据问题 → Schema / Database / Constraint
- 配置问题 → Preflight
- 重复执行问题 → Idempotency / Dedup / Lock
- 不可可靠处理 → Human Escalation

## G10. 优化不得削弱安全边界

性能、成本、延迟、Token 优化，不能删除必要的：

- Validation
- Logging
- Stop Condition
- Idempotency
- Auditability
- Human Escalation

## G11. 优先问真实案例，不优先问抽象需求

如果用户说“我们每天处理订单”，优先问：

> “拿一笔真实、普通、刚刚处理过的订单，从它出现开始，人实际做了什么？”

真实轨迹优先于抽象 SOP。

## G12. 不以“100% 自动化”为默认目标

目标是最大化**可证明正确的自动执行范围**，而不是最大化自动覆盖率。

## G13. 真实执行前必须存在最小执行状态

正式的数据库与状态架构可以在后续阶段完善，但从第一次真实执行开始，至少必须保留：

- `task_id`
- `source_ref`
- `execution_status`
- `current_step`
- `external_action_attempted`
- `external_action_result`
- `verification_result`
- `human_override`

这样即使会话、工具或系统中断，也不能仅靠 Agent 记忆判断“刚才到底执行了没有”。

## G14. 不预设最终运行形态

Universal FDE 不要求所有项目最终变成“无人值守自动化”。项目必须显式记录目标运行契约，至少包含三个维度：

- `lifecycle`: `RECURRING | FINITE`
- `execution_authority`: `READ_ONLY | HUMAN_GUIDED | GUARDED_AUTONOMY | BOUNDED_AUTONOMY`
- `orchestration_mode`: `DETERMINISTIC | HYBRID | AGENTIC_BOUNDED`

原则：

1. 稳定、重复、确定且值得工程化的部分应优先固化。
2. 本质上需要语义推理、探索或专家判断的部分可以继续由 Agent 承担，但必须有输入/输出契约、停止条件、验证和人工路线。
3. 永久 Human-in-the-loop 是合法终态，不得为了“自动化率”强行移除 Human Gate。
4. 完全不需要 AI 的最终系统也是合法终态。
5. 一次性/阶段性项目允许在验收与归档后结束，不强迫进入永久持续生产循环。

---

<a id="FDE_PROJECT_STATE"></a>
# 1. Persistent Project State

整个训练会话持续维护以下状态。可以保存为 YAML / JSON / Markdown，但字段语义应保持一致。

```yaml
FDE_PROJECT_STATE:
  project:
    name: null
    objective: null
    owner_goal: null
    baseline_type: null        # EXISTING | PARTIAL | GREENFIELD
    lifecycle: null            # RECURRING | FINITE
    execution_authority: null  # READ_ONLY | HUMAN_GUIDED | GUARDED_AUTONOMY | BOUNDED_AUTONOMY
    orchestration_mode: null   # DETERMINISTIC | HYBRID | AGENTIC_BOUNDED
    current_stage: FDE_STAGE_00_INIT
    stage_status: not_started
    stage_result: null
    stage_history: []

  business:
    users: []
    operators: []
    systems: []
    input_sources: []
    outputs: []
    external_effects: []
    normal_cases: []
    exception_cases: []

  knowledge:
    facts: []
    assumptions: []
    unknowns: []
    validated_rules: []
    rejected_rules: []

  workflow:
    manual_steps: []
    machine_steps: []
    branch_points: []
    irreversible_steps: []
    verification_points: []

  delegation:
    ai_tasks: []
    program_tasks: []
    human_tasks: []
    prohibited_autonomy: []

  capability:
    confirmed: []
    unconfirmed: []
    failed: []
    control_surfaces: []

  safety:
    risk_actions: []
    stop_conditions: []
    approval_points: []
    rollback_paths: []

  artifacts:
    manual_sop: null
    workflow_map: null
    feasibility_report: null
    prototype: null
    golden_trace: null
    rule_set: null
    schemas: []
    tool_library: []
    workflow: null
    state_machine: null
    data_model: null
    audit_model: null

  evidence:
    screenshots: []
    logs: []
    test_results: []
    user_confirmations: []
    production_observations: []

  execution:
    current_task_id: null
    minimal_ledger: []
    minimum_error_boundary: []

  gate_evidence: []
  failures: []
  human_corrections: []
  open_questions: []
  next_required_action: null
```

### State Maintenance Rules

1. 每次阶段结束前更新状态。
2. `facts` 必须能追溯到用户确认或系统证据。
3. `validated_rules` 必须说明验证方式。
4. `open_questions` 为空不代表可以自动进入下一阶段；仍需检查 Gate。
5. 任何生产事故都进入 `failures`，不能只在聊天里解释后遗忘。
6. 人工纠正必须保存“AI 原输出”和“最终正确输出”的差异。
7. 每次 Stage 结束必须记录 `stage_result` 与支持 Gate 判断的 `gate_evidence`。
8. 一旦进入真实执行阶段，`execution.minimal_ledger` 必须持续更新，不能只依赖聊天上下文记住任务状态。

---

<a id="FDE_STAGE_00_INIT"></a>
# FDE_STAGE_00_INIT — Mission Initialization

## PURPOSE

把模糊的“帮我理解/辅助/改造/自动化这个业务”转换为一个可训练的 FDE 项目，并阻止 Agent 在不了解任务时直接开始开发。

## ENTRY CONDITIONS

- 用户提出一个真实企业业务目标，希望用 Agent 进行理解、辅助、改造、工具化或自动化。

## REQUIRED STATE

- 无前置项目状态；本阶段负责初始化。

## REQUIRED ARTIFACTS

- 无。

## REQUIRED INPUT

至少需要一个业务目标。其他信息可以未知。

## ACTIONS

1. 创建 `FDE_PROJECT_STATE`。
2. 用一句话复述业务目标，避免直接提出技术方案。
3. 标记当前已知输入、输出、系统、人员、外部动作。
4. 初步标记业务基线类型：`EXISTING / PARTIAL / GREENFIELD`。
5. 初步记录目标生命周期与权限方向：`RECURRING / FINITE`，以及当前期望是只读辅助、人工带做还是有限自主；这些只是初始目标，Stage 03 再正式确认。
6. 做一次**粗粒度早期风险扫描**：是否明显涉及不可逆动作、真实资金、真实外部发送、审批、删除、状态改变或实体设备控制。
7. 如果存在明显高影响动作，只记录并限制权限；此阶段不做完整风险设计。
8. 进入 Production Discovery。

## QUESTIONS

优先只问能启动真实发现的问题：

- 这项工作最终想达成什么结果？
- 谁现在在做这项工作？
- 能否拿一笔普通真实案例，从开始到结束演示？
- 这项工作主要在哪些系统、文件、设备或沟通渠道中发生？
- 这是持续重复的业务，还是一次性/阶段性项目？
- 你期望 AI 最终只是辅助人，还是允许它在某个明确边界内独立推进？

## PROHIBITED

- 不写最终架构。
- 不直接承诺“完全自动化”。
- 不因为用户提到某个技术栈就锁定技术方案。

## REQUIRED OUTPUT

- 项目一句话目标
- 第一版系统/人员/输入/输出清单
- 第一版高影响动作提示
- `baseline_type / lifecycle / execution_authority` 初始记录
- 下一阶段需要观察的真实案例、参考轨迹或真实业务输入

## GATE

满足以下条件即可进入 Stage 01：

- 已定义业务目标。
- 已知道至少一个现实操作人员、现实操作来源、现有系统流程或真实业务输入来源。
- 已能取得至少一个真实案例、参考轨迹、真实业务输入，或足以描述“触发条件 → 主要动作/判断 → 目标结果”的具体业务事件。

如果既无法取得真实案例/输入，也无法取得现实流程、约束或周边系统事实，停留本阶段并明确说明：当前只能做概念设计，不能进入真实 FDE 训练。

## EVIDENCE

- 用户明确的业务目标，或真实业务材料中可直接确认的目标。
- 至少一个可用于 Stage 01 的现实案例来源或具体业务轨迹来源。

## BACKTRACK

- 本阶段是起点，无更早 Stage。
- 如果业务目标本身发生改变，重新初始化本阶段并创建新的目标版本。

## NEXT

`FDE_STAGE_01_DISCOVERY`

---

<a id="FDE_STAGE_01_DISCOVERY"></a>
# FDE_STAGE_01_DISCOVERY — Production Discovery

## PURPOSE

理解真实业务基线，而不是只理解用户对业务的概括描述。既支持已有人工/系统流程，也支持部分已有流程和 Greenfield 新流程。

## ENTRY CONDITIONS

Stage 00 Gate 已通过。

## REQUIRED STATE

- `project.objective`
- `project.baseline_type` 已有初始值。
- 至少一个现实操作人员、现有系统流程、参考轨迹或真实业务输入来源。

## REQUIRED ARTIFACTS

- Stage 00 的项目目标与第一版系统/人员/输入/输出清单。

## OBJECTIVE

回答：

> 对已有流程：一笔普通任务现实中如何发生？
> 对 Greenfield：真实输入、业务约束、周边系统和人类决策目前是什么，哪些步骤只是待设计候选？

## ACTIONS

1. 如果 `baseline_type=EXISTING/PARTIAL`，优先选一笔“正常、常见、刚做过”的真实案例或现有系统轨迹。
2. 如果 `baseline_type=GREENFIELD`，选择至少一个真实业务输入/事件，观察其周边现有流程、约束、决策者、Source of Truth / Authoritative Source Policy 与目标结果；尚不存在的步骤必须标记 `DESIGN_CANDIDATE`，不得伪装成 FACT。
3. 从源头开始逐步追问，直到现实结果或目标结果产生。
4. 记录每一步的：
   - 人/系统看到了什么
   - 人/系统判断什么
   - 人/系统执行什么
   - 状态发生什么变化
   - 下一步由什么信号决定
   - 该步骤是 `OBSERVED` 还是 `DESIGN_CANDIDATE`
5. 收集已知异常；如果没有真实异常样本，记录为 `UNKNOWN`，并把合理的失败候选标成 `ASSUMPTION`，不得声称已发生。
6. 区分：机械动作、业务判断、经验判断、系统规则、外部沟通、状态变化。
7. 记录用户说不清或“凭经验”的部分为 `UNKNOWN`，不要自行整理成规则。

## HUMAN INTERVIEW STRATEGY

不要一次发送几十个问题。

每轮聚焦一个局部，并根据回答追问。

优先使用：

- “这一步你实际看哪里？”
- “什么情况下你不会这样做？”
- “你怎么知道这一步成功了？”
- “如果信息不完整，你平时怎么处理？”
- “有没有例外？”
- “如果新人来做，他最容易错在哪？”

## EVIDENCE PRIORITY

优先级：

1. 真实屏幕 / 文件 / 系统记录 / 操作日志
2. 人现场演示
3. 真实案例复盘
4. 书面 SOP
5. 抽象口述

## REQUIRED OUTPUT

- `baseline_trace_v0`（已有流程为真实生产轨迹；Greenfield 为真实输入/约束 + 设计候选轨迹）
- 人工/系统角色清单
- 系统清单
- 输入/输出清单
- 正常案例轨迹
- 异常案例列表
- Unknown 列表

## PROHIBITED

- 不自动把“通常”写成“必须”。
- 不使用历史记忆补全当前业务。
- 不开始开发完整自动化。

## GATE

进入 Stage 02 前必须满足：

- `EXISTING/PARTIAL`：至少有一条可追溯的现实任务/系统轨迹，并覆盖触发条件、关键动作/判断和现实结束结果；缺失部分已显式列为 Unknown。
- `GREENFIELD`：至少有一个真实输入/事件、明确目标结果和真实业务约束；所有尚未存在的步骤均标记为 `DESIGN_CANDIDATE`。
- 已知道触发条件和目标结果。
- 已知道主要参与系统/人。
- 已知异常被记录；没有真实异常证据时，没有把假设伪装成事实。
- 关键 Unknown 已列出，而非被假设填平。

## EVIDENCE

- 已有流程项目：至少一条可追溯到真实演示、记录、文件或业务复盘的现实轨迹。
- Greenfield：至少一个真实业务输入/事件、现实约束与目标结果；设计步骤有明确 `DESIGN_CANDIDATE` 标记。
- 异常/失败场景来自真实事实，或被明确标记为待验证候选，不得伪装成已发生事实。

## BACKTRACK

- 如果无法获得任何真实输入、现实案例、现有流程、业务约束或周边系统事实，回到 `FDE_STAGE_00_INIT`，项目只能保留为概念设计。
- 如果业务目标在观察后被证明定义错误，回到 `FDE_STAGE_00_INIT` 重新定义目标。

## NEXT

`FDE_STAGE_02_MANUAL_WORKFLOW`

---

<a id="FDE_STAGE_02_MANUAL_WORKFLOW"></a>
# FDE_STAGE_02_MANUAL_WORKFLOW — Manual Workflow Reconstruction

## PURPOSE

把真实业务基线转换为可执行、可验证的操作指南。基线可以来自人工经验、现有系统流程，或经明确标记的 Greenfield 设计候选。

## ENTRY CONDITIONS

- `FDE_STAGE_01_DISCOVERY` 为 `PASS`。

## REQUIRED STATE

- 已有 `baseline_trace_v0`。
- 主要系统、人员、触发条件、目标结果与关键 Unknown 已登记。

## REQUIRED ARTIFACTS

- `baseline_trace_v0`
- 正常案例/设计候选轨迹
- 异常案例列表
- Unknown 列表

## OBJECTIVE

形成第一版 `Manual SOP` 与 workflow graph。

## ACTIONS

将业务工作拆为原子动作。每一步必须至少包含：

```yaml
step:
  id:
  name:
  input:
  action:
  decision:
  output:
  success_signal:
  failure_signal:
  next:
  actor:
  evidence_status: OBSERVED | DESIGN_CANDIDATE | VALIDATED
  notes:
```

拆分标准：

- 一步中若同时包含多个不同目标，则继续拆。
- 如果“成功”无法客观描述，则继续追问。
- 如果人工说“看情况”，必须拆出其判断依据或保留为 Unknown。
- 如果一个动作产生真实外部状态变化，单独成步。

## REQUIRED ANALYSIS

标注：

- 纯机械动作
- 需要语义理解的动作
- 需要业务规则的动作
- 需要历史/状态的动作
- 需要人工授权的动作
- 需要外部系统反馈的动作

## REQUIRED OUTPUT

- `manual_SOP_v1`
- `workflow_graph_v1`
- `branch_list_v1`
- `unknown_decisions_v1`

## EVIDENCE

- `EXISTING/PARTIAL`：至少用 Stage 01 的真实普通案例逐步回放一次 `manual_SOP_v1`；所有实际动作都能映射到某个 SOP step。
- `GREENFIELD`：至少用一个真实业务输入/事件走查 `manual_SOP_v1`，确认真实约束、Source of Truth / Authoritative Source Policy 与目标结果均有落点；尚未在现实中执行的步骤继续保持 `DESIGN_CANDIDATE`，不得标成 VALIDATED。
- 无法映射或无法解释的内容必须进入 Unknown 或返回补充发现。

## GATE

- `EXISTING/PARTIAL`：普通案例可按 SOP 从触发条件走到现实结束结果。
- `GREENFIELD`：设计候选路径能从真实触发条件走到目标结果，且每个未验证步骤都明确标记为 `DESIGN_CANDIDATE`。
- 每个关键步骤都有明确输入、动作、输出与下一步。
- 每个关键状态变化被单独标出。
- 仍无法描述的判断被明确列出，且没有被假设填平。
- 不存在已观察到、但 SOP 中没有位置的关键动作。

## BACKTRACK

- 如果无法解释真实轨迹中的关键动作，回到 `FDE_STAGE_01_DISCOVERY` 补充生产发现。
- 如果发现业务目标或结束结果定义错误，回到 `FDE_STAGE_00_INIT`。

## NEXT

`FDE_STAGE_03_DELEGATION_BOUNDARY`

---

<a id="FDE_STAGE_03_DELEGATION_BOUNDARY"></a>
# FDE_STAGE_03_DELEGATION_BOUNDARY — Risk & Delegation Boundary

## PURPOSE

确定每一步可以委托给谁、以多大权限执行，以及何时必须停止。

## ENTRY CONDITIONS

- `FDE_STAGE_02_MANUAL_WORKFLOW` 为 `PASS`。

## REQUIRED STATE

- 已有可从头走到尾的 `manual_SOP_v1`。
- 外部状态变化步骤已在 SOP 中单独标出。

## REQUIRED ARTIFACTS

- `manual_SOP_v1`
- `workflow_graph_v1`
- `branch_list_v1`
- `unknown_decisions_v1`

## CORE QUESTION

不要问：

> “AI 能不能担责？”

要问：

> “这个动作能否委托给 AI？如果判断错误，能否在产生不可接受后果前被发现、阻断、回滚或转人工？”

## ACTION CLASSIFICATION

建议为每个动作标记：

- `A0_READ`：只读、观察、检索、解析。
- `A1_PREPARE`：生成草稿、参数、候选动作，不改变真实状态。
- `A2_REVERSIBLE_WRITE`：会写入，但可可靠撤回或修正。
- `A3_EXTERNAL_ACTION`：发送、提交、改变真实业务状态。
- `A4_CRITICAL_ACTION`：高影响、不可逆或错误难以自动发现。

## ACTIONS

1. 为 Manual SOP 中每个关键动作标记风险等级。
2. 为每个动作指定默认执行者：AI / Program / Human。
3. 标记需要人工批准、程序 Gate 或必须停止的动作。
4. 对高风险动作执行以下 Delegation Test。
5. 基于业务目标与证据，正式确认 `lifecycle / execution_authority / orchestration_mode`，写回 `FDE_PROJECT_STATE.project` 并生成 `target_runtime_contract_v1`。不要默认目标是全自动。

## DELEGATION TEST

对每一步回答：

1. 输入是否明确？
2. 规则是否明确？
3. 输出是否可校验？
4. 错误是否可检测？
5. 是否可回滚？
6. 是否会影响外部真实对象？
7. 是否需要审批？
8. 是否存在未知异常域？

## REQUIRED OUTPUT

生成：

- `action_risk_map`
- `ai_scope_v1`
- `program_scope_v1`
- `human_scope_v1`
- `approval_points_v1`
- `stop_conditions_v1`
- `target_runtime_contract_v1`

## PROHIBITED

- 不因为技术上“可以点”就允许自动执行。
- 不因为动作可逆就省略日志。
- 不把异常域默认交给 AI 自由处理。

## GATE

- 所有外部状态变化动作已识别。
- 每个关键步骤已指定默认执行者。
- 所有 A3/A4 动作有明确批准/停止策略。
- 当前允许的最大自主权限已写清楚。
- `target_runtime_contract_v1` 已明确项目是持续还是有限期、目标执行权限和编排模式。

## EVIDENCE

- `action_risk_map` 中每个关键 SOP step 都有风险等级与默认执行者。
- 每个 A3/A4 动作能追溯到明确批准、程序 Gate 或停止策略。

## BACKTRACK

- 如果某个动作无法判断其真实副作用，回到 `FDE_STAGE_01_DISCOVERY` 或 `FDE_STAGE_02_MANUAL_WORKFLOW` 补足事实。
- 如果风险边界要求改变 SOP 拆分方式，回到 `FDE_STAGE_02_MANUAL_WORKFLOW`。

## NEXT

`FDE_STAGE_04_VERIFICATION_MODEL`

---

<a id="FDE_STAGE_04_VERIFICATION_MODEL"></a>
# FDE_STAGE_04_VERIFICATION_MODEL — Success, Failure & Verification Model

## PURPOSE

在技术实现之前先定义“**需要怎样证明系统做对了**”。本阶段设计验证要求，不假装已经证明技术上可执行；验证通道是否真实可用由 Stage 05 证明。

## ENTRY CONDITIONS

- `FDE_STAGE_03_DELEGATION_BOUNDARY` 为 `PASS`。

## REQUIRED STATE

- 每个关键动作已有风险等级、默认执行者和当前最大自主权限。
- 所有外部状态变化动作已经识别。

## REQUIRED ARTIFACTS

- `manual_SOP_v1`
- `action_risk_map`
- `approval_points_v1`
- `stop_conditions_v1`

## OBJECTIVE

为每个关键步骤建立验证模型。

## ACTIONS

对关键步骤定义：

```yaml
verification:
  preconditions: []
  expected_action: null
  expected_state_change: null
  positive_signals: []
  negative_signals: []
  candidate_independent_check: null
  timeout: null
  uncertain_behavior: STOP
  proof_status: DESIGN_ONLY
```

## PRINCIPLES

- “没有报错”不是成功证据。
- “页面看起来差不多”不是高质量验证。
- 提交之后，优先重新读取目标系统状态确认。
- 如果动作和验证使用同一个不可靠信号，应寻找第二信号。
- 关键写操作应尽量具备 pre-check + post-check。

## REQUIRED OUTPUT

- `success_definition`
- `failure_definition`
- `verification_matrix`
- `uncertain_state_policy`

## EVIDENCE

- 每个关键动作的成功/失败信号均能追溯到 SOP 中的预期状态变化。
- 对尚未证明技术可行的验证方式明确标记为 `DESIGN_ONLY`，不得写成“已验证”。

## GATE

- 整体任务存在明确“完成”定义。
- 每个关键写操作都定义了至少一个候选后验证信号，或明确标记为必须人工确认。
- 每个关键写操作都定义了“不确定时怎么办”。
- 验证设计没有依赖尚未被确认存在的技术能力而伪装成事实。

## BACKTRACK

- 如果无法定义一个动作应产生什么状态变化，回到 `FDE_STAGE_02_MANUAL_WORKFLOW`。
- 如果验证要求暴露出风险等级或授权边界错误，回到 `FDE_STAGE_03_DELEGATION_BOUNDARY`。

## NEXT

`FDE_STAGE_05_FEASIBILITY`

---

<a id="FDE_STAGE_05_FEASIBILITY"></a>
# FDE_STAGE_05_FEASIBILITY — Technical Feasibility & Control Surface Mapping

## PURPOSE

用真实测试回答“系统到底能不能被观察、控制和验证”，并把 Stage 04 的验证设计从 `DESIGN_ONLY` 升级为 `PROVEN`、`MANUAL_ONLY` 或 `BLOCKED`。

## ENTRY CONDITIONS

- `FDE_STAGE_04_VERIFICATION_MODEL` 为 `PASS`。

## REQUIRED STATE

- 已定义任务成功/失败、不确定状态策略与关键动作的候选验证信号。

## REQUIRED ARTIFACTS

- `verification_matrix`
- `success_definition`
- `failure_definition`
- `uncertain_state_policy`
- `action_risk_map`

## THREE PROOFS

必须分别证明：

1. `OBSERVABILITY`：是否能读到完成任务所需的信息？
2. `CONTROLLABILITY`：是否能可靠执行需要的动作？
3. `VERIFIABILITY`：执行后是否能确认结果？

## ACTIONS

针对每个系统寻找控制面：

- API / SDK
- CLI
- 文件接口
- 数据库接口
- DOM / Accessibility Tree
- 应用脚本接口
- RPA
- Vision UI
- 人工接力

对每个候选通道进行最小测试，并记录：

```yaml
control_surface:
  name:
  type:
  can_read:
  can_write:
  can_verify:
  auth_requirements:
  stability:
  latency:
  major_failure_modes:
  evidence:
  verification_proof:
  proof_status: PROVEN | MANUAL_ONLY | BLOCKED
```

## IMPORTANT

本阶段只验证能力，不追求完整闭环。

可以做：

- 打开页面
- 查一条测试记录
- 读取一个字段
- 写入测试环境字段
- 调用测试 API
- 导出测试文件

但默认不做真实不可逆生产动作。

## REQUIRED OUTPUT

- `feasibility_report`
- `capability_map`
- `control_surface_map`
- `technical_blockers`

## EVIDENCE

- 每个宣称可用的控制面都有一次最小真实测试、测试结果与证据引用。
- Stage 04 中用于关键写操作的验证方式均被标记为 `PROVEN`、`MANUAL_ONLY` 或 `BLOCKED`。

## GATE

进入 Stage 06 前必须满足：

- 核心输入存在已证明可用的读取路径。
- 至少存在一条已证明可用的控制路径，或该业务的目标本身不需要机器执行动作。
- 每个关键结果存在 `PROVEN` 的验证路径，或被明确设为 `MANUAL_ONLY`。
- 所有 `BLOCKED` 能力均已登记，且不会被后续 Demo 当成已可用能力。
- 主要技术阻断点已知。

若核心任务不可观察、不可控制且无法由人工接力、或关键结果既不可验证也无法人工确认，应在本阶段 `BLOCKED`，不得进入 Stage 06。

## BACKTRACK

- 如果真实测试证明 Stage 04 的验证模型建立在错误状态假设上，回到 `FDE_STAGE_04_VERIFICATION_MODEL`。
- 如果控制面测试暴露出 SOP 实际不可执行或动作副作用理解错误，回到 Stage 02 或 Stage 03。

## NEXT

`FDE_STAGE_06_SAFE_PROTOTYPE`

---

<a id="FDE_STAGE_06_SAFE_PROTOTYPE"></a>
# FDE_STAGE_06_SAFE_PROTOTYPE — Minimal Safe Prototype

## PURPOSE

做第一个最小闭环 Demo，用最少结构证明端到端路径，而不是直接搭生产系统。

## ENTRY CONDITIONS

- `FDE_STAGE_05_FEASIBILITY` 为 `PASS`。

## REQUIRED STATE

- 核心读取、控制与验证能力已被真实测试。
- `BLOCKED` 能力不会被原型路径依赖。

## REQUIRED ARTIFACTS

- `feasibility_report`
- `capability_map`
- `control_surface_map`
- `verification_matrix`

## RULE

`Prototype for learning, not for scale.`

## ACTIONS

1. 选一条最普通、最短的正常路径。
2. 优先使用测试数据、模拟数据或可回滚环境；如果业务无法提供测试环境，则使用 dry-run、历史 replay、数字孪生、人工接力或其他不产生未授权副作用的等价方式。
3. 允许临时、笨重、成本高的方式，只要可观察、可记录。
4. 尽量跑通：

`Input / Event → Interpret or Transform (if needed) → Decide / Prepare → Action (if any) → Verify → Result`

5. 若最后一步为高影响动作，则停在提交前。
6. 记录每个失败点和人工介入点。

## REQUIRED OUTPUT

- `prototype_v1`
- `prototype_trace`
- `failure_notes`
- `capability_gaps`

## PROHIBITED

- 不提前优化成本。
- 不因为 Demo 成功就宣布可生产。
- 不隐藏人工介入。

## GATE

- 至少一条普通案例路径可以走到安全终点。
- 所有人工介入点已记录。
- 已确认哪些步骤是“理论可行”与“真实已验证”。

## EVIDENCE

- `prototype_trace` 能从 Input 一直追溯到安全终点。
- 每个实际人工介入点、失败点和未验证能力都有记录。

## BACKTRACK

- 如果某个控制面并不可靠，回到 `FDE_STAGE_05_FEASIBILITY`。
- 如果原型暴露出流程本身理解错误，回到 Stage 01/02；如果暴露出授权边界错误，回到 Stage 03。

## NEXT

`FDE_STAGE_07_SHADOW_LEARNING`

---

<a id="FDE_STAGE_07_SHADOW_LEARNING"></a>
# FDE_STAGE_07_SHADOW_LEARNING — Production Shadow Learning

## PURPOSE

用真实生产事实校准 SOP，但默认不改变真实状态。优先 Live Shadow；无法安全/及时现场观察时，可使用可审计的历史回放、录屏、日志、事件记录或专家逐步复盘。

## ENTRY CONDITIONS

- `FDE_STAGE_06_SAFE_PROTOTYPE` 为 `PASS`。

## REQUIRED STATE

- 已有最小闭环原型。
- 已知当前禁止的真实外部动作与人工接力点。

## REQUIRED ARTIFACTS

- `prototype_trace`
- `manual_SOP_v1`
- `verification_matrix`
- `action_risk_map`

## MODE

`OBSERVE OR REPLAY FIRST / NO UNAUTHORIZED FINAL ACTION`

## ACTIONS

1. 选择观察模式：`LIVE_SHADOW | CONTROLLED_PILOT | REPLAY | AUDITED_TRACE | EXPERT_WALKTHROUGH`。
2. 选取真实生产任务、受控试点中的真实业务输入、历史真实任务或可审计业务轨迹。
3. Agent 按 SOP 预测下一步，但不执行未授权危险动作。
4. 对照真实员工操作、受控试点结果、系统结果、日志或审计轨迹。
5. 每发现偏差，记录：

```yaml
observation:
  expected:
  observed:
  difference:
  cause_guess:
  confidence:
  confirmed_rule: false
  needs_human_confirmation: true
```

6. 更新 SOP、规则候选和异常清单。
7. 重点寻找：
   - 隐藏状态
   - 口头未提及规则
   - UI/系统例外
   - 实际业务先后顺序
   - 新人容易犯错的位置

## REQUIRED OUTPUT

- `production_observation_log`（包含 `observation_mode`）
- `manual_SOP_v2`
- `rule_candidates_v2`
- `exception_map_v2`

## EVIDENCE

- 每个观察到的偏差都有 `expected / observed / difference / resolution_status`。
- 所有确认后的变化已经写回 SOP、规则候选或异常路线。

## GATE

- 已验证的普通业务 critical path 中，每一步都能映射到 `manual_SOP_v2`。
- 所有已观察到的结构性偏差均已解决，或被登记为明确的 exception / human route。
- 不存在会阻断普通路径、但既没有解释也没有人工路线的 `UNKNOWN`。
- 尚未理解的生产例外全部有明确 `STOP` 或 Human escalation 路线。

## BACKTRACK

- 如果真实生产顺序与 SOP 冲突，回到 `FDE_STAGE_02_MANUAL_WORKFLOW` 修订并重新验证受影响阶段。
- 如果发现新的外部副作用或权限需求，回到 Stage 03。
- 如果发现原验证设计无效，回到 Stage 04/05。

## NEXT

`FDE_STAGE_08_GUIDED_FIRST_RUN`

---

<a id="FDE_STAGE_08_GUIDED_FIRST_RUN"></a>
# FDE_STAGE_08_GUIDED_FIRST_RUN — Human-Guided First Real Execution

## PURPOSE

在人类监督下完成第一笔真实任务，建立第一条 Golden Execution Trace。**在任何真实写操作开始前，先建立最小执行状态与最低错误边界。**

## ENTRY CONDITIONS

- `FDE_STAGE_07_SHADOW_LEARNING` 为 `PASS`。

## REQUIRED STATE

- 普通生产 critical path 已能映射到 SOP。
- 高风险动作、批准点与验证方式已经定义并完成可行性证明。

## REQUIRED ARTIFACTS

- `manual_SOP_v2`
- `action_risk_map`
- `approval_points_v1`
- `verification_matrix`
- `production_observation_log`

## MODE

`AI proposes → Human supervises → Tool executes safe step → Verify → Continue`

## ACTIONS

1. 在执行任何真实写操作前创建 `minimal_execution_ledger`，至少记录：
   - `task_id`
   - `source_ref`
   - `execution_status`
   - `current_step`
   - `external_action_attempted`
   - `external_action_result`
   - `verification_result`
   - `human_override`
2. 建立 `minimum_error_boundary_v0`：
   - `UNKNOWN` 影响关键字段或关键动作 → `STOP`
   - 关键来源冲突 → `STOP`
   - 验证失败或结果不确定 → `STOP`
   - 到达权限边界 → `STOP / REQUEST APPROVAL`
   - 任何自动 Retry 必须有明确上限
3. 选择一笔普通、低异常概率的真实任务。
4. 每一步先说明：
   - 当前状态
   - 准备做什么
   - 为什么
   - 预期结果
   - 验证方式
5. 在允许范围内执行；每次状态变化同步写入 `minimal_execution_ledger`。
6. 遇到阻断时不绕过，先定位原因。
7. 人工修正必须记录差异。
8. 完成后回放整条轨迹，并确认 ledger 与真实系统最终状态一致。

## FAILURE ANALYSIS

失败优先分类为：

- Understanding
- Rule
- Tool
- UI
- State
- Data
- Permission
- Verification
- Environment
- Unknown

## REQUIRED OUTPUT

- `minimal_execution_ledger`
- `minimum_error_boundary_v0`
- `golden_execution_trace_001`
- `human_corrections_001`
- `failure_classification_001`
- `SOP_v3`

## EVIDENCE

- Golden Trace、ledger 和目标系统最终状态三者能够互相对应。
- 所有真实外部动作均有执行前状态、动作结果和执行后验证记录。

## GATE

- `minimum_error_boundary_v0` 已建立并实际生效。
- `minimal_execution_ledger` 已覆盖整笔任务。
- 至少完成一次真实任务。
- 真实结果被独立验证或按既定策略由人确认。
- 所有人工干预都有记录。
- 不存在“执行结果未知但继续向后运行”的步骤。
- 尚未解决问题未被隐藏。

## BACKTRACK

- 如果真实执行暴露流程理解错误，回到 Stage 01/02。
- 如果暴露授权或风险边界错误，回到 Stage 03。
- 如果暴露验证不可行，回到 Stage 04/05。
- 如果控制面不可靠，回到 Stage 05/06。

## NEXT

`FDE_STAGE_09_GUARDED_AUTONOMY`

---

<a id="FDE_STAGE_09_GUARDED_AUTONOMY"></a>
# FDE_STAGE_09_GUARDED_AUTONOMY — First Guarded Autonomous Trial

## PURPOSE

在**最低错误边界和最小执行状态已经存在**的前提下，验证 Agent 能否独立维护完整任务状态，而不是验证它能否“点完所有按钮”。

## ENTRY CONDITIONS

- `FDE_STAGE_08_GUIDED_FIRST_RUN` 为 `PASS`。
- `minimum_error_boundary_v0` 已建立。
- `minimal_execution_ledger` 已在真实任务中使用并与真实状态对齐。

## REQUIRED STATE

- 已有一条真实成功 Golden Trace。
- 当前最大自主权限没有因 Stage 08 自动扩大。

## REQUIRED ARTIFACTS

- `golden_execution_trace_001`
- `minimum_error_boundary_v0`
- `minimal_execution_ledger`
- `SOP_v3`
- `verification_matrix`
- `action_risk_map`
- `target_runtime_contract_v1`

## DEFAULT AUTHORITY

允许：

- 读取
- 解析
- 导航
- 准备参数
- 执行已验证的低风险动作
- 执行明确可逆写入

默认禁止：

- 未授权最终提交
- 未授权外部发送
- 未授权审批/拒绝
- 未授权删除
- 未授权不可逆状态改变

## ACTIONS

1. 选择新的普通真实案例。
2. 人类不逐步指导，Agent 自主推进。
3. 每个关键状态点执行程序验证。
4. 到达权限边界必须暂停。
5. 记录自主决策、工具调用、验证结果。

## REQUIRED OUTPUT

- `autonomous_trial_001`
- `decision_log`
- `stop_reason_log`
- `state_tracking_report`

## GATE

- Agent 能在无逐步指导下到达预定安全终点。
- 未发生越权。
- 状态追踪与实际系统一致。
- 关键错误会停止，而不是猜测继续。

## EVIDENCE

- `decision_log` 能解释所有关键自主推进点。
- `state_tracking_report` 与真实系统状态一致。
- 权限边界触发记录表明 Agent 会停，而不是自行扩大权限。

## NOT_APPLICABLE

如果 `target_runtime_contract_v1.execution_authority` 明确为 `READ_ONLY` 或 `HUMAN_GUIDED`，并且目标运行模式要求人在每个物质性决策/动作前持续介入，则“无逐步指导的独立推进”可以标记 `NOT_APPLICABLE`。必须记录：

- 为什么独立推进不属于目标运行模式；
- 哪些 Human Gate 永久保留；
- Stage 10 将使用哪些 Guided Run / Golden Trace 作为错误边界训练依据。

将以上内容保存为 `stage09_na_rationale` 并写入 stage history。

不得仅因为 Agent 自主能力不足而使用 N/A。

## BACKTRACK

- 如果自主试跑出现流程理解错误，回到 Stage 02/07。
- 如果出现验证或控制面失败，回到 Stage 04/05。
- 如果最低错误边界不足以阻止已知危险路径，回到 Stage 08 补足最低边界后重试。

## NEXT

`FDE_STAGE_10_ERROR_BOUNDARY`

---

<a id="FDE_STAGE_10_ERROR_BOUNDARY"></a>
# FDE_STAGE_10_ERROR_BOUNDARY — Error Boundary & Negative Cases

## PURPOSE

在 Stage 08 的最低错误边界基础上，系统化训练“什么时候不能做”，把已知失败域扩展成可回归的 Negative Case Suite。

## ENTRY CONDITIONS

- `FDE_STAGE_09_GUARDED_AUTONOMY` 为 `PASS` 或合法 `NOT_APPLICABLE`。

## REQUIRED STATE

- 已有 Stage 08 的真实 Guided Run 与最低错误边界。
- 若 Stage 09 为 `PASS`，还应有真实 autonomous stop reason / decision log；若 Stage 09 为 N/A，则使用 Guided Run 中的停止、人工接管和验证证据。

## REQUIRED ARTIFACTS

- `golden_execution_trace_001`
- `minimum_error_boundary_v0`
- `failure_classification_001`
- `autonomous_trial_001 / decision_log / stop_reason_log`（Stage 09 为 PASS 时）
- `stage09_na_rationale` 与 Human Gate 记录（Stage 09 为 N/A 时）

## PRINCIPLE

可靠性不仅来自正确执行，也来自正确拒绝。

## ACTIONS

建立 Negative Case Set：

- 缺字段
- 冲突字段
- 模糊输入
- 不存在记录
- 重复记录
- 状态不一致
- 登录失效
- 页面变化
- 网络中断
- 已完成任务再次出现
- 验证失败
- 不支持类型
- 超出权限动作

对每个 Case 定义：

```yaml
negative_case:
  trigger:
  expected_behavior:
  stop_or_retry:
  max_retry:
  human_message:
  recovery:
```

## REQUIRED OUTPUT

- `error_boundary_v1`
- `negative_case_suite`
- `retry_policy`
- `escalation_policy`

## GATE

- 常见错误条件有明确停止/重试行为。
- 不允许无限重试。
- 不允许在关键数据冲突时猜测。
- 不确定状态默认 Fail-Closed。

## EVIDENCE

- 每个 Negative Case 均有可复现触发条件、预期停止/重试行为和恢复/人工路线。
- 至少对与当前业务相关的高概率/高影响 Negative Case 运行实际或模拟测试。

## BACKTRACK

- 如果某个错误源于过去 SOP、风险边界或验证模型错误，回到对应 Stage 修复根因。
- 不允许仅在 Stage 10 追加一句 Prompt 来掩盖前序设计问题。

## NEXT

`FDE_STAGE_11_CAPABILITY_EXTRACTION`

---

<a id="FDE_STAGE_11_CAPABILITY_EXTRACTION"></a>
# FDE_STAGE_11_CAPABILITY_EXTRACTION — Capability Extraction & Determinization

## PURPOSE

审查业务中重复、稳定、可确定化且值得工程化的能力，将其下沉为确定性组件；没有 AI 或没有值得下沉的候选时，也要通过证据说明。

## ENTRY CONDITIONS

- `FDE_STAGE_10_ERROR_BOUNDARY` 为 `PASS`。

## REQUIRED STATE

- 已有真实 Guided/Autonomous 执行轨迹与 Negative Case 行为。
- 已知哪些步骤仍依赖重复模型推理。

## REQUIRED ARTIFACTS

- `golden_execution_trace_001`
- `autonomous_trial_001 / decision_log`（Stage 09 为 PASS 时）
- `stage09_na_rationale` 与 Guided Run 证据（Stage 09 为 N/A 时）
- `error_boundary_v1`
- `negative_case_suite`
- `SOP_v3`

## ACTIONS

分析前几次执行中的重复推理：

- 是否每次重新找同一个按钮？
- 是否每次重新解析同一种固定格式？
- 是否每次重复做相同字段映射？
- 是否每次重新判断固定业务规则？
- 是否每次都用视觉处理一个可结构化读取的页面？

对每项候选能力判断是否可转换为：

- API
- Script
- Function
- Parser
- Selector
- Schema
- Rule
- Local cache
- Lookup table
- State transition

## TOOL CONTRACT

每个工具应尽量定义：

```yaml
tool:
  name:
  purpose:
  input_schema:
  output_schema:
  preconditions:
  side_effects:
  errors:
  verification:
  idempotent:
```

## REQUIRED OUTPUT

- `tool_library_v1`
- `rule_library_v1`
- `parser_library_v1`
- `page_or_system_map_v1`

## EVIDENCE

- 对已记录的重复推理逐项给出：`compile / keep_in_AI / human_only` 结论与原因。
- 新工具至少通过与其目标能力对应的正常/失败测试；如果没有值得确定化的候选能力，必须有“无候选/暂不值得工程化”的证据说明。

## GATE

- 所有已记录的重复推理候选均已被审查，没有遗漏为“以后再说”的未分类项。
- 被判定可确定化的能力已有替代组件，或有明确阻断原因。
- 对实际存在的新工具，输入输出可结构化且失败能够显式返回，不静默吞错。
- 如果 `tool_library_v1` 为空，必须证明不存在当前值得工程化的稳定重复能力，而不是漏检。

## BACKTRACK

- 如果发现某项所谓“稳定能力”其实依赖未建模业务语义，回到 Stage 02/07/10 补充模型。
- 如果工具化改变了副作用或授权边界，回到 Stage 03/04。

## NEXT

`FDE_STAGE_12_WORKFLOW_COMPILATION`

---

<a id="FDE_STAGE_12_WORKFLOW_COMPILATION"></a>
# FDE_STAGE_12_WORKFLOW_COMPILATION — Workflow Compilation

## PURPOSE

把已证明稳定的部分编译成确定性流程，同时为无法/不应确定化的 Agentic 或 Human 段建立明确契约。目标不是消灭 Agent，而是消灭**没有必要的自由度**。

## ENTRY CONDITIONS

- `FDE_STAGE_11_CAPABILITY_EXTRACTION` 为 `PASS`。

## REQUIRED STATE

- 稳定重复能力已经完成一次系统化审查。
- 可工具化/规则化能力已有确定性组件或明确阻断原因。

## REQUIRED ARTIFACTS

- `tool_library_v1`
- `rule_library_v1`
- `parser_library_v1`
- `page_or_system_map_v1`
- `error_boundary_v1`
- `verification_matrix`
- `target_runtime_contract_v1`

## BEFORE

`Agent decides A → tool A → Agent reads → decides B → tool B → ...`

## TARGET

通用目标是一个 **Bounded Runtime Workflow**：

`Input/Event → Contracted Segment(s) → Validation / Human Gate as needed → Verification → Result`

每个 Segment 可以是：

- `DETERMINISTIC`：程序、规则、脚本、API、状态机等；
- `AGENTIC_BOUNDED`：仍需要 AI 推理/探索，但有输入、输出、允许工具、停止条件、验证和升级路线；
- `HUMAN`：业务明确要求人判断/授权/执行。

其中：

- 输入可以本来就是结构化数据，不要求必须先经过 AI。
- AI 不是固定架构节点；完全不需要 AI 的流程合法。
- Human 也不是“失败兜底”才出现；如果目标运行契约规定 Human Gate，它可以是正常主路径的一部分。
- 只有稳定且值得工程化的部分才要求确定化；高度语义化、低频或探索性工作可以保留 `AGENTIC_BOUNDED`。

## ACTIONS

1. 把已验证的稳定步骤组合为 `DETERMINISTIC` Segment。
2. 对仍需 AI 的步骤建立 `AGENTIC_BOUNDED` Contract：输入、允许工具、输出 Schema、停止条件、最大尝试、验证与 Human escalation。
3. 对必须由人完成的步骤建立 Human Gate / Human Task Contract。
4. 显式定义 Segment 顺序、分支与允许状态转移。
5. 所有 Segment 之间使用结构化 Contract 传递状态；不得依赖隐式聊天记忆。
6. 加入 pre-check 与 post-check。
7. 测试正常路径、Agentic 停止路径和可恢复性。

## REQUIRED OUTPUT

- `workflow_v1`
- `workflow_schema`
- `state_transition_map_v1`
- `execution_contract`

## EVIDENCE

- 对确定性 Segment，固定输入可重复得到同一类结果。
- 对 Agentic Segment，输入/输出/工具权限/停止/验证/升级契约明确。
- 对 Human Segment，触发条件、所需上下文和回传结果明确。

## GATE

- 所有被判定为稳定且值得工程化的步骤已确定化，或有明确保留理由。
- 所有 `AGENTIC_BOUNDED` Segment 都有输入/输出、允许工具、停止、验证和 Human escalation 契约。
- 所有 Human Gate 都是显式状态，而不是隐式等待。
- Segment 之间通过结构化 Contract 传递状态。
- 每个关键 workflow 状态都有验证。
- 异常能够退出到明确位置，不会继续执行未知路径。
- 如果业务不需要 AI，Workflow 在无 AI 节点时仍完整成立；如果业务本质高度 Agentic，允许 Agentic Segment 保留在普通路径。

## BACKTRACK

- 如果某个固定顺序其实包含未建模语义判断，回到 Stage 11 或更早阶段。
- 如果 Workflow 的状态/验证设计无法可靠恢复，回到 Stage 04/10/11 修正。

## NEXT

`FDE_STAGE_13_STATE_DATA_AUDIT`

---

<a id="FDE_STAGE_13_STATE_DATA_AUDIT"></a>
# FDE_STAGE_13_STATE_DATA_AUDIT — State, Persistence & Audit Architecture

## PURPOSE

把“会运行的流程”变成“能够可靠知道自己做过什么、现在是什么状态、为什么失败”的运行体系。持久化介质可以是数据库、文件/日志、外部 Source of Truth、工作流引擎或最小 ledger，不强制必须建设数据库。

## ENTRY CONDITIONS

- `FDE_STAGE_12_WORKFLOW_COMPILATION` 为 `PASS`。

## REQUIRED STATE

- 已有可执行 Workflow 与明确状态转移。
- Stage 08 起已经存在 `minimal_execution_ledger`；本阶段不是第一次记状态，而是把它升级为长期架构。

## REQUIRED ARTIFACTS

- `workflow_v1`
- `state_transition_map_v1`
- `execution_contract`
- `minimal_execution_ledger`
- `human_corrections_001`
- `error_boundary_v1`

## NOTE

日志从 Stage 00 就开始存在；本阶段是把零散日志正式升级为与业务生命周期匹配的持久化与审计架构。

## ACTIONS

1. 确定任务标识策略，以及 `Source of Truth / Authoritative Source Policy`：可以是单一权威源，也可以是多来源优先级、冲突规则或明确的人类裁决者。
2. 选择与生命周期匹配的 persistence mode：`MINIMAL_LEDGER | FILE_LOG | DATABASE | EXTERNAL_SYSTEM | WORKFLOW_ENGINE | OTHER`。
3. 定义需要持久化的状态、日志、人工纠正和版本信息。
4. 明确重启、重复输入和远端/本地状态冲突时的行为；对于可安全从头重跑的有限期无副作用任务，可以把“安全重跑”作为恢复策略。
5. 将此前阶段的零散日志迁移或映射到正式状态/审计模型。

## MINIMUM DATA MODEL

至少考虑保存：

- Raw Input
- Parsed / Normalized Input
- AI Interpretation
- Confidence / Uncertainty（如果实际使用）
- Rule Version
- Prompt Version
- Workflow Version
- Tool Calls
- Pre-check Result
- Action Result
- Post-check Result
- Human Correction
- Failure Type
- Final Outcome
- Timestamp
- Correlation / Task ID

## STATE QUESTIONS

必须回答：

- 系统如何知道这条任务处理过？
- 系统重启后状态从哪里恢复？
- 是否存在单一 Source of Truth？如果不存在，权威来源优先级和冲突裁决策略是什么？
- 本地状态与远端/其他来源冲突怎么办？
- 如何区分“未执行”“执行中”“执行成功但未记录”“执行失败”？
- 如何防止旧任务重新进入队列？

## REQUIRED OUTPUT

- `data_model_v1`
- `state_model_v1`
- `audit_log_spec_v1`
- `human_correction_spec_v1`
- `persistence_mode`

## GATE

- 每条任务可追踪。
- 中断后能够恢复、安全停止或按已证明策略安全重跑。
- `Source of Truth / Authoritative Source Policy` 已定义；不存在单一真值时，优先级与裁决路线明确。
- 人工修正有差异记录。

## EVIDENCE

- 使用至少一条已有真实执行轨迹映射到新 data/state/audit model，确认没有关键状态丢失。
- 能明确回答“任务是否已执行、执行到哪、外部动作是否发生、验证结果是什么”。

## BACKTRACK

- 如果无法定义可靠 Task ID 或 Authoritative Source Policy，回到 Stage 01/02/04 补足业务与状态定义。
- 如果 Workflow 本身无法支持安全恢复，回到 Stage 12。

## NEXT

`FDE_STAGE_14_PRODUCTION_HARDENING`

---

<a id="FDE_STAGE_14_PRODUCTION_HARDENING"></a>
# FDE_STAGE_14_PRODUCTION_HARDENING — Production Hardening

## PURPOSE

主动攻击流程，寻找 Demo 阶段不会暴露的生产故障。

## ENTRY CONDITIONS

- `FDE_STAGE_13_STATE_DATA_AUDIT` 为 `PASS`。

## REQUIRED STATE

- 每条任务可追踪。
- Source of Truth、持久状态与恢复策略已定义。

## REQUIRED ARTIFACTS

- `workflow_v1`
- `state_model_v1`
- `data_model_v1`
- `audit_log_spec_v1`
- `error_boundary_v1`
- `negative_case_suite`

## ACTIONS

主动执行故障注入、恢复测试和重复执行测试，不以“正常路径跑通”代替生产验证。

## REQUIRED FAILURE TESTS

根据业务适配，但至少思考并测试：

- 重复执行
- 同一任务并发执行
- 系统重启
- 任务执行一半崩溃
- 网络超时
- 登录过期
- 页面/接口短暂异常
- 外部系统响应迟到
- 提交成功但本地超时
- 本地认为成功但远端失败
- 历史数据重新出现
- 配置遗漏
- 依赖服务不可用
- 数据格式变化

## PRODUCTION MECHANISMS

按需加入：

- Idempotency
- Deduplication
- Task Lock / Batch Lock
- Stop Line / Cursor / Watermark
- Retry Limit
- Timeout
- Circuit Breaker
- Watchdog
- Scheduler
- Preflight
- Postflight
- Health Check
- Rollback / Compensation
- Dead-letter Queue
- Manual Escalation Queue

## REQUIRED OUTPUT

- `production_failure_matrix`
- `hardening_controls`
- `preflight_v1`
- `recovery_runbook_v1`

## EVIDENCE

- 每个被选为 release-blocking 的故障场景都有测试结果、预期行为与实际行为。
- 对所有已识别的重复执行路径都有 idempotency / dedup / lock / stop 或明确人工阻断之一。

## GATE

- 当前业务的关键重复/中断/恢复场景均已测试或明确标记无法模拟并给出替代验证。
- 不存在无限重试。
- 已识别的重复提交路径均有阻断机制，并通过对应测试。
- 关键配置缺失会在执行前暴露。
- 失败不会静默变成成功。
- `production_failure_matrix` 中没有未处置的 release-blocking 项。

## BACKTRACK

- 如果故障暴露状态模型错误，回到 Stage 13。
- 如果故障来自 Workflow 结构，回到 Stage 12。
- 如果故障来自错误边界或验证设计，回到 Stage 10 或 Stage 04/05。

## NEXT

`FDE_STAGE_15_AUTONOMY_EXPANSION`

---

<a id="FDE_STAGE_15_AUTONOMY_EXPANSION"></a>
# FDE_STAGE_15_AUTONOMY_EXPANSION — Controlled Autonomy Expansion

## PURPOSE

只在证据充分时扩大自主权限。

## ENTRY CONDITIONS

- `FDE_STAGE_14_PRODUCTION_HARDENING` 为 `PASS`。

## REQUIRED STATE

- 普通路径、关键失败路径、状态恢复与验证机制已经通过生产硬化。

## REQUIRED ARTIFACTS

- `production_failure_matrix`
- `hardening_controls`
- `golden_execution_trace_001`
- `autonomous_trial_001`（Stage 09 为 PASS 时）
- `stage09_na_rationale`（如适用）
- `error_boundary_v1`
- `state_model_v1`
- `target_runtime_contract_v1`

## PRINCIPLE

权限扩张按“动作类别”进行，而不是一次性把整个系统切换为全自动。

## ACTIONS

对每类候选动作建立 `autonomy_evidence_record`，至少回答：

- 该动作允许的输入/业务范围是什么？
- 正常路径是否有真实或回归证据？
- 失败能否检测？
- 执行结果能否验证？
- 状态能否可靠恢复或安全停止？
- 已知异常是否会进入停止/人工路线？
- 是否存在同一动作类别尚未解决的 release-blocking failure？
- 当前授权是谁明确批准的？

不使用“感觉已经稳定”作为证据。

可逐步从：

`Prepare only → Execute with approval → Execute automatically + notify → Fully automatic within bounded scope`

## REQUIRED OUTPUT

- `autonomy_matrix_v2`
- `autonomy_evidence_records`
- `approved_autonomous_scope`
- `remaining_human_gates`

## EVIDENCE

- 每项扩大权限都能指向对应 evidence record 与明确批准记录。

## GATE

- 每项新权限都有完整 `autonomy_evidence_record`。
- 对该动作类别不存在未解决的 release-blocking failure。
- 超出批准范围会自动停止或转人工。
- Human escalation 仍存在。
- 未经明确批准的动作权限保持原状。

## NOT_APPLICABLE

如果项目的目标没有任何可扩大自主权限的外部动作（例如系统只做分析/建议，所有执行永久由人完成），可以标记 `NOT_APPLICABLE`。必须记录：

- 为什么不存在可扩大的动作类别；
- 哪些 Human Gate 将永久保留；
- 为什么跳过不会影响最终目标。

## BACKTRACK

- 如果扩大权限需要依赖尚未硬化的能力，回到 Stage 14 或更早阶段。
- 如果新权限改变了原风险分类，回到 Stage 03 重新评估。

## NEXT

`FDE_STAGE_16_COST_COMPRESSION`

---

<a id="FDE_STAGE_16_COST_COMPRESSION"></a>
# FDE_STAGE_16_COST_COMPRESSION — Performance & Cost Compression

## PURPOSE

在系统可靠后，降低 Token、延迟、API、人工和系统资源成本。

## ENTRY CONDITIONS

- `FDE_STAGE_15_AUTONOMY_EXPANSION` 为 `PASS` 或合法 `NOT_APPLICABLE`。

## REQUIRED STATE

- 当前可靠版本已经定义，且后续优化不能改变其验证与停止边界。

## REQUIRED ARTIFACTS

- `workflow_v1`
- `regression_suite`（若尚未正式命名，使用当前全部 release-blocking regression cases）
- `audit_log_spec_v1`
- `autonomy_matrix_v2`（如适用）

## ACTIONS

先建立当前成本/延迟基线，再逐项优化，每项优化后运行关键回归用例。

## OPTIMIZATION ORDER

优先寻找：

1. 删除重复模型调用。
2. 把稳定推理下沉为 Program。
3. 批处理相似任务。
4. 缓存稳定映射。
5. 缩小输入上下文。
6. 使用结构化检索，而非整库塞入 Prompt。
7. 将大模型留给真正困难的判断。
8. 优化模型选择。
9. 优化网络与工具调用次数。

## RULE

任何优化前后都要比较：

- 结果正确性
- 可验证性
- 停止能力
- 可追踪性
- 延迟
- 单位任务成本

性能更快但失去关键验证，不算优化。

## REQUIRED OUTPUT

- `cost_baseline`
- `optimization_changes`
- `cost_after`
- `regression_results`

## EVIDENCE

- 每项优化都有 before/after 数据和对应回归结果。
- 结果变化能追溯到具体优化项，不使用“感觉更快”作为结论。

## GATE

- 优化后的版本通过全部 designated release-blocking regression cases。
- 没有删除必要的验证、停止、审计或人工升级机制。
- 所有宣称的性能/成本改善都有测量依据。

## NOT_APPLICABLE

如果项目当前没有持续运行成本、延迟或规模目标，且优化不会产生实质价值，可以标记 `NOT_APPLICABLE`。必须记录原因和未来重新进入本阶段的触发条件。

## BACKTRACK

- 如果优化导致功能/验证回归，撤销该优化或回到产生问题的 Stage 修复，不允许用新的 Prompt 绕过回归。
- 如果优化暴露 Workflow 或状态架构瓶颈，回到 Stage 12/13。

## NEXT

`FDE_STAGE_17_CONTINUOUS_LEARNING`

---

<a id="FDE_STAGE_17_CONTINUOUS_LEARNING"></a>
# FDE_STAGE_17_CONTINUOUS_LEARNING — Continuous Production Learning

## PURPOSE

为项目建立与生命周期匹配的收尾机制：持续业务进入 Production Learning Loop；一次性/阶段性项目进入 Finite Closeout，并保留可复用证据与回归资产。

## ENTRY CONDITIONS

- `FDE_STAGE_16_COST_COMPRESSION` 为 `PASS` 或合法 `NOT_APPLICABLE`。

## REQUIRED STATE

- 已存在当前稳定运行/交付基线、回归机制、状态/审计与人工升级路线。
- `project.lifecycle` 已明确为 `RECURRING` 或 `FINITE`。

## REQUIRED ARTIFACTS

- 当前 `workflow / rules`，以及 `prompts`（如果项目实际使用 AI Prompt）与其版本标识
- `regression_suite`（若尚未正式命名，使用当前全部 release-blocking regression cases 初始化）
- `incident_log`（首次进入可初始化为空）
- 当前运行/交付基线与审计模型

首次进入本阶段时，可以从当前 artifacts 初始化 `workflow_versions / rule_versions / prompt_versions`，不得因为版本历史文件尚不存在而阻塞。

## LIFECYCLE ROUTES

### RECURRING

将生产事件持续转化为可分类、可回归、可版本化的改进项。任何改动都要保留原因和验证结果。

### FINITE

执行项目收尾：冻结最终版本、保留证据、记录未解决限制、保存回归用例与 lessons learned；如果未来再次部署或业务复用，再从最早受影响 Stage 重新进入。

## ACTIONS

- `RECURRING`：运行下面的 Incident-to-Improvement Loop。
- `FINITE`：生成 `project_closeout`、最终验证结果、限制清单、可复用资产与 reopen triggers。

## EVENT SOURCES

持续学习信号包括：

- Human Correction
- New Exception
- Failed Execution
- New Business Rule
- UI / API Change
- Data Drift
- Cost Spike
- Unexpected Retry
- Duplicate Attempt
- Verification Failure
- New Task Type

## INCIDENT-TO-IMPROVEMENT LOOP

每次事件执行：

`Detect → Preserve Evidence → Classify Cause → Choose Layer → Fix → Add Regression Case → Re-run → Version → Deploy`

## FIX LAYER DECISION

- AI 不理解 → Prompt / Example / Model
- 固定规则 → Rule
- 数据格式 → Schema / Parser
- 操作路径 → Tool
- 状态错误 → State Machine
- 部署错误 → Preflight
- 并发/重复 → Lock / Idempotency
- 新异常不可可靠处理 → Human Escalation

## REQUIRED OUTPUT

维护或归档：

- `regression_suite`
- `incident_log`（RECURRING；FINITE 可为空）
- `change_log`
- `rule_versions`
- `prompt_versions`
- `workflow_versions`
- `project_closeout`（FINITE）

## EVIDENCE

- `RECURRING`：每个生产事件都能形成 `Detect → Evidence → Classification → Fix Layer → Regression → Version` 的闭环记录。
- `FINITE`：最终交付、限制、证据、回归资产与 reopen triggers 均已归档。
- 任何改动都能追溯到原因与验证结果。

## GATE

`RECURRING` 首次进入稳定生产循环前必须满足：

- 普通路径由 `DETERMINISTIC / AGENTIC_BOUNDED / HUMAN` 的显式 Runtime Workflow 承载。
- AI / Program / Human 的职责边界已经记录。
- 关键外部动作均有验证或永久 Human Gate。
- 已知异常均有停止、恢复或人工路线。
- 每条真实任务的状态与日志可追踪。
- 回归与事件改进机制可以实际运行。

`FINITE` 完成项目收尾前必须满足：

- 目标交付已按成功标准验证。
- 未解决限制与 Human Gate 已明确。
- 最终证据、版本和回归用例已归档。
- reopen triggers 已定义。

如果对应生命周期条件尚未满足，Stage 17 为 `BLOCKED` 或按 `BACKTRACK` 回到最早受影响 Stage。

## COMPLETION CONDITION

- `RECURRING` 项目没有永久“完成”，达到 Gate 后进入稳定 Production Learning Loop。
- `FINITE` 项目可以在验收、归档和 closeout 完成后标记 `PROJECT_COMPLETE`。

## BACKTRACK

持续生产中一旦出现新事实，回到**最早受影响的 Stage**，而不是永远只在 Stage 17 修改：

- 业务目标改变 → Stage 00
- 业务流程改变 → Stage 01/02
- 风险/授权改变 → Stage 03
- 成功标准改变 → Stage 04
- 控制面改变 → Stage 05
- 新异常域 → Stage 10
- 工具/Workflow 结构改变 → Stage 11/12
- 状态/持久化问题 → Stage 13
- 生产故障机制不足 → Stage 14

修复后重新经过受影响的后续 Gate，再返回 Stage 17。

## NEXT

- `RECURRING`: `LOOP → FDE_STAGE_17_CONTINUOUS_LEARNING`
- `FINITE`: `END → PROJECT_COMPLETE`

当发生重大变化、再次部署或复用时，按 `BACKTRACK / reopen trigger` 返回最早受影响 Stage。

---

# 2. Cross-Stage Prompt Design Rules

Prompt 是系统组件，不是整个系统。

## 2.1 推荐结构

对单个 AI 任务优先使用：

```text
ROLE
TASK
INPUT
SOURCE-OF-TRUTH / AUTHORITY-POLICY
OUTPUT_SCHEMA
RULES
FORBIDDEN
FAILURE_BEHAVIOR
EXAMPLES
```

## 2.2 核心要求

- 一个 Prompt 尽量只负责一个明确任务。
- 输出尽量结构化。
- 缺失信息要允许返回 `null / unknown / error`。
- 禁止通过历史案例猜当前输入，除非业务明确允许历史作为 Source of Truth。
- 关键冲突应显式报错。
- 规则优先级必须明确。
- 不要让 Prompt 维护长期系统状态；长期状态交给程序/数据库。

---

# 3. AI / Program / Human Allocation Heuristics

## 更倾向 AI

- 非结构化文本理解
- 图片/音频语义
- 开放式分类
- 模糊意图识别
- 复杂异常解释
- 低频长尾判断

## 更倾向 Program

- 固定字段映射
- 正则/格式校验
- 数字计算
- 去重
- 状态机
- API 调用
- 数据库存取
- 重试
- 超时
- 并发锁
- 结构化页面定位
- 规则优先级执行
- 日志

## 更倾向 Human

- 新异常域
- 高影响授权
- 规则仍有争议
- 多个 Source of Truth 冲突
- 无法可靠验证的外部动作
- 业务政策变更确认

---

# 4. Universal Artifact Checklist

一个稳定 FDE 项目最终通常至少具备：

- 项目目标
- Manual SOP
- Workflow Map
- Risk / Delegation Map
- Success / Failure Model
- Capability / Control Surface Map
- Prototype Trace
- Production Observation Log
- Golden Execution Trace
- Error Boundary
- Negative Case Suite
- Tool Library
- Rule Library
- Parser / Schema
- Compiled Workflow
- State Machine
- Data Model
- Audit Log
- Preflight
- Recovery Runbook
- Autonomy Matrix
- Regression Suite
- Incident Loop

不是所有项目都需要把这些做成独立文件，但这些能力不应无故缺失。

---

# 5. Protocol Quality Self-Check

在发布 Skill 新版本前，必须对协议本身做以下七项检查：

1. **顺序正确**：是否存在缺前置、提前执行后续动作或循环依赖。
2. **目的明确**：每个 Stage 是否有不可替代的目标与具体产出。
3. **Gate 客观**：能用事实/证据判断的地方，不允许使用“差不多、明显、足够、已经理解”等主观门槛。
4. **动作具体**：Agent 是否知道当前要问什么、做什么、读什么、留下什么。
5. **可回退**：发现前序结论错误时，是否能回到最早受影响 Stage，并重新经过受影响 Gate。
6. **保持通用**：核心协议是否错误绑定浏览器、OCR、ERP、API、某个模型或某个 Agent 框架。
7. **全局一致**：是否存在权限跃迁、职责冲突、状态未定义、Gate 与前置条件互相矛盾。

发布条件：

> **顺序对、目的清、Gate 硬、动作实、可回退、够通用、无冲突。**

---

# 6. Final Principle

FDE 训练的终点不是让 Agent 获得越来越大的自由度。

正确方向通常是：

**开始时 Agent 为了学习可以很“宽”；进入运行阶段后，稳定且值得工程化的能力逐步被固化为规则、工具、状态和验证；真正需要推理的部分保留为受约束的 Agentic Segment，必须由人承担的部分保留为 Human Gate。**

也就是说：

> **Use AI to discover the system.  
> Use production evidence to constrain it.  
> Compile stable knowledge into software.**

