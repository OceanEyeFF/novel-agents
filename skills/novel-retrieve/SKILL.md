---
name: novel-retrieve
description: 小说记忆检索 - 从上下文中检索相关信息。
metadata:
  {
    "openclaw":
      {
        "emoji": "🔎",
        "description": "小说记忆检索工具",
      },
  }
---

# novel-retrieve

从小说世界的各个角落检索相关信息，支撑上下文组装。

> 执行前提：必须先锁定 `novel_id`，默认只检索 `projects/{{novel_id}}/workspace/` 内的数据，禁止隐式跨书召回。

## When to Use

✅ **USE this skill when:**

- 写当前场景时需要参考之前的内容
- 需要查找某个角色的历史行为
- 需要召回相关的伏笔
- 需要获取特定时间点的状态
- 需要给 `prose_director` / `writer` 组装上下文包

## 检索来源

### 固定层（始终可查）
- `projects/{{novel_id}}/workspace/canon/bible.md` - 核心设定
- `projects/{{novel_id}}/workspace/canon/style/` - 风格模板
- `projects/{{novel_id}}/workspace/canon/characters/` - 角色基础设定
- `projects/{{novel_id}}/workspace/plans/scene/` - 当前 Scene Beats
- `projects/{{novel_id}}/workspace/plans/prose/` - 当前 Prose Brief

### 动态状态层
- `projects/{{novel_id}}/workspace/states/characters/*.md` - 角色动态状态
- `projects/{{novel_id}}/workspace/states/timeline/index.md` - 时间线
- `projects/{{novel_id}}/workspace/states/open_loops/index.md` - 伏笔表
- `projects/{{novel_id}}/workspace/manifests/character_constraints/` - 角色约束

### 内容层
- `projects/{{novel_id}}/workspace/summaries/chapter-*.md` - 章节摘要
- `projects/{{novel_id}}/workspace/drafts/scenes/draft-*.md` - 草稿正文
- `projects/{{novel_id}}/workspace/drafts/bridges/*.md` - 过桥稿
- `projects/{{novel_id}}/workspace/reviews/review-*.md` - 历史评审

## 四层检索策略

### Layer 1: 精确匹配
- 角色名、章节号、伏笔ID、scene_id

### Layer 2: 语义检索
- 意图理解 + 相关召回

### Layer 3: 时间范围
- 最近的N章/N个场景

### Layer 4: 关系网络
- 同一场景/同一地点/同一角色/同一开放循环

## 输出要求

- 优先返回当前任务真正需要的最小上下文包
- 明确标出来源文件
- 若召回内容跨章节或跨时间跨度较大，要提示调用方注意边界

## 注意事项

1. 默认禁止跨 `novel_id` 检索。
2. `writer` 与 `prose_director` 优先拿“当前 Scene 必需上下文”，不要把整本书一次性灌入上下文。
3. 若检索结果和当前 Beat / Prose Brief 冲突，应优先交给 `critic` / `architect` 判断，而不是擅自覆盖现有产物。
