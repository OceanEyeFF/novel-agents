---
name: novel-novelize
description: 生成 Prose Brief，并在不改剧情事实的前提下做小说化扩写，降低场景的剧本化、摘要化与提纲化风险。
---

# novel-novelize

这个 Skill 负责把结构正确的 Scene Beat 转成可执行的小说表达方案，并在需要时协助做小说化扩写。

> 执行前提：必须先锁定 `novel_id`，所有输入与输出都必须绑定 `projects/{{novel_id}}/workspace/`。

## When to Use

✅ **USE this skill when:**

- Scene Beat 已经结构正确，但直接写容易变成提纲扩写
- 需要先生成 `Prose Brief`
- Writer 已完成 skeleton，需要补足小说质感与文本密度
- Critic 提示 `script_like_risk` 偏高，需要做定向修复

## 输入来源

- `projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md`
- `projects/{{novel_id}}/workspace/manifests/character_constraints/{{scene_id}}.md`
- `projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md`
- `projects/{{novel_id}}/workspace/canon/style/{{style_id}}.md`
- `projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md`（扩写时）

## 输出位置

- `projects/{{novel_id}}/workspace/plans/prose/prose-{{ID}}.md`
- 必要时只对当前 Scene Draft 提供扩写建议，不改写 Beat 结构边界

## Workflow

1. 读取已批准的 `scene beat`、角色约束与风格模板。
2. 先产出 **prose brief**，不要跳过这一步直接写正文。
3. Writer 先根据 brief 生成 **scene skeleton**。
4. 再做 **novelization expansion**，补足感官、内心、潜台词和段落节奏。
5. 若 `script_like_risk` 偏高，按修复清单做定向 densification。

## Required prose brief fields

- `chapter_id`
- `scene_id`
- `emotional_core`
- `narrative_focus`
- `slow_beats`
- `sensory_targets`
- `inner_monologue_targets`
- `dialogue_purpose`
- `hidden_information`
- `paragraph_rhythm`
- `prose_length_plan`

## Expansion rules

扩写时只能补以下内容，除非章纲明确允许更多：

- environment anchors
- body sensations
- inner misreads or associations
- dialogue subtext
- pacing pauses and aftershocks
- texture details that increase presence

**Do not:**

- invent new plot turns
- reveal hidden information early
- merge multiple scenes into one draft
- replace POV discipline with omniscient summary

## Script-like risk heuristic

如果以下特征命中 3 条以上，判定为高风险并请求修订：

- 对话承担了几乎全部推进
- 段落只报告动作或信息
- 情绪只被命名，没有被体验化
- 场景环境只被提到，没有参与叙事
- 对白纯功能化，潜台词过少
- 段落长度机械均匀
- 转场没有余波句
- POV 名义存在，但页面上没有视角粘性

## Repair checklist

当风险偏高时，至少补齐：

- 1 个 environment anchor
- 1 个 body sensation
- 1 个 inner misread or memory echo
- 1 个 slowed beat
- 1 个 non-functional texture detail
