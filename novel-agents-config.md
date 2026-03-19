# Writer Room Agent 配置 v0.1

> 每个 agent 的角色定义 + 工具权限

---

## 1. Architect（总规划师）

```yaml
name: architect
role: 总规划师
description: 维护全书弧光、卷纲、章纲，管控主题/高潮/节奏分布

# 可读权限
read:
  - canon/*          # 全部 canon 设定
  - plans/arc/*      # 卷纲
  - plans/chapter/*  # 章纲
  - states/characters/*   # 角色状态
  - states/timeline/*     # 时间线
  - states/open_loops/*  # 伏笔表

# 可写权限
write:
  - plans/chapter/*  # 创建/修改章纲
  - plans/arc/*      # 创建/修改卷纲

# 禁止操作
deny:
  - drafts/*         # 禁止写正文
  - reviews/*        # 禁止写评审
  - canon/create    # 禁止新建核心设定
```

---

## 2. Character Director（角色导演）

```yaml
name: character_director
role: 角色导演
description: 维护角色状态机，输出角色行动约束

read:
  - states/characters/*   # 角色状态
  - plans/chapter/current # 当前章节规划
  - canon/characters/*    # 角色基础设定

write:
  - states/characters/*   # 更新角色状态

deny:
  - drafts/*
  - plans/chapter/*       # 不允许改章纲
  - canon/*              # 不允许改基础设定
```

---

## 3. Scene Planner（场景策划）

```yaml
name: scene_planner
role: 场景策划
description: 把章纲拆成 scene beats，控制每个场景的结构

read:
  - plans/chapter/current      # 当前章纲
  - states/characters/current  # 当前章节涉及的角色状态
  - states/open_loops/*       # 伏笔表

write:
  - plans/scene/*              # 创建 scene beats

deny:
  - drafts/*
  - states/*                  # 不允许改状态
  - canon/*                   # 不允许改设定
```

---

## 4. Writer（正文写手）

```yaml
name: writer
role: 正文写手
description: 根据 scene beats 写 prose，保持局部文风

read:
  - plans/scene/current              # 当前场景 beats
  - states/characters/current        # 当前角色状态摘要
  - drafts/scenes/previous          # 上一场景（参考风格）
  - canon/style/*                    # 文风模板

write:
  - drafts/scenes/*                 # 写场景草稿

deny:
  - states/*                        # 禁止改状态
  - plans/*                         # 禁止改规划
  - canon/*                         # 禁止改设定
```

---

## 5. Critic（批评审校）

```yaml
name: critic
role: 批评审校
description: 检查逻辑/节奏/OOC/设定冲突，输出结构化评审

read:
  - drafts/scenes/current     # 待审场景
  - states/characters/*       # 角色状态（对照 OOC）
  - states/open_loops/*       # 伏笔表（检查泄露）
  - canon/*                   # 设定（检查冲突）

write:
  - reviews/*                 # 写评审报告

deny:
  - drafts/*                  # 禁止改正文
  - states/*                  # 禁止改状态
  - plans/*                   # 禁止改规划
```

---

## 6. Archivist（档案官）

```yaml
name: archivist
role: 档案官
description: 场景结束后更新状态层，写摘要，更新索引

read:
  - drafts/scenes/approved    # 已批准的正文
  - states/*                  # 当前全部状态

write:
  - states/characters/*       # 更新角色状态
  - states/timeline/*         # 更新时间线
  - states/open_loops/*       # 更新伏笔表
  - summaries/chapters/*     # 写章节摘要

deny:
  - drafts/*                  # 禁止写正文
  - plans/*                   # 禁止改规划
  - canon/*                   # 禁止改设定
```

---

## 7. Orchestrator（调度员）

```yaml
name: orchestrator
role: 调度员
description: 决定调用谁，监控质量，聚合结果

read:
  - all   # 读取全部（做路由决策）

write:
  - logs/*                    # 写执行日志

deny:
  - drafts/*                  # 禁止写正文
  - reviews/*                 # 禁止写评审
  - states/*                  # 禁止直接改状态
  - canon/*                   # 禁止改设定
```

---

## 工具映射表

| 工具 | architect | character_director | scene_planner | writer | critic | archivist |
|------|-----------|-------------------|---------------|--------|--------|-----------|
| get_canon | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| get_character_state | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| update_character_state | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| get_timeline_window | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| register_plot_point | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ |
| get_open_loops | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| create_chapter_plan | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| create_scene_beats | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| check_canon_conflict | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| check_character_consistency | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| check_style_drift | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| retrieve_relevant_memories | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| snapshot_scene | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 下一步

这个配置需要翻译成 OpenClaw 的实际 agent 配置格式。你知道 OpenClaw 的 agent 配置具体怎么写吗（比如 yaml/json 格式、tool allowlist 字段名）？

或者我先查一下文档？
