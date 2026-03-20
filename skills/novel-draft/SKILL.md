---
name: novel-draft
description: 小说创作辅助 - 场景 skeleton、prose expansion、桥接段生成与续写支持。
metadata:
  {
    "openclaw":
      {
        "emoji": "✍️",
        "description": "小说创作辅助工具",
      },
  }
---

# novel-draft

小说创作辅助工具，帮助 Writer Agent 完成 skeleton、expansion 和 bridge 三类写作任务。

> 执行前提：必须先锁定 `novel_id`，正文读写和续写参考只能来自 `projects/{{novel_id}}/workspace/`。

## When to Use

✅ **USE this skill when:**

- 需要先写一版结构正确的 scene skeleton
- 需要在不改剧情事实的前提下做 prose expansion
- 需要生成相邻 Scene 之间的过桥 / 余波 / 呼吸段
- 需要参考前文风格、人物状态和 prose brief

## 三种模式

### 1. Skeleton Draft

目标：先把事件顺序、信息揭示、情绪方向写正确。

**输入至少包括：**
- 当前 Scene Beat
- 当前 Prose Brief
- 当前角色状态
- `must_not_include` / `hidden_information`

### 2. Prose Expansion

目标：在不改变结构功能的前提下补足：
- 环境锚点
- 身体感受
- 内心活动
- 潜台词
- 段落节奏变化

### 3. Bridge Draft

目标：只负责两个相邻 Scene 间的过桥。

**禁止：**
- 发明新的主事件
- 偷偷推进未批准的剧情
- 合并两个 Scene 的核心情节到一个文件

## 常用操作

### 读取当前 Scene Beat

```bash
read: projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 读取当前 Prose Brief

```bash
read: projects/{{novel_id}}/workspace/plans/prose/prose-{{ID}}.md
```

### 读取角色状态

```bash
read: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 读取最近正文作为风格参考

```bash
read: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 写场景草稿

```bash
write: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 写桥接草稿

```bash
write: projects/{{novel_id}}/workspace/drafts/bridges/bridge-{{ID-A}}-{{ID-B}}.md
```

## Writer 工作流

1. 读取 Scene Beat + Prose Brief + 角色状态。
2. 先写 skeleton，确认事件与边界正确。
3. 再做 expansion，补足小说性与密度。
4. 如相邻 Scene 有拼接感，再补 bridge。
5. 写完后交给 `novel-guard` / `novel-verify`。

## 注意事项

1. `novel-draft` 是写作辅助，不是绕过 Beat / Prose Brief 的自由发挥许可。
2. expansion 不能改写结构事实。
3. bridge 只能连接相邻 Scene，不能替代章节组装。
