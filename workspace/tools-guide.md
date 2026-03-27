# Writer Room - 完整工具指南

> 小说创作系统的 Agent × Skill × 工作区读写约定

---

## 🛠️ Skill / Capability 列表

| Skill / Capability | 名称 | 功能 | 主要使用方 |
|--------------------|------|------|------------|
| `briefing` | 🧭 立项澄清 | 归纳 confirmed / unresolved / decision rights，判断是否满足准入条件 | 入口服务 / orchestrator |
| `bootstrap` | 🧱 状态契约启动 | 生成 `project.yaml` / `runtime.yaml` / 初始任务上下文 | 入口服务 / orchestrator |
| `novel-canon` | 📖 核心设定 | 读取/管理 Canon Bible、角色基础设定 | architect, character_director, critic |
| `novel-plan` | 📋 规划管理 | 创建卷纲、章纲、Scene Beats | architect, scene_planner |
| `novel-state` | 🔄 状态管理 | 角色动态状态、时间线、伏笔追踪 | character_director, archivist |
| `novel-verify` | 🔍 质量验证 | Gate A / Gate B、OOC、Canon 冲突、文本密度与小说性检查 | critic |
| `novel-guard` | 🛡️ 放行闸门 | 防串章、混稿、标题错配、审批链缺口 | writer, critic, archivist |
| `novel-draft` | ✍️ 创作辅助 | skeleton、续写、过渡、桥接场景 | writer |
| `novel-novelize` | ✨ 小说化表达 | 生成 Prose Brief、做扩写、降低剧本化风险 | prose_director, writer |
| `novel-retrieve` | 🔎 记忆检索 | 从上下文中检索相关信息 | writer, critic, orchestrator |
| `novel-snapshot` | 📸 版本快照 | 创建/回滚/管理版本 | archivist |

> `briefing` / `bootstrap` 当前仍建议作为 **preflight capabilities** 验证，不默认计入 8 个常驻执行层 Agent。

---

## 🔐 先决条件：必须先确定 novel_id

所有 Skill 在执行前都必须明确以下信息：

- `novel_id`
- `workspace_root`
- `project.yaml` 是否存在
- `runtime.yaml` 是否存在
- 当前 `chapter_id` / `scene_id`（如适用）
- 当前 `runtime.stage` 与任务目标（规划 / prose / drafting / review / archive）

推荐解析方式：

1. 读取 `projects/{{novel_id}}/manifest.yaml`
2. 校验 `projects/{{novel_id}}/project.yaml` 与 `projects/{{novel_id}}/workspace/runtime/runtime.yaml` 已就绪
3. 从 `workspace_root` 解析实际路径
4. 所有后续读写都使用绝对根：`projects/{{novel_id}}/workspace/`
5. 若任务涉及场景写作，额外确认 `chapter_id`、`scene_id`、`source_beat`、`source_prose_brief`

如果缺少上述任一项，**应暂停任务，不得继续写文件**。

---

## 📍 文件位置

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml
    ├── project.yaml
    ├── intake/latest-summary.yaml
    └── workspace/
        ├── runtime/runtime.yaml
        ├── canon/
        │   ├── bible.md                    # Canon Bible
        │   ├── style/                      # 文风模板 / 风格基准
        │   └── characters/                 # 角色设定
        ├── plans/
        │   ├── arc/                        # 卷纲
        │   ├── chapter/                    # 章纲
        │   ├── scene/                      # Scene Beats
        │   └── prose/                      # Prose Brief
        ├── states/
        │   ├── characters/                 # 角色状态
        │   ├── timeline/                   # 时间线
        │   └── open_loops/                 # 伏笔表
        ├── drafts/
        │   ├── scenes/                     # Scene skeleton / expanded draft
        │   └── bridges/                    # Scene 间桥接稿
        ├── reviews/                        # Critic / Guard 报告
        ├── summaries/                      # 章节摘要
        ├── snapshots/                      # 快照
        ├── logs/                           # 编排日志
        ├── jobs/                           # 状态机任务记录
        └── manifests/                      # 角色约束 / 流转清单
```

---

## 🔄 完整工作流

### 0. 准入阶段

```text
⓪ briefing capability 归纳作者自然语言
   写 projects/{{novel_id}}/intake/latest-summary.yaml

