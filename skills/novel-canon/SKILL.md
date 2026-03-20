---
name: novel-canon
description: 获取和管理小说核心设定（Canon Bible）。用于查询世界观、规则、文风模板、禁止事项等不可违背的事实。
metadata:
  {
    "openclaw":
      {
        "emoji": "📖",
        "description": "小说核心设定管理",
      },
  }
---

# novel-canon

管理小说核心设定（Canon Bible），所有创作必须遵循的事实。

> 执行前提：必须先锁定 `novel_id`，所有读写只允许发生在 `projects/{{novel_id}}/workspace/` 内。

## When to Use

✅ **USE this skill when:**

- 查询世界观、设定、规则
- 确认某设定是否已有明确规定
- 需要引用 canon 中的禁止事项
- 规划新情节时确认不冲突
- 需要读取文风模板给 `prose_director` / `writer`

## 文件位置

- `projects/{{novel_id}}/workspace/canon/bible.md` - 当前小说的核心设定（世界观、剧情主线、文风、禁止事项）
- `projects/{{novel_id}}/workspace/canon/characters/` - 当前小说的角色基础设定
- `projects/{{novel_id}}/workspace/canon/style/` - 文风模板、语言基准、节奏要求

## 常用操作

### 读取完整 Canon Bible

直接读取文件：

```
read: projects/{{novel_id}}/workspace/canon/bible.md
```

### 读取特定角色设定

```
read: projects/{{novel_id}}/workspace/canon/characters/{{角色名}}.md
```

### 读取文风模板

```
read: projects/{{novel_id}}/workspace/canon/style/{{style_id}}.md
```

### 列出所有角色

```
exec: ls projects/{{novel_id}}/workspace/canon/characters/
```

## 使用原则

1. Canon 是事实，不是建议。
2. 文风模板属于 canon 的一部分，供 `prose_director` / `writer` 参考，但不能被私自改写。
3. 若发现正文与 canon 冲突，应先阻断生产，再由 Architect / 人工决策修订。
