# Writer Room - OpenClaw 小说生成系统设计

> 基于 OpenClaw 多 Agent 协作架构的叙事生产系统

---

## 一、核心定位

**Writer Room OS** — 一个常驻的小说创作房间，而不是单次 prompt 包装器。

---

## 二、6+1 Agent 结构

### 1. Architect（总规划师）
- 维护全书弧光、卷纲、章纲
- 管控主题、高潮、节奏
- **不可写正文**，只做规划
- 权限：可读 Canon Bible、全书大纲、伏笔表；可写章纲

### 2. Character Director（角色导演）
- 维护角色状态机（目标、恐惧、关系变化）
- 动态人设解释器
- 输出角色行动约束，不是 prose

### 3. Scene Planner（场景策划）
- 把章纲拆成 scene beats
- 固定输出结构：Scene ID / POV / Goal / Conflict / Reveal/Hide / Emotional Shift / Exit State / Token Budget

### 4. Writer（正文写手）
- 只根据批准的 scene beats 写 prose
- 保持局部文风、语言质感
- 不得擅自改 canon

### 5. Critic（批评审校）
- 检查逻辑、节奏、OOC、设定冲突
- 结构化输出：问题级别 + 类型 + 建议动作
- 与 Writer 隔离

### 6. Archivist（档案官）
- 场景结束后更新状态层
- 写摘要、更新时间线/伏笔表/角色状态表

### 7. Orchestrator（调度员，可选）
- 决定调用谁、监控质量、触发 cron

---

## 三、权限分层

| Agent | 可读 | 可写 | 不可写 |
|-------|------|------|--------|
| Architect | 全局规划、状态表 | 章纲、弧光 | 正文、基础 canon |
| Character Director | 人设、时间线 | 角色状态表 | 正文 |
| Scene Planner | 章纲、状态表 | scene beats | 全局 canon |
| Writer | 当前场景材料 | scene draft | 状态表、全局规划 |
| Critic | scene draft + 状态 | 评审报告 | 正文定稿 |
| Archivist | approved scene + 状态 | 摘要、索引 | 正文创作 |

---

## 四、专用工具集

### A. Canon / State 类
- `get_canon(section_ids)` — 取固定设定
- `get_character_state(character_ids, chapter_id)` — 角色动态状态
- `update_character_state(diff)` — 更新状态
- `get_timeline_window(chapter_range)` — 时间线局部
- `register_plot_point(plot_point)` — 登记情节点
- `get_open_loops()` — 查未回收伏笔

### B. Planning 类
- `create_chapter_plan(arc_id, constraints)`
- `create_scene_beats(chapter_id, target_length)`
- `replan_future_chapters(from_chapter_id, cause)`
- `score_chapter_function(chapter_id)`

### C. Verification 类
- `check_canon_conflict(scene_text)`
- `check_character_consistency(scene_text, character_ids)`
- `check_reveal_leak(scene_text)`
- `check_style_drift(scene_text, style_profile_id)`
- `evaluate_scene_goal(scene_text, scene_id)`

### D. Memory / Retrieval 类
- `retrieve_relevant_memories(query, top_k, memory_types)`
- `snapshot_scene(scene_id, summary, state_diff, hooks)`

---

## 五、四层上下文设计

### 1. 固定层
- Canon Bible
- 当前风格模板
- 当前章批准版章纲
- 当前 scene beats

### 2. 动态状态层
- POV 角色状态
- 相关配角状态
- 开放悬念
- 本章状态变化

### 3. 近期语言层
- 上一场景正文
- 本章前几场景摘要

### 4. 长程召回层
- 相关旧伏笔
- 相关历史事件
- 关键旧场景摘要

---

## 六、双循环系统

### 内循环：写一场景
1. Scene Planner 产出 beat
2. Character Director 产出角色约束
3. Writer 写场景
4. Critic 审核
5. 通过 → Archivist 归档
6. 不通过 → 局部重写

### 外循环：写一章/一卷
1. Architect 检查章节弧光目标
2. 检查角色关系推进
3. 检查伏笔投放/回收
4. 必要时重规划未来 1-3 章

---

## 七、双模式

### 模式 1：岗位型多 Agent
- Architect / Writer / Critic 等
- 工程稳定、权限清晰

### 模式 2：角色型多 Agent
- 主角/反派/配角各一个轻量 agent
- 只参与：对局势的私有判断、行动倾向
- 限定轮数的结构化博弈

---

## 八、后台创作维护任务（Cron）

### 每章完成后
- 自动更新角色状态
- 生成本章摘要
- 更新伏笔索引
- 检查未闭合信息

### 每天夜间
- 风格漂移分析
- 节奏分析
- 角色出场权重统计
- 找出久未回收的伏笔

### 每卷完成后
- 卷总结
- 主线推进率
- 角色弧线偏差报告
- 下卷风险预警

---

## 九、审批闸门

### Gate 1：章纲审批
Architect 章纲 → 人工批准 → 进入 scene planning

### Gate 2：重大设定变更
影响 canon 的变更 → 人工批准

### Gate 3：发布审批
Critic + 人工终审

---

## 十、深水区设计（进阶）

### 1. Narrative State Graph
- 节点：人物/线索/组织/地点/承诺/谣言/秘密
- 边：知晓/误解/怀疑/敌对/依赖/欠债/隐瞒

### 2. Reader Model Agent
- 维护读者已知/猜测/悬念/误导点
- 辅助 Scene Planner 控悬念

### 3. Style Profile Agent
- 统计句长/描写对话比/比喻密度/视角/词性/情绪词
-Critic 据此给出量化修订建议

### 4. Failure Recovery
- 场景草稿版本化
- 审批结果落盘
- 状态表只 append diff
- 支持回滚到上一批准场景

---

## 十一、目录结构

```
novel-room/<project-id>/
├── canon/
├── plans/
│   ├── arc/
│   ├── chapter/
│   └── scene/
├── states/
│   ├── characters/
│   ├── timeline/
│   └── open_loops/
├── drafts/scenes/
├── reviews/
├── snapshots/
└── metrics/
    ├── style/
    └── pacing/
```

---

## 十二、下一步建议

1. **定死 Agent 边界** — 谁能写什么
2. **定义状态结构** — 角色状态 / 时间线 / 伏笔表
3. **定审批流** — 什么情况通过/重写/升级/人工介入
