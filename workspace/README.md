# Writer Room - 小说工作区

> 基于 OpenClaw 的多 Agent 协作叙事生产系统

---

## 设计目标

工作区的首要原则是：**一部小说，一个独立根目录。**

这意味着：

- 不允许多部小说共享同一个 `canon/`、`states/`、`drafts/` 目录。
- 所有 Agent / Skill 在执行前都必须拿到明确的 `novel_id`。
- 所有路径解析都必须从 `projects/{{novel_id}}/workspace/` 开始。
- 如果任务上下文缺少 `novel_id`，应先补齐，再执行任何读写。
- `briefing` / `bootstrap` 当前默认作为 **preflight capability**，先负责准入和状态契约，而不是直接算入常驻执行层 Agent。

---

## 推荐目录结构

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml                    # 小说元信息与发布配置
    ├── project.yaml                     # 长期项目契约（题材、风格、baseline）
    ├── intake/
    │   └── latest-summary.yaml          # 对话式立项归纳
    └── workspace/
        ├── runtime/                     # 当前运行态
        │   └── runtime.yaml
        ├── canon/                       # 核心设定（不可违背）
        │   ├── bible.md
        │   ├── style/
        │   └── characters/
        ├── plans/                       # 规划与表达方案
        │   ├── arc/
        │   ├── chapter/
        │   ├── scene/
        │   └── prose/                   # Prose Brief
        ├── states/                      # 动态状态
        │   ├── characters/
        │   ├── timeline/
        │   └── open_loops/
        ├── drafts/                      # 草稿
        │   ├── scenes/
        │   └── bridges/
        ├── reviews/                     # 评审报告 / Guard 报告
        ├── summaries/                   # 章节摘要
        ├── snapshots/                   # 版本快照
        ├── logs/                        # 编排日志
        ├── jobs/                        # 状态机任务记录
        ├── manifests/                   # 流转清单、角色约束等
        └── metrics/                     # 质量指标
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

## 当前执行层 Agent（v0.2）

1. `orchestrator`：读取 `runtime.yaml`，推进状态机、安排返工与升级人工。
2. `architect`：维护全书弧光、卷纲、章纲。
3. `character_director`：维护角色约束、情绪边界与行动一致性。
4. `scene_planner`：把章纲拆成可执行的 Scene Beats。
5. `prose_director`：把 Scene Beats 翻译成 Prose Brief，定义叙事重心和慢写点。
6. `writer`：先写 `skeleton draft`，再做 `prose expansion` / `bridge`。
7. `critic`：先做结构闸门，再做内容审核与小说性评分。
8. `archivist`：在通过后更新状态、摘要和快照。

补充说明：`briefing` 与 `bootstrap` 仍属于入口能力，不建议在未验证前直接升格为常驻 Room Agents。

---

## 工作流

### 0. Intake / Bootstrap 阶段
1. `briefing capability` 归纳作者输入，生成 `projects/{{novel_id}}/intake/latest-summary.yaml`。
2. 创建 `projects/{{novel_id}}/manifest.yaml`。
3. `bootstrap capability` 生成 `projects/{{novel_id}}/project.yaml`。
4. 初始化 `projects/{{novel_id}}/workspace/runtime/runtime.yaml`。
5. 校验 `workspace_root`、`novel_id` 与实际路径一致。
6. 将 `novel_id` 写入任务上下文与运行日志。

### 1. 规划阶段
1. `orchestrator` 读取 `runtime.stage`、`narrative_state`、`task`，决定下一跳。
2. `architect` 创建/修改卷纲 → `projects/{{novel_id}}/workspace/plans/arc/`。
3. `architect` 创建/修改章纲 → `projects/{{novel_id}}/workspace/plans/chapter/`。
4. `scene_planner` 拆解 Scene Beats → `projects/{{novel_id}}/workspace/plans/scene/`。
5. `character_director` 输出角色约束 / 情绪边界 → `projects/{{novel_id}}/workspace/manifests/character_constraints/`。
6. `prose_director` 生成 Prose Brief → `projects/{{novel_id}}/workspace/plans/prose/`。

### 2. 创作阶段
1. `writer` 先写 `skeleton draft` → `projects/{{novel_id}}/workspace/drafts/scenes/`。
2. `writer` 再执行 `prose expansion`，必要时写 `bridge draft` → `projects/{{novel_id}}/workspace/drafts/bridges/`。
3. `novel-guard` / `critic` 在送审前后检查结构完整性、目标归属与审批链闭环。

