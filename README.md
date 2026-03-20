# Writer Room - 多 Agent 协作小说生产系统

> 基于 OpenClaw 架构的多 Agent 协作叙事生产系统

## 🎭 Agent 阵容（v0.2）

| Agent | 名称 | 职责 |
|-------|------|------|
| **orchestrator** | 调度员 | 常驻中心调度、状态机推进、返工路由 |
| **architect** | 架构师 | 维护全书弧光、卷纲、章纲 |
| **character-director** | 角色导演 | 维护角色状态机、追踪角色动态 |
| **scene-planner** | 场景策划 | 把章纲拆成 Scene Beats |
| **prose-director** | 小说导演 | 把 Scene Beats 翻译成小说表达方案 |
| **writer** | 写手 | 按 prose brief 完成 skeleton + expansion |
| **critic** | 审稿 | 检查结构错误、OOC、Canon 冲突、文本密度 |
| **archivist** | 档案官 | 更新状态、写摘要、管理快照 |

## 🧰 Skills 组成

当前仓库包含 9 个专用 Skills：

- `novel-canon`：核心设定查询
- `novel-plan`：卷纲 / 章纲 / Scene Beats 规划
- `novel-state`：角色状态、时间线、伏笔状态
- `novel-verify`：质量、结构与文本密度验证
- `novel-guard`：发布前边界校验与审批闸门
- `novel-draft`：正文创作辅助（skeleton / expansion / bridge）
- `novel-novelize`：把 scene beats 转成 prose brief，并做小说化扩写
- `novel-retrieve`：长短期记忆检索
- `novel-snapshot`：版本快照与回滚

## ⚠️ v0.2 当前重点改进

本轮文档强化主要解决以下高风险问题：

1. **剧本化风险**：Scene Planner 不再只输出功能型 beats，而是通过 Prose Director 生成小说化表达方案。
2. **人工推进过多**：Orchestrator 从“可选路由器”升级为主控状态机，负责自动推进内循环与返工分流。
3. **片段短、空、流水账**：Writer 改为 skeleton → expansion 双阶段写法，并使用段落级预算代替单一 token budget。
4. **章节拼接感**：新增 chapter assembly / chapter-level critic，补 scene 间过桥段、余波段、呼吸段。
5. **错误通过审核**：Critic 除结构闸门外，必须输出 Scene Density / Novelness / Chapter Cohesion 评分。

## 📖 作品展示

本系统已完成的作品：**《暗恋我的女生过于熟练却没有恋爱经验》**

- 第1卷：5章，约 19,000字
- 状态：完结 ✓

## 🛠️ 系统架构

```text
novel-agents/
├── agents/              # 8个岗位型 Agent 说明
├── skills/              # 9个专用 Skills
│   ├── novel-canon/
│   ├── novel-plan/
│   ├── novel-state/
│   ├── novel-verify/
│   ├── novel-guard/
│   ├── novel-draft/
│   ├── novel-novelize/
│   ├── novel-retrieve/
│   └── novel-snapshot/
├── docs/                # v0.2 架构与状态机设计
└── workspace/           # 项目工作区
```

## 🚀 使用方式

1. 配置 OpenClaw Agent
2. 导入 Skills
3. 创建 Canon Bible
4. 以章节为生产单元执行：规划 → beats → prose brief → skeleton → expansion → 审核 → 组章 → 归档

## 📝 工作流（v0.2）

```text
Orchestrator 读取 chapter 状态
→ Architect 创建/修订章纲
→ Scene Planner 生成 scene beats
→ Prose Director 生成 prose brief
→ Writer 先写 skeleton，再做 prose expansion
→ Critic 先做结构闸门，再做 scene-level critic
→ Orchestrator 执行 chapter assembly / chapter-level critic
→ Archivist 仅归档“已批准且元信息完整”的稿件
```

## 📚 关键文档

- `novel-system-design.md`：v0.2 系统设计
- `novel-agents-config.md`：岗位权限与工具映射
- `docs/v0.2-architecture.md`：状态机、返工路由、技能接口草案

## 📄 许可证

MIT

---

**Author**: OceanEyeFF  
**系统**: OpenClaw Multi-Agent
