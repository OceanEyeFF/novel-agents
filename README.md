# Writer Room - 多 Agent 协作小说生产系统

> 基于 OpenClaw 架构的多 Agent 协作叙事生产系统

## 🎭 Agent 阵容

| Agent | 名称 | 职责 |
|-------|------|------|
| **architect** | 架构师 | 维护全书弧光、卷纲、章纲 |
| **character-director** | 角色导演 | 维护角色状态机、追踪角色动态 |
| **scene-planner** | 场景策划 | 把章纲拆成 Scene Beats |
| **writer** | 写手 | 根据 Scene Beats 写正文 |
| **critic** | 审稿 | 检查 OOC、Canon 冲突、文风 |
| **archivist** | 档案官 | 更新状态、写摘要、管理快照 |

## 📖 作品展示

本系统已完成的作品：**《暗恋我的女生过于熟练却没有恋爱经验》**

- 第1卷：5章，约 19,000字
- 状态：完结 ✓

## 🛠️ 系统架构

```
novel-agents/
├── skills/              # 7个专用 Skills
│   ├── novel-canon/    # 核心设定管理
│   ├── novel-plan/     # 规划管理
│   ├── novel-state/    # 状态管理
│   ├── novel-verify/  # 质量验证
│   ├── novel-draft/   # 创作辅助
│   ├── novel-retrieve/ # 记忆检索
│   └── novel-snapshot/ # 版本快照
├── workspace/          # 工作区
│   ├── canon/         # 核心设定
│   ├── plans/         # 规划
│   ├── states/       # 动态状态
│   ├── drafts/       # 草稿
│   ├── reviews/      # 评审
│   └── summaries/    # 摘要
└── docs/             # 文档
```

## 🚀 使用方式

1. 配置 OpenClaw Agent
2. 导入 Skills
3. 创建 Canon Bible
4. 按工作流：规划 → 创作 → 审核 → 归档

## 📝 工作流

```
Architect 创建章纲 → Scene Planner 拆解 Beats → Writer 写正文 → Critic 审核 → Archivist 归档
```

## 📄 许可证

MIT

---

**Author**: OceanEyeFF  
**系统**: OpenClaw Multi-Agent
