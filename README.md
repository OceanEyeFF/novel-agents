# Writer Room - 多 Agent 协作小说生产系统

> 基于 OpenClaw 架构的多 Agent 协作叙事生产系统

## 🎭 执行层 Agent 阵容（v0.2）

| Agent | 名称 | 职责 |
|-------|------|------|
| **orchestrator** | 调度员 | 常驻中心调度、状态机推进、返工路由 |
| **architect** | 架构师 | 维护全书弧光、卷纲、章纲 |
| **character-director** | 角色导演 | 维护角色状态机、追踪角色动态 |
| **scene-planner** | 场景策划 | 把章纲拆成 Scene Beats |
| **walk-on-director** | 路人导演 | 根据剧情缺口补充路人/一次性配角人设 |
| **prose-director** | 小说导演 | 把 Scene Beats 翻译成小说表达方案 |
| **writer** | 写手 | 按 prose brief 完成 skeleton + expansion |
| **critic** | 审稿 | 检查结构错误、OOC、Canon 冲突、文本密度 |
| **archivist** | 档案官 | 更新状态、写摘要、管理快照 |

## 🧭 前台/入口能力（实验性，不计入核心 Room Agents）

> 这两项更适合先作为 **preflight capabilities / orchestrator 子流程** 验证，而不是立刻提升为常驻 Room Agents。

| Capability | 名称 | 当前定位 |
|-----------|------|----------|
| **briefing** | 立项澄清 | 对话式归纳已确认/未确认信息，判断是否具备准入条件 |
| **bootstrap** | 启动包转换 | 把 intake summary 转成 `project.yaml` 与 `runtime.yaml`，形成状态契约 |

在它们通过真实工作负载测试前，默认仍以现有 9 个执行层 Agent 作为核心房间编制。

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
6. **多书串稿风险**：所有技能与工作流都必须绑定显式 `novel_id`，每本书拥有独立工作区。
7. **同步不稳定**：GitHub 继续作为源码仓库，调试/安装改为优先消费 clawskills.sh 上的已发布版本。

## 📖 作品展示

本系统已完成的作品：**《暗恋我的女生过于熟练却没有恋爱经验》**

- 第1卷：5章，约 19,000字
- 状态：完结 ✓

## 🛠️ 系统架构

```text
novel-agents/
├── agents/                         # 8个执行岗位型 Agent 说明
├── skills/                         # 9个专用 Skills
│   ├── novel-canon/
│   ├── novel-plan/
│   ├── novel-state/
│   ├── novel-verify/
│   ├── novel-guard/
│   ├── novel-draft/
│   ├── novel-novelize/
│   ├── novel-retrieve/
│   └── novel-snapshot/
├── docs/                           # v0.2 架构与发布设计
├── projects/                       # 多小说独立工作区（推荐）
│   └── {{novel_id}}/
│       ├── manifest.yaml
│       ├── project.yaml            # Project Layer：长期约束
│       ├── intake/
│       │   └── latest-summary.yaml # Intake Layer：对话归纳
│       └── workspace/
│           └── runtime/
│               └── runtime.yaml    # Runtime Layer：当前叙事/任务/调度状态
└── workspace/                      # 工作区规范与模板说明
```

## 🧱 多小说隔离约定

从 v0.2 起，**不再推荐把所有作品都写进单一 `novel-workspace/`**。

推荐使用以下目录约定：

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml               # 小说元信息、slug、发布配置
    └── workspace/
        ├── canon/
        ├── plans/
        ├── states/
        ├── drafts/
        ├── reviews/
        ├── summaries/
        ├── snapshots/
        └── metrics/
```

核心要求：

1. **所有 Agent / Skill 调用前必须先确定 `novel_id`。**
2. **所有读写路径必须落在 `projects/{{novel_id}}/workspace/` 下。**
3. **任何缺少 `novel_id` 或无法唯一定位工作区的任务一律中止。**
4. **跨小说检索必须显式声明允许，否则默认禁止。**

## 🚀 使用方式

1. 通过 intake/briefing 子流程做对话式立项 / 任务澄清
2. 生成 `projects/{{novel_id}}/intake/latest-summary.yaml`
3. 由 bootstrap 子流程产出 `manifest.yaml`、`project.yaml`、`workspace/runtime/runtime.yaml`
4. 配置 OpenClaw Agent 并导入 Skills
5. 初始化该小说的独立工作区
6. 以章节为生产单元执行：规划 → beats → prose brief → skeleton → expansion → 审核 → 组章 → 归档
7. 将稳定版本发布到 clawskills.sh，供 OpenClaw 环境按版本安装和调试

## 🔄 推荐分发方式

**推荐链路：GitHub → CI 校验 → clawskills.sh 发布 → OpenClaw 安装指定版本。**

这样做的原因：

- GitHub 适合作为**源码真相源**与评审入口。
- clawskills.sh 更适合作为**可安装、可回滚、可对照版本**的分发层。
- OpenClaw 不再直接依赖“仓库最新状态”，调试时更容易复现环境。
- 当同步偶发失败时，可以先对比“Git commit / 发布版本 / 实际安装版本”三者是否一致。

详细设计见：`docs/distribution-and-publishing.md`。

## 📝 工作流（v0.2）

```text
作者自然语言输入
→ briefing capability 归纳 confirmed / unresolved / decision rights
→ bootstrap capability 生成 project.yaml + runtime.yaml
→ Orchestrator 确认 novel_id、workspace root、runtime.stage
→ Architect 创建/修订章纲
→ Scene Planner 生成 scene beats
→ Prose Director 生成 prose brief
→ Writer 先写 skeleton，再做 prose expansion
→ Critic 先做结构闸门，再做 scene-level critic
→ Orchestrator 执行 chapter assembly / chapter-level critic
→ Archivist 仅归档“已批准且元信息完整”的稿件
→ Release Manager/CI 发布已验证技能版本到 clawskills.sh
```

## 📚 关键文档

- `novel-system-design.md`：v0.2 系统设计
- `novel-agents-config.md`：岗位权限与工具映射
- `docs/v0.2-architecture.md`：状态机、返工路由、技能接口草案
- `docs/intake-and-state-contract.md`：对话式立项、准入流程、`project.yaml` / `runtime.yaml` 契约
- `docs/idea-narrative-runtime-and-chapter-quality.md`：Narrative Runtime 与章节质量控制的双层设计提炼
- `docs/distribution-and-publishing.md`：多小说隔离与发布策略
- `workspace/README.md`：独立工作区结构规范
- `workspace/tools-guide.md`：带 `novel_id` 的工具读写约定

## 📄 许可证

MIT

---

**Author**: OceanEyeFF  
**系统**: OpenClaw Multi-Agent


## 🧪 新能力落地原则

- `briefing` / `bootstrap` 很重要，但**是否要升级成新的 Room Agents，需要先做讨论和真实 workload tests**。
- 在验证前，建议把它们视为 **pre-orchestrator capabilities**，由 Orchestrator 或单独入口服务承载。
- 只有当它们能稳定降低人工澄清次数、减少误启动、且不显著拖慢吞吐时，再考虑把它们升格为常驻 Agent。
