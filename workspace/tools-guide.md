# Writer Room - 完整工具指南

> 小说创作系统的所有工具汇总

---

## 🛠️ 工具列表

| Skill | 名称 | 功能 |
|-------|------|------|
| `novel-canon` | 📖 核心设定 | 读取/管理 Canon Bible、角色基础设定 |
| `novel-plan` | 📋 规划管理 | 创建卷纲、章纲、Scene Beats |
| `novel-state` | 🔄 状态管理 | 角色动态状态、时间线、伏笔追踪 |
| `novel-verify` | 🔍 质量验证 | OOC、Canon 冲突，信息泄露、文风漂移检查 |
| `novel-draft` | ✍️ 创作辅助 | 续写、风格参考、过渡生成 |
| `novel-retrieve` | 🔎 记忆检索 | 从上下文中检索相关信息 |
| `novel-snapshot` | 📸 版本快照 | 创建/回滚/管理版本 |

---

## 🔐 先决条件：必须先确定 novel_id

所有 Skill 在执行前都必须明确以下信息：

- `novel_id`
- `workspace_root`
- 当前 `chapter_id` / `scene_id`（如适用）

推荐解析方式：

1. 读取 `projects/{{novel_id}}/manifest.yaml`
2. 从 `workspace_root` 解析实际路径
3. 所有后续读写都使用绝对根：`projects/{{novel_id}}/workspace/`

如果缺少上述任一项，**应暂停任务，不得继续写文件**。

---

## 📍 文件位置

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml
    └── workspace/
        ├── canon/                    # 核心设定
        │   ├── bible.md             # Canon Bible
        │   └── characters/          # 角色设定
        ├── plans/                   # 规划
        │   ├── arc/                 # 卷纲
        │   ├── chapter/             # 章纲
        │   └── scene/               # Scene Beats
        ├── states/                  # 动态状态
        │   ├── characters/          # 角色状态
        │   ├── timeline/            # 时间线
        │   └── open_loops/          # 伏笔表
        ├── drafts/scenes/           # 草稿
        ├── reviews/                 # 评审报告
        ├── summaries/               # 章节摘要
        └── snapshots/               # 快照
```

---

## 🔄 完整工作流

### 1. 规划阶段

```text
① Architect 创建卷纲
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/arc/arc-{{卷号}}.md

② Architect 创建章纲
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/chapter/chapter-{{卷}}-{{章}}.md

③ Scene Planner 拆解 Scene Beats
   novel-plan → 写 projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 2. 创作阶段

```text
④ Writer 获取上下文
   novel-canon → 读取角色设定
   novel-plan → 读取当前 Scene Beats
   novel-state → 读取角色动态状态
   novel-retrieve → 检索相关记忆

⑤ Writer 写正文
   novel-draft → 创作/续写/过渡
   写 projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md

⑥ Writer 自检
   novel-verify → 初步检查 OOC/Canon
```

### 3. 审核阶段

```text
⑦ Critic 审核
   novel-verify → 完整评审
   写 projects/{{novel_id}}/workspace/reviews/review-{{ID}}.md

⑧ 根据评审结果：
   - 通过 → ⑨ 归档
   - 需修改 → ④ 重新创作
   - 需重写 → ⑤ 重新创作
```

### 4. 归档阶段

```text
⑨ Archivist 归档
   novel-state → 更新角色状态
   novel-state → 更新时间线
   novel-state → 更新伏笔表

⑩ 写章节摘要
   写 projects/{{novel_id}}/workspace/summaries/chapter-{{ID}}.md

⑪ 创建快照
   novel-snapshot → 创建章节快照
```

### 5. 发布阶段

```text
⑫ 选择待发布 Git commit
⑬ 在 CI / 本地校验 Skill 包内容
⑭ 发布到 clawskills.sh
⑮ OpenClaw 安装指定版本并验证
```

---

## 📋 Agent × Skill 映射

| Agent | 主要使用 Skill |
|-------|----------------|
| **Architect** | novel-plan, novel-canon |
| **Character Director** | novel-state, novel-canon |
| **Scene Planner** | novel-plan, novel-state |
| **Writer** | novel-draft, novel-retrieve, novel-canon, novel-plan |
| **Critic** | novel-verify, novel-canon, novel-state |
| **Archivist** | novel-state, novel-snapshot |

---

## 🔍 检索优先级

当需要获取上下文时，按以下顺序：

1. **固定层** - Canon Bible、当前 Scene Beats（始终读取）
2. **动态状态层** - 角色状态、时间线（按需读取）
3. **内容层** - 章节摘要、最近场景（按需读取）
4. **历史层** - 早期章节、伏笔（必要时召回）

前提条件：上述检索都只能在**当前 `novel_id` 对应工作区内部**进行，禁止默认跨书扫描。

---

## ⚡ 常见操作速查

### 读取 manifest
```bash
read: projects/{{novel_id}}/manifest.yaml
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

### 读取当前 Scene Beats
```bash
read: projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 读取最近正文
```bash
read: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 写新 Scene Beats
```bash
write: projects/{{novel_id}}/workspace/plans/scene/scene-{{ID}}.md
```

### 写正文草稿
```bash
write: projects/{{novel_id}}/workspace/drafts/scenes/draft-{{ID}}.md
```

### 更新角色状态
```bash
edit: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 创建评审报告
```bash
write: projects/{{novel_id}}/workspace/reviews/review-{{ID}}.md
```

---

## 📊 状态流转

```text
[章纲] → [Scene Beats] → [草稿] → [评审] → [归档] → [快照] → [发布]
                            ↓
                      [需修改] → 回到 Scene Beats
                            ↓
                      [需重写] → 重新创作
```

---

## 🎯 关键原则

1. **Canon 不可违背** - 任何创作都不能违背核心设定
2. **状态必须同步** - 场景/章节结束后立即更新状态
3. **评审必须独立** - Critic 和 Writer 要分开
4. **快照要及时** - 重要节点必须创建快照
5. **检索要精准** - 先精确后扩大范围
6. **工作区必须隔离** - 每本小说单独 `workspace_root`
7. **发布优先于直连同步** - 生产环境优先安装 clawskills.sh 已发布版本