⓪-1 bootstrap capability 生成状态契约
   写 projects/{{novel_id}}/project.yaml
   写 projects/{{novel_id}}/workspace/runtime/runtime.yaml
```

### 1. 规划阶段

```text
① Orchestrator 读取 runtime.stage 并判断路由

② Architect 创建卷纲
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/arc/arc-{{卷号}}.md

③ Architect 创建章纲
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/chapter/chapter-{{卷}}-{{章}}.md

④ Scene Planner 拆解 Scene Beats
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md

⑤ Character Director 输出角色约束
   novel-state → 读写 projects/{{novel_id}}/workspace/states/characters/
   写 projects/{{novel_id}}/workspace/manifests/character_constraints/{{scene_id}}.md

⑥ Prose Director 生成 Prose Brief
   novel-novelize → 写 projects/{{novel_id}}/workspace/plans/prose/prose-{{ID}}.md
```

### 2. 创作阶段

```text
⑦ Writer 获取上下文
   novel-canon → 读取角色设定 / 风格模板
   novel-plan → 读取当前 Scene Beats
   novel-state → 读取角色动态状态
   novel-retrieve → 检索相关记忆
   novel-novelize → 读取 Prose Brief

⑧ Writer 写 Skeleton Draft
   novel-draft → 写 projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md

⑨ Writer 做 Prose Expansion / Bridge
   novel-draft + novel-novelize → 更新 drafts/scenes/ 或写 drafts/bridges/

⑩ novel-guard 做送审前边界检查
   novel-guard → 写 projects/{{novel_id}}/workspace/reviews/guard-{{ID}}.md
```

### 3. 审核阶段

```text
⑪ Critic 审核
   novel-verify → 先做 Gate A，再做 Gate B / Gate C
   写 projects/{{novel_id}}/workspace/reviews/review-{{ID}}.md

⑫ 根据评审结果：
   - 通过 → ⑬ 组章 / 归档
   - required_action = expand_scene / bridge_scenes / fix_local_language → 回到 Writer
   - required_action = rewrite_scene / replan_scene → 回到 Scene Planner
   - required_action = replan_chapter → 回到 Architect
   - required_action = rhythm_revise 且 emotion_wave_health 不佳、其余核心指标可接受 → 进入 `rhythm_revise_pending`，回到 Writer 做节奏微调
   - `rhythm_revise_pending` 完成后进入 `rhythm_revise_review`，由 Critic 做针对性节奏复审
   - rhythm 复审通过且无 scene 增减 → Orchestrator 可直接合并 `plans/rhythm-revised/` 增量产物
```

### 4. 组章与归档阶段

```text
⑬ Orchestrator 组织 chapter assembly
   检查 scene 间桥接、余波段、章节级连贯性

⑭ Archivist 归档
   novel-state → 更新角色状态 / 时间线 / 伏笔表

⑮ 写章节摘要
   写 projects/{{novel_id}}/workspace/summaries/chapter-{{ID}}.md

⑯ 创建快照
   novel-snapshot → 创建章节快照
