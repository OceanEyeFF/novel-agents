---
name: novel-plan
description: 小说规划工具 - 创建和管理卷纲、章纲、Scene Beats。
metadata:
  {
    "openclaw":
      {
        "emoji": "📋",
        "description": "小说规划管理",
      },
  }
---

# novel-plan

管理卷纲、章纲、Scene Beats 的创建与修改。

> 执行前提：必须先锁定 `novel_id`，所有规划文件都应写入 `projects/{{novel_id}}/workspace/plans/`。

## When to Use

✅ **USE this skill when:**

- 创建新卷/章节的规划
- 修改已有章纲
- 需要把章纲拆成 scene beats
- 查看当前章节规划

## 目录结构

```text
projects/{{novel_id}}/workspace/plans/
├── arc/                    # 卷纲
│   └── arc-{{卷号}}.md
├── chapter/                # 章纲
│   └── chapter-{{卷}}-{{章}}.md
└── scene/                  # Scene Beats
    └── scene-{{卷}}-{{章}}-{{序号}}.md
```

## 规划标识规范

所有规划文件都必须携带可机读的唯一标识：

- `arc_id`
- `chapter_id`
- `scene_id`
- `sequence_index`
- `status`

### 章纲最小头信息

```markdown
# Chapter Plan: {{chapter_id}}
- arc_id: {{arc_id}}
- chapter_id: {{chapter_id}}
- title: {{章节标题}}
- status: approved
```

### Scene Beat 最小头信息

```markdown
# Scene Beat: {{scene_id}}
- chapter_id: {{chapter_id}}
- scene_id: {{scene_id}}
- sequence_index: {{序号}}
- status: approved
```

## 常用操作

### 创建新卷纲

1. 复制模板:
```text
read: projects/{{novel_id}}/workspace/plans/arc/arc-template.md
```

2. 写入新卷纲:
```text
write: projects/{{novel_id}}/workspace/plans/arc/arc-{{卷号}}.md
```

### 创建新章纲

1. 复制模板:
```text
read: projects/{{novel_id}}/workspace/plans/chapter/chapter-template.md
```

2. 写入章纲:
```text
write: projects/{{novel_id}}/workspace/plans/chapter/chapter-{{卷}}-{{章}}.md
```

### 创建 Scene Beats

1. 复制模板:
```text
read: projects/{{novel_id}}/workspace/plans/scene/scene-beat-template.md
```

2. 写入 Scene:
```text
write: projects/{{novel_id}}/workspace/plans/scene/scene-{{卷}}-{{章}}-{{序号}}.md
```

## Scene Beats 必需字段

每个 Scene 必须包含以下字段：

| 字段 | 说明 |
|------|------|
| Scene ID | S{{卷}}-{{章}}-{{序号}} |
| Chapter ID | CH{{卷}}-{{章}} |
| POV | 视角角色 |
| Scene Goal | 场景目标 |
| Conflict | 冲突 |
| Reveal | 揭示的信息 |
| Hide | 隐藏的信息 |
| 进入状态 | 角色进入状态 |
| 退出状态 | 角色退出状态 |
| Token Budget | 字数预算 |
| Must Not Include | 明确禁止写入的跨章信息 |

## 规划流程

### 完整流程

1. **Architect** 创建/修改卷纲 → `projects/{{novel_id}}/workspace/plans/arc/`
2. **Architect** 创建/修改章纲 → `projects/{{novel_id}}/workspace/plans/chapter/`
3. **Scene Planner** 拆解 scene beats → `projects/{{novel_id}}/workspace/plans/scene/`
4. **Critic / Guard** 校验 ID 和顺序完整性 → 才能进入写作

## 规划检查清单

- [ ] 卷纲已确认
- [ ] 章纲已确认
- [ ] Scene beats 已创建
- [ ] 每个 Scene 都有唯一 `scene_id`
- [ ] 所有 Scene 的 `chapter_id` 一致且可追溯到章纲
- [ ] 涉及角色已确认
- [ ] 伏笔投放/回收已标记
- [ ] `must_not_include` 已填写

## 注意事项

1. **顺序**: 必须先有卷纲，再有章纲，最后有 scene beats
2. **一致性**: 规划内容不得违背 canon
3. **可追溯**: 每次修改应有明确原因
4. **防串章**: 若一个规划文件无法唯一映射到单一 `chapter_id`，该规划视为无效
