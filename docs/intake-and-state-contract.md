# Writer Room v0.2 Intake 与 State Contract

## 目标

把“作者自然语言想法”转换为“Orchestrator 可执行的状态契约”，避免系统直接消费模糊 prompt。

---

## 1. 两层系统

### 对话前台层

负责：

- 判断信息是否足够进入中台
- 只追问当前推进所需的最小关键缺口
- 显式输出已确认 / 未确认 / 假设 / 决策权限

推荐先实现为 `briefing` capability；是否独立成 agent 取决于后续 workload tests。

### 小说中台层

负责：

- 读取 `project.yaml` 与 `runtime.yaml`
- 根据 `runtime.stage` 自动推进章节工作流
- 把返工路由到正确层级

推荐先实现为 `bootstrap` capability + `orchestrator` + 执行层 Agents。

---

## 2. Intake 应澄清的五组信息

1. **项目身份信息**：书名、题材、平台、目标读者、项目阶段。
2. **创作意图信息**：希望读者获得的体验、节奏、文风、避坑项。
3. **设定与基线信息**：世界观、主角、锁死设定、可调整设定、已有文档。
4. **当前任务信息**：现在要推进什么、边界在哪里、本轮交付到什么程度。
5. **决策权限信息**：哪些可自动决定，哪些必须回问作者。

---

## 3. Intake Summary 模板

```yaml
intake_summary:
  confirmed:
    - 这是一部偏男频番茄向的群像权谋玄幻
    - 前五章重点是入局，不急着爆设定
  assumed:
    - 第一卷优先建立压迫感与信任危机
  unresolved:
    - 顾长歌到底是纯敌对还是摇摆立场
    - 第一卷高潮落在哪一章
  blocked_by:
    - 若直接写正文，顾长歌立场缺少边界
  decision_rights:
    auto_allowed:
      - scene 内补局部环境细节
      - 不改 canon 前提下补过桥段
    human_required:
      - 改动 baseline
      - 调整卷级高潮位置
  next_recommended_step: 先进入第一卷卷纲规划
  ready_for_backend: true
```

---

## 4. State Contract 拆分

### `project.yaml`（长期 / 慢变）

```yaml
project:
  novel_id: lin-yuan
  novel_name: 临渊
  version: v0.2
  genre: [玄幻, 群像, 权谋]
  target_platform: 番茄
  target_audience: 男频20-35
  style_profile:
    narration_style: 第三人称贴近视角
    prose_density: medium-high
    dialogue_ratio: 0.4
    pacing: medium
  baseline:
    canon_locked: true
    theme: 权力与信任的崩塌
    forbidden:
      - 穿越设定
      - 系统外挂
```

### `runtime.yaml`（快变 / 执行控制）

```yaml
narrative_state:
  current_arc: 第一卷·入局
  current_chapter: 5
  current_scene: 2
  pov_character: 林渊
  active_characters:
    - 林渊
    - 顾长歌
  emotional_target: 压迫+试探
  narrative_function: 关系试探+信息误导
  open_loops:
    - 顾长歌真实立场
  reveal_constraints:
    must_not_reveal:
      - 幕后组织
    can_hint:
      - 顾长歌不完全可信

task:
  task_id: scene-01-05-02
  task_type: scene_generation
  input_refs:
    chapter_plan_id: chapter-01-05
    scene_beat_id: scene-01-05-02
  requirements:
    min_length: 900
    max_length: 1500
    must_include:
      - 环境压迫感
      - 林渊的内心误判
    must_avoid:
      - 信息直接解释
      - 总结式句子
    quality_target:
      density: high
      immersion: high
      plot_progress: medium

runtime:
  stage: drafting
  auto_flow: true
  allow_replan: true
  retry_policy:
    max_retry: 2
    escalate_on_fail: replan_scene
  human_gate:
    require_approval: false
    approval_type: null
  next_action_hint: call_writer
```

---

## 5. Agent 输入 / 输出规范

- **briefing capability 输入**：作者自然语言、已有设定、历史项目状态。
- **briefing capability 输出**：`intake_summary`。
- **bootstrap capability 输入**：`intake_summary` + baseline 文档。
- **bootstrap capability 输出**：`project.yaml`、`runtime.yaml`。
- **Orchestrator 输入**：`project.yaml`、`runtime.yaml`、工具回执。
- **执行层 Agent 输出**：产物 + 对 `runtime.yaml` 局部字段的更新建议。

---

## 6. 准入规则

只有同时满足以下条件时，才允许进入中台：

1. `intake_summary.ready_for_backend == true`。
2. `project.yaml` 已存在，且包含 `style_profile` 与 `baseline`。
3. `runtime.yaml` 已存在，且包含 `narrative_state`、`task`、`runtime` 三段。
4. `runtime.stage` 不处于 `idea_raw` / `brief_partial`。

若上述任一条件不满足，Orchestrator 必须回退到 Briefing / Bootstrap，而不是硬启动规划或写作。


---

## 7. 是否需要成为新的 Room Agents

我的判断是：**它们很重要，但现在更需要讨论和 workload tests，而不是直接扩编。**

原因：

1. `briefing` / `bootstrap` 解决的是“入口状态契约”问题，这个问题确实关键。
2. 但它们和 Orchestrator、Architect、甚至前端产品形态的边界还没有完全定型。
3. 如果过早把它们固定成 Room Agents，可能会带来额外协作成本、上下文切换和状态重复维护。

### 建议的落地顺序

1. 先把它们实现为 **preflight modes / services**。
2. 用真实任务批次测试它们是否稳定降低人工介入和误启动。
3. 只有在边界清晰、收益稳定后，再升格成新的 Room Agents。

### 建议 workload tests

- **Idea → Brief**：作者只有一句模糊想法，测试能否收敛成可执行 intake summary。
- **Brief → Runtime**：测试 bootstrap 生成的 `project.yaml` / `runtime.yaml` 是否字段完整、无层级混淆。
- **Re-entry**：中途修改任务时，测试是否能只更新快变层而不污染 project layer。
- **Failure routing**：缺字段、冲突 baseline、权限不足时，测试是否能正确回退，而不是误入写作流程。
- **Cost / latency**：比较新增 preflight 前后的轮次、token、总耗时。
