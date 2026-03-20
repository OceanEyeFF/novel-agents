# Writer Room - OpenClaw 小说生成系统设计（v0.2）

> 基于 OpenClaw 多 Agent 协作架构的章节导演制叙事生产系统

---

## 一、核心定位

**Writer Room OS** — 一个常驻的小说创作房间，而不是单次 prompt 包装器。

v0.2 的目标不是继续细化单个 Scene 的功能流水线，而是把系统从“场景功能完成”升级为“章节导演制小说工坊”：

- 以 **chapter** 作为核心生产单元，而不是孤立 scene。
- 通过 **Prose Director** 把场景功能翻译成小说表达方案。
- 通过 **Orchestrator** 自动推进内循环、状态机与返工决策。
- 通过 **chapter assembly** 消除多 scene 并列导致的拼接感。

---

## 二、8 Agent 结构

### 1. Orchestrator（调度员，主控）
- 常驻中心管理 Agent，负责状态机推进
- 根据产物状态自动调用下游 Agent
- 负责返工分流：局部修文 / 场景重写 / 重规划 / 人工升级
- 只写日志、任务状态、执行记录，不直接改正文或状态表

### 2. Architect（总规划师）
- 维护全书弧光、卷纲、章纲
- 管控主题、高潮、节奏
- **不可写正文**，只做规划
- 权限：可读 Canon Bible、全书大纲、伏笔表；可写章纲

### 3. Character Director（角色导演）
- 维护角色状态机（目标、恐惧、关系变化）
- 输出角色行动约束、情绪边界、信息认知差
- 不负责 prose，只负责角色约束

### 4. Scene Planner（场景策划）
- 把章纲拆成 scene beats
- 负责 scene sequencing、entry/exit state、隐藏信息边界
- 输出不再只强调事件功能，还要为 prose 层提供可转译骨架

### 5. Prose Director（小说导演）
- 把 scene beat 翻译成 **小说表达方案（prose brief）**
- 指定叙述重心、慢写点、感官目标、内心活动点、对话功能、段落节奏
- 不改剧情，不直接写最终正文

### 6. Writer（正文写手）
- 按已批准的 prose brief 生成正文
- 采用两阶段：`skeleton draft` → `prose expansion`
- 不得擅自改 canon、章纲或状态层

### 7. Critic（批评审校）
- 先检查结构闸门，再检查逻辑、节奏、OOC、设定冲突
- 新增文本质量维度：**Scene Density / Novelness / Chapter Cohesion**
- 输出结构化返工建议，供 Orchestrator 自动路由

### 8. Archivist（档案官）
- 场景结束后更新状态层
- 写摘要、更新时间线/伏笔表/角色状态表
- 保存草稿版本、审批链与快照索引

---

## 三、生产状态机

### Chapter 状态

- `planned`：章纲已建立，但未拆 scene
- `beat_ready`：scene beats 已生成
- `prose_ready`：prose brief 已生成
- `drafting`：scene skeleton / expansion 进行中
- `review_failed_minor`：局部修文即可
- `review_failed_major`：需要重写 scene 或补桥段
- `replan_required`：需回退到 Scene Planner / Architect
- `scene_approved`：scene 审核通过
- `chapter_assembling`：进行组章、过桥段、余波段补写
- `chapter_approved`：章级审查通过
- `archived`：完成归档与快照

### Critic 返工动作枚举

- `fix_local_language`
- `expand_scene`
- `rewrite_scene`
- `bridge_scenes`
- `replan_scene`
- `replan_chapter`
- `escalate_human`

Orchestrator 必须根据 `required_action` 把任务回退到正确层级，而不是把所有问题都退回 Writer。

---

## 四、权限分层

| Agent | 可读 | 可写 | 不可写 |
|-------|------|------|--------|
| Orchestrator | 全部元信息、日志、审批链 | logs、jobs、状态机记录 | 正文、评审、核心状态 |
| Architect | 全局规划、状态表 | 章纲、弧光 | 正文、基础 canon |
| Character Director | 人设、时间线、章纲 | 角色状态表、角色约束 | 正文 |
| Scene Planner | 章纲、状态表、开放循环 | scene beats | 全局 canon、正文 |
| Prose Director | scene beats、角色状态、近邻场景摘要、文风模板 | prose briefs | 章纲、正文定稿、状态表 |
| Writer | prose brief、角色状态摘要、近邻场景、风格模板 | scene skeleton、scene draft、bridge draft | 状态表、全局规划 |
| Critic | draft、状态、canon、prose brief | 评审报告 | 正文定稿 |
| Archivist | approved draft、评审、状态 | 摘要、索引、快照、状态 diff | 正文创作 |

---

## 五、中间表示升级：从 Scene Beat 到 Prose Brief

### Scene Beat（规划层）

每个 scene beat 至少包含：

- `scene_id`
- `pov`
- `narrative_function`
- `goal`
- `conflict`
- `reveal_or_hide`
- `entry_state`
- `exit_state`
- `must_include`
- `must_not_include`

### Prose Brief（小说化层）

