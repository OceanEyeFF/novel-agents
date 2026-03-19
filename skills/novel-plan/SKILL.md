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

## When to Use

✅ **USE this skill when:**

- 创建新卷/章节的规划
- 修改已有章纲
- 需要把章纲拆成 scene beats
- 查看当前章节规划

## 目录结构

```
plans/
├── arc/                    # 卷纲
│   └── arc-{{卷号}}.md
├── chapter/                # 章纲
│   └── chapter-{{卷}}-{{章}}.md
└── scene/                  # Scene Beats
    └── scene-{{卷}}-{{章}}-{{序号}}.md
```

## 常用操作

### 创建新卷纲

1. 复制模板:
```
read: plans/arc/arc-template.md
```

2. 写入新卷纲:
```
write: plans/arc/arc-{{卷号}}.md
```

### 创建新章纲

1. 复制模板:
```
read: plans/chapter/chapter-template.md
```

2. 写入章纲:
```
write: plans/chapter/chapter-{{卷}}-{{章}}.md
```

### 创建 Scene Beats

1. 复制模板:
```
read: plans/scene/scene-beat-template.md
```

2. 写入 Scene:
```
write: plans/scene/scene-{{卷}}-{{章}}-{{序号}}.md
```

### 查看当前规划

- 当前卷纲: `plans/arc/`
- 当前章纲: `plans/chapter/`
- 当前 Scene: `plans/scene/`

## Scene Beats 必需字段

每个 Scene 必须包含以下字段：

| 字段 | 说明 |
|------|------|
| Scene ID | S{{卷}}-{{章}}-{{序号}} |
| POV | 视角角色 |
| Scene Goal | 场景目标 |
| Conflict | 冲突 |
| Reveal | 揭示的信息 |
| Hide | 隐藏的信息 |
| 进入状态 | 角色进入状态 |
| 退出状态 | 角色退出状态 |
| Token Budget | 字数预算 |

## 规划流程

### 完整流程

1. **Architect** 创建/修改卷纲 → `plans/arc/`
2. **Architect** 创建/修改章纲 → `plans/chapter/`
3. **Scene Planner** 拆解 scene beats → `plans/scene/`

### 规划检查清单

- [ ] 卷纲已确认
- [ ] 章纲已确认
- [ ] Scene beats 已创建
- [ ] 涉及角色已确认
- [ ] 伏笔投放/回收已标记

## 注意事项

1. **顺序**: 必须先有卷纲，再有章纲，最后有 scene beats
2. **一致性**: 规划内容不得违背 canon
3. **可追溯**: 每次修改应有明确原因