### 3. 审核与组章阶段
1. `critic` 先执行 Gate A（结构完整性），再执行 Gate B（内容质量）与 Gate C（小说性评分） → `projects/{{novel_id}}/workspace/reviews/`。
2. `orchestrator` 根据 `required_action` 决定是 `expand_scene`、`rewrite_scene`、`bridge_scenes`、`replan_scene` 还是 `replan_chapter`。
3. Scene 全部通过后，`orchestrator` 进入 `chapter_assembling`，组织章节过桥、余波段和章节级连贯性检查。

### 4. 归档阶段
1. `archivist` 在 `decision = approved` 且 `gate_a_result = pass` 后更新 `states/`。
2. `archivist` 写章节摘要 → `projects/{{novel_id}}/workspace/summaries/`。
3. `archivist` 创建快照 → `projects/{{novel_id}}/workspace/snapshots/`。
4. `orchestrator` 记录任务结果到 `logs/`、`jobs/` 与 `manifests/`。

### 5. 发布阶段
1. 从 GitHub 中选定待发布 commit。
2. 对目标 Skill 版本执行校验。
3. 发布到 clawskills.sh。
4. OpenClaw 侧安装该**已发布版本**而非仓库浮动 HEAD。

---

## 文件命名规范

- 卷纲: `arc-{{卷号}}.md`
- 章纲: `chapter-{{卷}}-{{章}}.md`
- Scene Beat: `scene-{{卷}}-{{章}}-{{序号}}.md`
- Prose Brief: `prose-{{卷}}-{{章}}-{{序号}}.md`
- 场景草稿: `draft-{{卷}}-{{章}}-{{序号}}.md`
- Bridge 草稿: `bridge-{{卷}}-{{章}}-{{序号A}}-{{序号B}}.md`
- 评审: `review-{{卷}}-{{章}}-{{序号}}.md`
- Guard 报告: `guard-{{卷}}-{{章}}-{{序号}}.md`
- 快照: `snapshot-{{类型}}-{{ID}}-{{YYYYMMDD}}.md`

建议附加前缀：

- 运行日志：`run-{{novel_id}}-{{时间戳}}.log`
- 任务状态：`job-{{chapter_id}}-{{scene_id}}.yaml`
- 导出包：`release-{{novel_id}}-{{version}}.tgz`

---

## 隔离规则

1. **禁止默认工作区**：不能再假设当前目录下唯一的 `novel-workspace/` 就是目标小说。
2. **禁止省略 `novel_id`**：凡是写操作都必须携带 `novel_id`。
3. **禁止跨书回填状态**：A 书的状态、伏笔、摘要不可写到 B 书。
4. **禁止用标题当主键**：标题可重复，必须使用 `novel_id`、`chapter_id`、`scene_id`。
5. **允许共享模板，但不共享数据**：模板可以共用，实例数据必须隔离。
6. **禁止混合文档归档**：正文、摘要、评审、下章预告不能混写到同一个交付文件。
7. **无批准不得归档**：没有 `gate_a_result = pass` 与 `decision = approved` 时，`archivist` 不得入档。

---

## 状态流转

```text
[idea_raw]
→ [brief_partial]
→ [brief_ready]
→ [planned]
→ [beat_ready]
→ [prose_ready]
→ [drafting]
→ [review_failed_minor / review_failed_major / replan_required]
→ [scene_approved]
→ [chapter_assembling]
→ [chapter_approved]
→ [archived]
```

返工原则：

- `expand_scene` / `fix_local_language` / `bridge_scenes` → 回到 `writer`
- `rewrite_scene` / `replan_scene` → 回到 `scene_planner`
- `replan_chapter` → 回到 `architect`
- `missing_project_contract` / `missing_runtime_contract` → 回到 `bootstrap`

---

## 状态契约建议

- `project.yaml`：保存题材、平台、目标读者、风格模板、baseline、禁区。
- `workspace/runtime/runtime.yaml`：保存 `narrative_state`、`task`、`runtime` 三段。
- `intake/latest-summary.yaml`：保存 confirmed / unresolved / 决策权限，用于回看为什么允许进入中台。
- `plans/prose/*.md`：保存 Prose Brief，包括 `emotional_core`、`slow_beats`、`sensory_targets`、`paragraph_rhythm` 等字段。
- `reviews/*.md`：至少保存 `gate_a_result`、`gate_b_result`、`scene_density`、`novelness`、`chapter_cohesion`、`decision`、`required_action`。