每个 prose brief 至少包含：

- `scene_id`
- `emotional_core`
- `narrative_focus`
- `slow_beats`（必须放慢描写的 2–3 个点）
- `sensory_targets`
- `inner_monologue_targets`
- `dialogue_purpose`
- `hidden_information`
- `paragraph_rhythm`
- `prose_length_plan`

### 段落级预算替代总 Token Budget

不再只给单一 `token_budget`，改成按段落职能分配长度，例如：

- 开场锚定段：150–250 字
- 主互动段：250–400 字
- 情绪深化段：150–300 字
- 收束/钩子段：100–200 字

这样可以防止 Writer 为了赶预算而退化成事件摘要。

---

## 六、四层上下文设计（v0.2）

### 1. 固定层
- Canon Bible
- 当前风格模板
- 当前章批准版章纲
- 当前 scene beats
- 当前 prose brief

### 2. 动态状态层
- POV 角色状态
- 相关配角状态
- 开放悬念
- 本章状态变化
- 当前 scene 的 hidden information 列表

### 3. 近期语言层
- 上一场景正文
- 本章前几场景摘要
- 已批准过桥段/余波段

### 4. 长程召回层
- 相关旧伏笔
- 相关历史事件
- 关键旧场景摘要
- 与当前情绪核心相关的历史回声

---

## 七、双循环系统

### 内循环：写一场 Scene
1. Scene Planner 产出 beat
2. Character Director 产出角色约束
3. Prose Director 产出 prose brief
4. Writer 写 `skeleton draft`
5. Writer 执行 `prose expansion`
6. Critic 审核
7. Guard 检查章节边界与审批链
8. 通过 → Archivist 归档 scene 级状态 diff
9. 不通过 → Orchestrator 根据 `required_action` 自动回退

### 外循环：写一章/一卷
1. Architect 检查章节弧光目标
2. 检查角色关系推进
3. Scene Planner 拆 3–5 个 scenes
4. Writer 完成各 scene 后，由 Orchestrator 进行 chapter assembly
5. Critic 跑 chapter-level critic
6. 必要时补写 bridge scene / emotional aftershock / breathing paragraph
7. Archivist 写章节摘要并生成快照

---

## 八、Chapter Assembly

Chapter Assembly 不是单独的创作岗位，而是由 Orchestrator 管理的章级组装过程，目标是把“多个完成任务的 Scene”变成“连续可阅读的一章”。

必须检查：

- scene 与 scene 之间是否跳切过硬
- 是否缺少过桥段、余波段、呼吸段
- 章节前半与后半的段落密度是否失衡
- 情绪曲线是否递进，而非平铺并列
- 本章结尾是否有明确的收束感或钩子

---

## 九、Critic 扩展指标

### Gate A：结构完整性
任何正文在进入内容审核前，必须先验证：
- 文件名 / 标题 / `chapter_id` / `scene_id` 一致
- 一个文档只归属于一个章节或一个 Scene
- 正文、摘要、评审报告是分离产物
- 引用的 scene beat 与 prose brief 已批准

### Gate B：内容质量
- OOC
- Canon 冲突
- 信息泄露
- 文风漂移
- 场景目标是否达成

### Gate C：文本密度与小说性
- **Scene Density**：感官描写密度、动作细节、情绪承载密度、非摘要句比例
- **Novelness**：是否像提纲扩写、对白脚本、事件总结；是否具备 POV 粘性
- **Chapter Cohesion**：scene 是否只是并列片段；是否形成章节级起伏和收束

### 剧本化风险判定
当一个 scene 出现以下任意 3 条以上时，必须标记 `script_like_risk = high`：
- 以对话推进为主，几乎没有感官描写
- 每段只交代动作和信息，没有停顿和反应
- 人物情绪只被命名，没有被体验化
- 场景环境只被点名，没有参与叙事
- 对白过于功能化，潜台词太少
- 段落长度非常均匀，像提纲展开
- 转场过快，没有余波句
- POV 虽存在，但缺少视角粘性

若风险过高，Critic 至少要求补齐：
- 1 个环境锚点
- 1 个身体感受
- 1 个内心误判或联想
- 1 个节奏放慢点
- 1 个非功能性但增强真实感的细节

---

## 十、审批闸门

### Gate 1：章纲审批
Architect 章纲 → 人工批准 → 进入 scene planning

### Gate 2：重大设定变更
影响 canon 的变更 → 人工批准

### Gate 3：发布审批
Critic + Guard + 人工终审

### 人工介入原则（v0.2）
人类默认只介入以下节点：
- 首次章纲批准
- 重大设定变更
- 最终发布前抽检

其余常规返工应由 Orchestrator 自动推进。

---

## 十一、下一步建议

1. **把 Orchestrator 从可选改成主控**
2. **在 Scene Planner 与 Writer 之间引入 Prose Director**
3. **将 Writer 改为 skeleton → expansion 双阶段**
4. **增加 chapter assembly 与 chapter-level critic**
5. **为技能层补充 novel-novelize 能力**
