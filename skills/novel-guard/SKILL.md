---
name: novel-guard
description: 小说发布闸门 - 检查章节边界、元信息一致性、审批完备性，阻止串章/混稿/误批准。
metadata:
  {
    "openclaw":
      {
        "emoji": "🛡️",
        "description": "小说发布闸门",
      },
  }
---

# novel-guard

这是一个专门用于“放行前”做边界检查的 Skill。它不负责润色文本，而是负责阻止错误的文档进入审核通过或归档阶段。

> 执行前提：必须先锁定 `novel_id`，所有检查对象与输出报告都必须绑定 `projects/{{novel_id}}/workspace/`。

## When to Use

✅ **USE this skill when:**

- Writer 刚完成场景稿，准备送审
- Critic 准备出最终审批结论前
- Archivist 准备归档前
- 怀疑出现了串章、混稿、标题错乱、错误批准

## 输入与输出

### 常见输入
- `projects/{{novel_id}}/workspace/drafts/scenes/`
- `projects/{{novel_id}}/workspace/drafts/bridges/`
- `projects/{{novel_id}}/workspace/plans/scene/`
- `projects/{{novel_id}}/workspace/plans/prose/`
- `projects/{{novel_id}}/workspace/reviews/`

### 输出报告
- `projects/{{novel_id}}/workspace/reviews/guard-{{target_id}}.md`

## 守门目标

1. **单文档单归属**：一个文档只能属于一个 `chapter_id` / `scene_id`。
2. **元信息一致**：文件名、标题、正文头、引用的 Beat / Prose Brief 必须一致。
3. **审批链闭环**：未完成结构检查的稿件不能被批准，也不能被归档。
4. **摘要与正文分离**：章节摘要、审稿报告、正文必须是不同产物。

## 检查清单

### A. 身份一致性

- [ ] 文件名包含目标 `chapter_id` / `scene_id`
- [ ] 标题中的 ID 与文件名一致
- [ ] 正文头中的 ID 与标题一致
- [ ] `source_beat` 与 `source_prose_brief` 存在且属于相同章节

### B. 范围一致性

- [ ] 文档内容只覆盖一个 Scene 或单个明确的章节产物
- [ ] 没有混入其他章节标题
- [ ] 没有混入其他 Scene 的未声明正文
- [ ] 没有把摘要、提纲、审稿意见写进正文正文区

### C. 审批一致性

- [ ] 结构检查已完成
- [ ] 评审报告的 `review_target` 与正文一致
- [ ] 决策字段存在且唯一
- [ ] 若 `decision ≠ approved`，则禁止归档

## 失败处理

### 如果失败，必须明确给出以下动作之一：

- `split_document`
- `rename_or_reheader`
- `rewrite_scene`
- `relink_source_beat`
- `escalate_to_architect`

## 推荐输出格式

```markdown
# Guard Report - {{target_id}}

## Result
- pass: yes / no
- target_type: scene_draft / bridge_draft / chapter_summary / review

## Findings
| 类型 | 问题 | 严重度 | 建议动作 |
|------|------|--------|----------|

## Final Gate
- allow_review: yes / no
- allow_archive: yes / no
```

## 注意事项

1. Guard 不是可选优化，而是发布前闸门。
2. Guard 检查失败时，不允许用“问题不大”放行。
3. Guard 结论应优先于风格层面的轻微问题。
