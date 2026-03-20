# Writer Room - 小说工作区

> 基于 OpenClaw 的多 Agent 协作叙事生产系统

---

## 目录结构

```
novel-workspace/
├── canon/                    # 核心设定（不可违背）
│   ├── bible.md             # Canon Bible
│   └── character-template.md # 角色模板
├── plans/                   # 规划
│   ├── arc/                # 卷纲
│   │   └── arc-template.md
│   ├── chapter/            # 章纲
│   │   └── chapter-template.md
│   └── scene/              # Scene Beats
│       └── scene-beat-template.md
├── states/                 # 动态状态
│   ├── characters/        # 角色状态
│   │   └── index.md
│   ├── timeline/           # 时间线
│   │   └── index.md
│   └── open_loops/        # 伏笔/承诺
│       └── index.md
├── drafts/                 # 草稿
│   └── scenes/
├── reviews/                # 评审报告
├── snapshots/              # 版本快照
├── summaries/              # 章节摘要
│   └── chapter-template.md
└── metrics/                # 质量指标
    ├── style/
    └── pacing/
```

---

## 工作流

### 1. 规划阶段
1. `Architect` 创建/修改卷纲 → `plans/arc/`
2. `Architect` 创建/修改章纲 → `plans/chapter/`
3. `Scene Planner` 拆解 scene beats → `plans/scene/`

### 2. 创作阶段
1. `Character Director` 确认角色状态 → `states/characters/`
2. `Writer` 写草稿 → `drafts/scenes/`
3. `Critic` 审核 → `reviews/`

### 3. 归档阶段
1. `Archivist` 更新状态 → `states/`
2. `Archivist` 写摘要 → `summaries/`
3. `Archivist` 创建快照 → `snapshots/`

---

## 文件命名规范

- 卷纲: `arc-{{卷号}}.md`
- 章纲: `chapter-{{卷}}-{{章}}.md`
- Scene: `scene-{{卷}}-{{章}}-{{序号}}.md`
- 草稿: `draft-{{卷}}-{{章}}-{{序号}}.md`
- 评审: `review-{{卷}}-{{章}}-{{序号}}.md`
- 快照: `snapshot-{{时间戳}}.zip`

---

## 状态流转

```
[章纲] → [Scene Beats] → [草稿] → [评审] → [归档] → [发布]
                            ↓
                      [需修改] → 回到 Scene Beats
```
