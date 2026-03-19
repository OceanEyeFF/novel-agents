# Writer Room - 多 Agent 协作小说生产系统

> 基于 OpenClaw 架构的多 Agent 协作叙事生产系统

## 🎭 Agent 阵容

| Agent | 名称 | 职责 |
|-------|------|------|
| **architect** | 架构师 | 维护全书弧光、卷纲、章纲 |
| **character-director** | 角色导演 | 维护角色状态机、追踪角色动态 |
| **scene-planner** | 场景策划 | 把章纲拆成 Scene Beats |
| **writer** | 写手 | 根据 Scene Beats 写正文 |
| **critic** | 审稿 | 检查结构错误、OOC、Canon 冲突、文风 |
| **archivist** | 档案官 | 更新状态、写摘要、管理快照 |

## 🧰 Skills 组成

当前仓库包含 8 个专用 Skills：

- `novel-canon`：核心设定查询
- `novel-plan`：卷纲 / 章纲 / Scene Beats 规划
- `novel-state`：角色状态、时间线、伏笔状态
- `novel-verify`：质量与结构验证
- `novel-guard`：发布前边界校验与审批闸门
- `novel-draft`：正文创作辅助
- `novel-retrieve`：长短期记忆检索
- `novel-snapshot`：版本快照与回滚

## ⚠️ 当前版本重点改进

本轮文档强化主要解决以下高风险问题：

1. **章节身份漂移**：正文文件名、文档头、Scene ID、章纲 ID 必须一致。
2. **串章/混章**：一个场景稿只能服务一个 `chapter_id`，禁止把多个章节的 Scene 混入同一文档。
3. **错误通过审核**：Critic 不再只检查文风和设定，还必须先做结构完整性闸门；未过闸门不得标记“通过”。
4. **归档污染**：Archivist 只能归档带有明确批准标记且审批信息完整的稿件。

## 📖 作品展示

本系统已完成的作品：**《暗恋我的女生过于熟练却没有恋爱经验》**

- 第1卷：5章，约 19,000字
- 状态：完结 ✓

## 🛠️ 系统架构

```text
novel-agents/
├── agents/              # 6个岗位型 Agent 说明
├── skills/              # 8个专用 Skills
│   ├── novel-canon/
│   ├── novel-plan/
│   ├── novel-state/
│   ├── novel-verify/
│   ├── novel-guard/
│   ├── novel-draft/
│   ├── novel-retrieve/
│   └── novel-snapshot/
├── workspace/           # 项目工作区
└── docs/                # 设计文档（如后续补充）
```

## 🚀 使用方式

1. 配置 OpenClaw Agent
2. 导入 Skills
3. 创建 Canon Bible
4. 按工作流执行：规划 → 约束 → 创作 → 结构校验 → 质量审核 → 归档

## 📝 工作流

```text
Architect 创建章纲
→ Scene Planner 拆解 Beats
→ Writer 按 chapter_id / scene_id 写正文
→ Critic 先做结构闸门，再做质量审核
→ Archivist 仅归档“已批准且元信息完整”的稿件
```

## 📄 许可证

MIT

---

**Author**: OceanEyeFF  
**系统**: OpenClaw Multi-Agent