```

### 5. 发布阶段

```text
⑰ 选择待发布 Git commit
⑱ 在 CI / 本地校验 Skill 包内容
⑲ 发布到 clawskills.sh
⑳ OpenClaw 安装指定版本并验证
```

---

## 📋 Agent × Skill 映射

| Agent | 主要使用 Skill / Capability |
|-------|------------------------------|
| **Orchestrator** | briefing, bootstrap, novel-retrieve |
| **Architect** | novel-plan, novel-canon, novel-retrieve |
| **Character Director** | novel-state, novel-canon |
| **Scene Planner** | novel-plan, novel-state |
| **Prose Director** | novel-novelize, novel-retrieve, novel-canon |
| **Writer** | novel-draft, novel-novelize, novel-retrieve, novel-canon, novel-plan |
| **Critic** | novel-verify, novel-guard, novel-canon, novel-state |
| **Archivist** | novel-state, novel-snapshot, novel-guard |

---

## 🧭 状态机与交接点

| 状态 | 说明 | 默认下一步 |
|------|------|------------|
| `idea_raw` | 想法仍模糊 | `briefing` |
| `brief_partial` | 已有部分约束 | `briefing` |
| `brief_ready` | 准入信息齐备 | `bootstrap` |
| `planned` | 章纲已就绪 | `scene_planner` |
| `beat_ready` | Scene Beats 已批准 | `prose_director` |
| `prose_ready` | Prose Brief 已就绪 | `writer` |
| `drafting` | Writer 正在写作 | `novel-guard` / `critic` |
| `review_failed_minor` | 局部语言 / 密度问题 | `writer` |
| `review_failed_major` | 结构或关键质量问题 | `scene_planner` / `writer` |
| `replan_required` | 章级规划出错 | `architect` |
| `scene_approved` | 场景已批准 | `chapter assembly` |
| `chapter_approved` | 章节已批准 | `archivist` |
| `archived` | 已更新状态并快照 | `publish` |

---

## 🔍 检索优先级

当需要获取上下文时，按以下顺序：

1. **固定层** - Canon Bible、风格模板、当前 Scene Beats、当前 Prose Brief（始终读取）
2. **动态状态层** - 角色状态、时间线、开放循环、角色约束（按需读取）
3. **内容层** - 最近场景稿、bridge 稿、章节摘要（按需读取）
4. **历史层** - 早期章节、旧快照、历史评审（必要时召回）

前提条件：上述检索都只能在**当前 `novel_id` 对应工作区内部**进行，禁止默认跨书扫描。

---

## ⚡ 常见操作速查

### 读取 manifest
```bash
read: projects/{{novel_id}}/manifest.yaml
```

### 读取 project contract
```bash
read: projects/{{novel_id}}/project.yaml
```

### 读取 runtime contract
```bash
read: projects/{{novel_id}}/workspace/runtime/runtime.yaml
```

### 读取当前 Scene Beats
```bash
read: projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 读取当前 Prose Brief
```bash
read: projects/{{novel_id}}/workspace/plans/prose/prose-{{ID}}.md
```

### 查询角色当前状态
```bash
read: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 查询伏笔情况
```bash
read: projects/{{novel_id}}/workspace/states/open_loops/index.md
```

### 查询时间线
```bash
read: projects/{{novel_id}}/workspace/states/timeline/index.md
```

### 读取 Canon
```bash
read: projects/{{novel_id}}/workspace/canon/bible.md
read: projects/{{novel_id}}/workspace/canon/characters/{{角色名}}.md
```

### 读取风格模板
```bash
read: projects/{{novel_id}}/workspace/canon/style/{{style_id}}.md
```

### 读取最近正文
```bash
read: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 写新 Scene Beats
```bash
write: projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 写 Prose Brief
```bash
write: projects/{{novel_id}}/workspace/plans/prose/prose-{{ID}}.md
```

### 写正文草稿
```bash
write: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 写桥接草稿
```bash
write: projects/{{novel_id}}/workspace/drafts/bridges/bridge-{{ID-A}}-{{ID-B}}.md
```

### 更新角色状态
```bash
edit: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 创建 Guard 报告
```bash
write: projects/{{novel_id}}/workspace/reviews/guard-{{ID}}.md
```

### 创建评审报告
```bash
write: projects/{{novel_id}}/workspace/reviews/review-{{ID}}.md
```

---

## 📊 状态流转

```text
[Intake Summary]
→ [project.yaml / runtime.yaml]
→ [Chapter Plan]
→ [Scene Beats]
→ [Prose Brief]
→ [Skeleton Draft]
→ [Prose Expansion / Bridge]
→ [Guard]
→ [Review]
→ [Chapter Assembly]
→ [Archive]
→ [Snapshot]
→ [Publish]
                 ↓
      [expand_scene / rewrite_scene / replan_scene / replan_chapter]
```

---

## 🎯 关键原则

1. **Canon 不可违背** - 任何创作都不能违背核心设定。
2. **先规划后 prose** - Scene Beat 未批准前，不应直接进入正文写作。
3. **先 skeleton 后 expansion** - 先保证事件正确，再补足小说性与密度。
4. **评审必须先过 Gate A** - 结构不完整时，不允许进入内容审核和归档。
5. **Guard 不是可选项** - 送审前与归档前都要阻断串章、混稿、错配文档。
6. **工作区必须隔离** - 每本小说单独 `workspace_root`。
7. **发布优先于直连同步** - 生产环境优先安装 clawskills.sh 已发布版本。
