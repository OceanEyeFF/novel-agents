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

## 2. Intake 应澄清的六组信息

1. **项目身份信息**：书名、题材、平台、目标读者、项目阶段。
2. **创作意图信息**：希望读者获得的体验、节奏、文风、避坑项。
3. **阅读体验信息**：读者应该在哪些节点笑、心酸、紧张、想截图发弹幕；哪些章节必须打出情绪记忆点。
4. **设定与基线信息**：世界观、主角、锁死设定、可调整设定、已有文档。
5. **当前任务信息**：现在要推进什么、边界在哪里、本轮交付到什么程度。
6. **决策权限信息**：哪些可自动决定，哪些必须回问作者。

---

## 2.1 Intake 不应只停留在情节大纲

如果 intake 只收集“发生了什么”，后续 Planner / Writer 往往只能写出功能正确但缺乏质感的章节。更实用的 intake 应该至少补上三类信息：

- **章节级体验目标**：这一章要让读者主要感到什么，是甜、酸、压迫、释然，还是混合情绪。
- **情绪命中点**：大致在哪个 scene 或段落应该让读者笑、心酸、紧张、想截图。
- **传播记忆点**：哪些句子、动作、误会、反转值得成为截图 / 弹幕 / 评论区讨论点。

这类信息不一定要在项目启动时一次性补齐，但至少要在进入章级规划前逐步收敛。

如果工作量过大，可以接受一个折中策略：

- intake 先只确定**卷级 / 章级体验目标**；
- 到 chapter table（章节表）阶段，再逐章补充情绪命中点；
- 具体段落级的“融情于景”由 Chapter Director 在写作前细化。

## 3. Intake Summary 模板

```yaml
intake_summary:
  confirmed:
    - 这是一部偏男频番茄向的群像权谋玄幻
    - 前五章重点是入局，不急着爆设定
  reading_experience:
    volume_targets:
      - 第一卷要让读者持续感到压迫与信任危机
    chapter_targets:
      - 第五章收尾要让读者产生"顾长歌不可信但又离不开他"的复杂感
    highlight_moments:
      - 在档案室对话的中段制造一次适合评论区讨论的误判
      - 结尾留一个适合截图传播的冷感句子
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

## 4. State Contract 之外，还需要三类长期写作素材

单靠 `project.yaml` / `runtime.yaml` 还不足以支撑章节质感。实操中建议额外维护三类长期素材，它们不一定都属于 state contract，但都应该被 Writer / Director 稳定读取。

### 4.1 Canon Bible 的氛围细节（Atmosphere Anchors）

传统 canon bible 往往只有设定与规则，但 Writer 更需要可复用的感官锚点，例如：

- 场所的声音、气味、光线、温度、触感
- 某类场景的社会噪音与背景动作
- 角色对这些环境细节的主观偏好 / 厌恶

建议只为**核心高频场景**建立基础描写库，而不要试图穷举所有地点。具体章节中如何“融情于景”，仍应由 Chapter Director 结合当章情绪目标来调度。

### 4.2 Character Card 的内心语法（Inner Grammar）

角色卡不能只停留在背景和性格标签，还应补足角色在压力、撒谎、亲密、失控时的内心语法与语言习惯。

一个实用做法是为主要角色设计 **7 个问答题**，例如：

1. 压力大时，内心独白会更绕还是更短？
2. 撒谎时，会解释过多、转移话题，还是故意留白？
3. 对喜欢的人，内心会自我否认、过度解读，还是先行动后后悔？
4. 情绪受伤时，会先观察细节、先攻击别人，还是先攻击自己？
5. 做决定时，是先算风险还是先顺情绪？
6. 尴尬时，语言会变快、变硬，还是故作平静？
7. 极度脆弱时，最可能暴露的不是秘密，而是什么口癖 / 动作 / 思维跳跃？

这样得到的不是抽象标签，而是 Writer 可直接调用的“内心与语气模板”。

### 4.3 Arc / Chapter 的情感曲线

章纲若只描述“发生什么”，仍然会导致章节情绪缺乏起伏。建议至少补充：

- 情绪起点
- 情绪高潮 / 误判点
- 结尾情感余额（收得更轻、还是压住不放）

考虑到工作量，这一层不必在 intake 阶段一次性硬写完。更务实的落地顺序是：

- 由 Planner / Director **自组织**生成初版情感曲线；
- 由 Critic / 人工审核做**自我审核**与微调；
- 只在关键章、高潮章、关系转折章要求完整情感曲线。

## 5. State Contract 拆分

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

## 6. Agent 输入 / 输出规范

- **briefing capability 输入**：作者自然语言、已有设定、历史项目状态。
- **briefing capability 输出**：`intake_summary`。
- **bootstrap capability 输入**：`intake_summary` + baseline 文档。
- **bootstrap capability 输出**：`project.yaml`、`runtime.yaml`。
- **Orchestrator 输入**：`project.yaml`、`runtime.yaml`、工具回执。
- **执行层 Agent 输出**：产物 + 对 `runtime.yaml` 局部字段的更新建议。

---

## 7. 准入规则

只有同时满足以下条件时，才允许进入中台：

1. `intake_summary.ready_for_backend == true`。
2. `project.yaml` 已存在，且包含 `style_profile` 与 `baseline`。
3. `runtime.yaml` 已存在，且包含 `narrative_state`、`task`、`runtime` 三段。
4. `runtime.stage` 不处于 `idea_raw` / `brief_partial`。

若上述任一条件不满足，Orchestrator 必须回退到 Briefing / Bootstrap，而不是硬启动规划或写作。


---

## 8. 是否需要成为新的 Room Agents

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
