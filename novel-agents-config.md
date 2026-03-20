# Writer Room Agent 配置 v0.2

> 每个 agent 的角色定义、工具权限与状态机协作

---

## 0. Orchestrator（调度员）

```yaml
name: orchestrator
role: 调度员
description: 常驻中心管理，驱动章节状态机，决定调用谁、何时返工、何时升级人工

read:
  - all                 # 读取全部元信息、产物状态、审批链、日志

write:
  - logs/*              # 执行日志
  - jobs/*              # 任务状态机记录
  - manifests/*         # chapter / scene 流转清单

state_machine:
  - planned
  - beat_ready
  - prose_ready
  - drafting
  - review_failed_minor
  - review_failed_major
  - replan_required
  - scene_approved
  - chapter_assembling
  - chapter_approved
  - archived

route_rules:
  - if: missing_chapter_plan
    then: architect
  - if: missing_scene_beats
    then: scene_planner
  - if: missing_prose_brief
    then: prose_director
  - if: critic.required_action == fix_local_language
    then: writer
  - if: critic.required_action == expand_scene
    then: writer
  - if: critic.required_action == bridge_scenes
    then: writer
  - if: critic.required_action == rewrite_scene
    then: writer
  - if: critic.required_action == replan_scene
    then: scene_planner
  - if: critic.required_action == replan_chapter
    then: architect
  - if: critic.required_action == escalate_human
    then: human_review

deny:
  - drafts/*            # 禁止直接写正文
  - reviews/*           # 禁止直接写评审
  - states/*            # 禁止直接改状态
  - canon/*             # 禁止改设定
```

---

## 1. Architect（总规划师）

```yaml
name: architect
role: 总规划师
description: 维护全书弧光、卷纲、章纲，管控主题/高潮/节奏分布

read:
  - canon/*
  - plans/arc/*
  - plans/chapter/*
  - states/characters/*
  - states/timeline/*
  - states/open_loops/*

write:
  - plans/chapter/*
  - plans/arc/*

deny:
  - drafts/*
  - reviews/*
  - canon/create
```

---

## 2. Character Director（角色导演）

```yaml
name: character_director
role: 角色导演
description: 维护角色状态机，输出角色行动约束、情绪边界、认知差

read:
  - states/characters/*
  - plans/chapter/current
  - plans/scene/current
  - canon/characters/*

write:
  - states/characters/*
  - manifests/character_constraints/*

deny:
  - drafts/*
  - plans/chapter/*
  - canon/*
```

---

## 3. Scene Planner（场景策划）

```yaml
name: scene_planner
role: 场景策划
description: 把章纲拆成 scene beats，控制顺序、进入/退出状态与隐藏信息边界

read:
  - plans/chapter/current
  - states/characters/current
  - states/open_loops/*

write:
  - plans/scene/*

required_fields:
  - chapter_id
  - scene_id
  - sequence_index
  - pov
  - narrative_function
  - scene_goal
  - conflict
  - reveal_or_hide
  - entry_state
  - exit_state
  - must_include
  - must_not_include

deny:
  - drafts/*
  - states/*
  - canon/*
```

---

## 4. Prose Director（小说导演）

```yaml
name: prose_director
role: 小说导演
description: 把 scene beats 翻译成 prose brief，指定叙述重心、慢写点、感官目标与段落节奏

read:
  - plans/scene/current
  - manifests/character_constraints/*
  - states/characters/current
  - drafts/scenes/previous
  - canon/style/*

write:
  - plans/prose/*

required_fields:
  - chapter_id
  - scene_id
  - emotional_core
  - narrative_focus
  - slow_beats
  - sensory_targets
  - inner_monologue_targets
  - dialogue_purpose
  - hidden_information
  - paragraph_rhythm
  - prose_length_plan

deny:
  - drafts/*
  - plans/chapter/*
  - states/*
```

---

## 5. Writer（正文写手）

```yaml
name: writer
role: 正文写手
description: 根据 prose brief 写 prose，先完成 skeleton，再执行 expansion 与 bridge 补写

read:
  - plans/scene/current
  - plans/prose/current
  - manifests/character_constraints/*
  - states/characters/current
  - drafts/scenes/previous
  - canon/style/*

write:
  - drafts/scenes/*
  - drafts/bridges/*

modes:
  - draft_skeleton
  - expand_prose
  - bridge_scenes

deny:
  - states/*
  - plans/chapter/*
  - canon/*
```

---

## 6. Critic（批评审校）

```yaml
name: critic
role: 批评审校
description: 先检查结构完整性，再检查逻辑/节奏/OOC/设定冲突，并评估文本密度与小说性

read:
  - drafts/scenes/current
  - drafts/bridges/*
  - plans/scene/current
  - plans/prose/current
  - states/characters/*
  - states/open_loops/*
  - canon/*

write:
  - reviews/*

required_scores:
  - gate_a_result
  - gate_b_result
  - scene_density
  - novelness
  - chapter_cohesion
  - decision
  - required_action

script_like_risk_rules:
  threshold: 3
  required_repairs:
    - environment_anchor
    - body_sensation
    - inner_misread_or_association
    - slow_beat
    - texture_detail

allow_required_action:
  - fix_local_language
  - expand_scene
  - rewrite_scene
  - bridge_scenes
  - replan_scene
  - replan_chapter
  - escalate_human

deny:
  - drafts/*
  - states/*
  - plans/*
```

---

## 7. Archivist（档案官）

```yaml
name: archivist
role: 档案官
description: 场景或章节结束后更新状态层，写摘要、更新时间线、保存快照与索引

read:
  - drafts/scenes/approved
  - drafts/bridges/approved
  - reviews/*
  - states/*

write:
  - states/characters/*
  - states/timeline/*
  - states/open_loops/*
  - summaries/chapters/*
  - snapshots/*

archive_checks:
  - review_exists
  - decision_is_approved
  - gate_a_result_is_pass
  - target_ids_match
  - artifact_is_not_mixed_chapter_doc

deny:
  - drafts/*
  - plans/*
  - canon/*
```

---

## 工具映射表（v0.2）

| 工具 | orchestrator | architect | character_director | scene_planner | prose_director | writer | critic | archivist |
|------|--------------|-----------|--------------------|---------------|----------------|--------|--------|-----------|
| get_canon | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ |
| get_character_state | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| update_character_state | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| get_timeline_window | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| register_plot_point | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| get_open_loops | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| create_chapter_plan | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| create_scene_beats | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| create_prose_brief | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| draft_scene_skeleton | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| expand_scene_prose | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| bridge_adjacent_scenes | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| check_canon_conflict | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| check_character_consistency | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| check_style_drift | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| score_scene_density | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| score_novelness | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| score_chapter_cohesion | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| retrieve_relevant_memories | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| snapshot_scene | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 人工审批策略

只在以下情况强制人工介入：

1. 首次章纲批准
2. 重大设定变更
3. 最终发布前抽检
4. Critic / Orchestrator 输出 `escalate_human`
