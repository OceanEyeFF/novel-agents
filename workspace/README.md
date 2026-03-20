# Writer Room - 小说工作区

> 基于 OpenClaw 的多 Agent 协作叙事生产系统

---

## 设计目标

工作区的首要原则是：**一部小说，一个独立根目录。**

这意味着：

- 不允许多部小说共享同一个 `canon/`、`states/`、`drafts/` 目录。
- 所有 Skill 在执行前都必须拿到明确的 `novel_id`。
- 所有路径解析都必须从 `projects/{{novel_id}}/workspace/` 开始。
- 如果任务上下文缺少 `novel_id`，应先补齐，再执行任何读写。

---

## 推荐目录结构

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml              # 小说元信息与发布配置
    ├── project.yaml               # 长期项目契约（题材、风格、baseline）
    ├── intake/
    │   └── latest-summary.yaml    # 对话式立项归纳
    └── workspace/
        ├── runtime/               # 当前运行态
        │   └── runtime.yaml
        ├── canon/                 # 核心设定（不可违背）
        │   ├── bible.md
        │   └── character-template.md
        ├── plans/                 # 规划
        │   ├── arc/
        │   ├── chapter/
        │   └── scene/
        ├── states/                # 动态状态
        │   ├── characters/
        │   ├── timeline/
        │   └── open_loops/
        ├── drafts/                # 草稿
        │   └── scenes/
        ├── reviews/               # 评审报告
        ├── snapshots/             # 版本快照
        ├── summaries/             # 章节摘要
        └── metrics/               # 质量指标
            ├── style/
            └── pacing/
```

---

## manifest.yaml 最小字段

每本小说至少需要一个 `projects/{{novel_id}}/manifest.yaml`：

```yaml
novel_id: an-lian-shu-lian
name: 暗恋我的女生过于熟练却没有恋爱经验
language: zh-CN
workspace_root: projects/an-lian-shu-lian/workspace
source_of_truth: github
publish:
  channel: clawskills
  package: writer-room/an-lian-shu-lian
  version: 0.2.0
```

推荐字段说明：

- `novel_id`：机器唯一标识，所有读写都依赖它。
- `workspace_root`：该小说的绝对工作区根。
- `source_of_truth`：源码真相源，默认 GitHub。
- `publish.channel`：推荐设为 `clawskills`。
- `publish.package`：发布包名，方便多书并存。
- `publish.version`：当前已发布版本。

---

## 工作流

### 0. Intake / Bootstrap 阶段
1. 通过 intake / briefing 子流程生成 `projects/{{novel_id}}/intake/latest-summary.yaml`
2. 创建 `projects/{{novel_id}}/manifest.yaml`
3. 生成 `projects/{{novel_id}}/project.yaml`
4. 初始化 `projects/{{novel_id}}/workspace/runtime/runtime.yaml`
5. 校验 `workspace_root` 与实际路径一致
6. 将 `novel_id` 写入任务上下文与运行日志

### 1. 规划阶段
1. `Orchestrator` 先读取 `runtime.stage`、`narrative_state`、`task`
2. `Architect` 创建/修改卷纲 → `projects/{{novel_id}}/workspace/plans/arc/`
2. `Architect` 创建/修改章纲 → `projects/{{novel_id}}/workspace/plans/chapter/`
3. `Scene Planner` 拆解 scene beats → `projects/{{novel_id}}/workspace/plans/scene/`

### 2. 创作阶段
1. `Character Director` 确认角色状态 → `projects/{{novel_id}}/workspace/states/characters/`
2. `Writer` 写草稿 → `projects/{{novel_id}}/workspace/drafts/scenes/`
3. `Critic` 审核 → `projects/{{novel_id}}/workspace/reviews/`

### 3. 归档阶段
1. `Archivist` 更新状态 → `projects/{{novel_id}}/workspace/states/`
2. `Archivist` 写摘要 → `projects/{{novel_id}}/workspace/summaries/`
3. `Archivist` 创建快照 → `projects/{{novel_id}}/workspace/snapshots/`

### 4. 发布阶段
1. 从 GitHub 中选定待发布 commit
2. 对目标 Skill 版本执行校验
3. 发布到 clawskills.sh
4. OpenClaw 侧安装该**已发布版本**而非仓库浮动 HEAD

---

## 文件命名规范

- 卷纲: `arc-{{卷号}}.md`
- 章纲: `chapter-{{卷}}-{{章}}.md`
- Scene: `scene-{{卷}}-{{章}}-{{序号}}.md`
- 草稿: `draft-{{卷}}-{{章}}-{{序号}}.md`
- 评审: `review-{{卷}}-{{章}}-{{序号}}.md`
- 快照: `snapshot-{{时间戳}}.zip`

建议附加前缀：

- 运行日志：`run-{{novel_id}}-{{时间戳}}.log`
- 导出包：`release-{{novel_id}}-{{version}}.tgz`

---

## 隔离规则

1. **禁止默认工作区**：不能再假设当前目录下唯一的 `novel-workspace/` 就是目标小说。
2. **禁止省略 `novel_id`**：凡是写操作都必须携带 `novel_id`。
3. **禁止跨书回填状态**：A 书的状态、伏笔、摘要不可写到 B 书。
4. **禁止用标题当主键**：标题可重复，必须使用 `novel_id`。
5. **允许共享模板，但不共享数据**：模板可以共用，实例数据必须隔离。

---

## 状态流转

```text
[对话 Intake] → [project.yaml / runtime.yaml] → [章纲] → [Scene Beats] → [草稿] → [评审] → [归档] → [发布]
                                                           ↓
                                                     [需修改] → 回到 Scene Beats / Bootstrap
```

## 状态契约建议

- `project.yaml`：保存题材、平台、目标读者、风格模板、baseline、禁区。
- `workspace/runtime/runtime.yaml`：保存 `narrative_state`、`task`、`runtime` 三段。
- `intake/latest-summary.yaml`：保存 confirmed / unresolved / 决策权限，用于回看为什么允许进入中台。
