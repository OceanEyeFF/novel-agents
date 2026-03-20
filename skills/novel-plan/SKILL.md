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

## 章纲最小头信息

```markdown
# Chapter Plan: {{chapter_id}}
- arc_id: {{arc_id}}
- chapter_id: {{chapter_id}}
- title: {{章节标题}}
- status: approved
```

## Scene Beat 最小头信息

```markdown
# Scene Beat: {{scene_id}}
- chapter_id: {{chapter_id}}
- scene_id: {{scene_id}}
- sequence_index: {{序号}}
- status: approved
```

## Scene Beats 必需字段

每个 Scene 必须包含以下字段：

| 字段 | 说明 |
|------|------|
| Scene ID | S{{卷}}-{{章}}-{{序号}} |
| Chapter ID | CH{{卷}}-{{章}} |
| POV | 视角角色 |
| Narrative Function | 本场景在章节中的叙事功能 |
| Scene Goal | 场景目标 |
| Conflict | 冲突 |
| Reveal / Hide | 揭示与隐藏信息 |
| 进入状态 | 角色进入状态 |
| 退出状态 | 角色退出状态 |
| Must Include | 本场必须出现的信息 / 动作 |
| Must Not Include | 明确禁止写入的跨章信息 |

## 规划流程

1. **Architect** 创建/修改卷纲 → `projects/{{novel_id}}/workspace/plans/arc/`
2. **Architect** 创建/修改章纲 → `projects/{{novel_id}}/workspace/plans/chapter/`
3. **Scene Planner** 拆解 Scene Beats → `projects/{{novel_id}}/workspace/plans/scene/`
4. **Critic / Guard** 校验 ID 和顺序完整性 → 才能进入写作
5. **Prose Director** 基于已批准的 Scene Beats 生成 Prose Brief

## 规划检查清单

- [ ] 卷纲已确认
- [ ] 章纲已确认
- [ ] Scene beats 已创建
- [ ] `sequence_index` 连续无跳号
- [ ] `must_not_include` 已写明边界
