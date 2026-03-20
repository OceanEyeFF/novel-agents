# Writer Room - 多小说隔离与发布策略

## 背景

当前系统如果默认把所有作品内容写入单一 `novel-workspace/`，会产生两个直接问题：

1. **多书混淆**：两本小说同时创作时，`canon`、`state`、`summary`、`snapshot` 很容易串写。
2. **同步不稳定**：如果 OpenClaw 直接依赖 GitHub 仓库最新状态，调试时很难确认“本地代码 / 已安装版本 / 线上运行版本”是否一致。

因此，建议将“工作区隔离”和“发布分发”视为同一个治理问题来解决。

---

## 一、工作区隔离：每本小说一个根目录

### 推荐结构

```text
projects/
└── {{novel_id}}/
    ├── manifest.yaml
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

### 为什么不能继续用单一 `novel-workspace/`

- `title` 不稳定，也可能重复，不能作为主键。
- 默认当前目录推断目标小说，会导致误写。
- 检索层如果不带 `novel_id`，会在摘要、伏笔、状态中混入别的作品。
- 快照、回滚、发布一旦不带项目边界，就无法准确恢复。

### 最低执行要求

1. 每次任务开始先锁定 `novel_id`。
2. 从 `manifest.yaml` 解析 `workspace_root`。
3. 所有 Skill 只允许访问该根目录内文件。
4. 如需跨书引用，必须显式声明 `allow_cross_novel: true`。

---

## 二、分发策略：GitHub 是源码源，clawskills.sh 是安装源

### 推荐角色分工

| 系统 | 角色 |
|------|------|
| GitHub | 源码真相源、代码评审、版本记录 |
| clawskills.sh | 可安装发布源、版本化分发、回滚锚点 |
| OpenClaw Runtime | 安装并运行指定版本 |

### 为什么推荐发布到 clawskills.sh

1. **版本可见**：能明确知道当前运行的是哪个发布版本。
2. **调试可复现**：问题出现时可以回到具体 package version，而不是模糊的“某次同步后的状态”。
3. **回滚更直接**：从 `0.2.3` 回到 `0.2.2` 比回滚一整套仓库同步状态更简单。
4. **环境更稳定**：OpenClaw 不需要直接追 GitHub HEAD，减少半同步、缓存、权限问题。
5. **多项目共存更自然**：可以按 `package` 或 channel 对不同小说/不同环境分别发版。

### 结论

**是的，优先发布到 `https://clawskills.sh/` 通常会比直接依赖 GitHub ↔ OpenClaw 同步更靠谱，也更利于调试。**

但这里的最佳实践不是“替代 GitHub”，而是：

> **GitHub 管源码，clawskills.sh 管分发，OpenClaw 只消费已发布版本。**

---

## 三、推荐发布流

```text
开发修改
→ 提交到 GitHub
→ CI 运行校验
→ 生成 Skill 发布包
→ 发布到 clawskills.sh
→ OpenClaw 安装指定版本
→ 用 manifest 记录当前小说绑定的技能版本
```

### 每次发布至少记录以下元信息

```yaml
release:
  git_commit: abcdef1234
  package: writer-room/core
  version: 0.2.0
  published_at: 2026-03-20T00:00:00Z
  target_runtime: openclaw
```

建议把这段信息写入：

- `projects/{{novel_id}}/manifest.yaml`
- 或独立 `projects/{{novel_id}}/release.lock`

这样出问题时，可以同时比对：

- GitHub commit
- clawskills 发布版本
- OpenClaw 实际安装版本

---

## 四、调试建议

### 本地开发阶段

- 允许使用 GitHub 工作树直接测试。
- 但仍然要绑定 `novel_id` 和独立工作区。

### 集成调试阶段

- 不要使用“仓库最新分支自动同步到 OpenClaw”作为唯一链路。
- 改为从 clawskills.sh 安装一个明确版本。
- 如果需要验证未发布变更，使用临时 prerelease tag，例如 `0.2.0-rc.1`。

### 生产运行阶段

- 只允许使用已发布版本。
- 所有运行实例都要有版本记录。
- 回滚只通过版本切换，不通过手工拷贝技能目录。

---

## 五、落地清单

### 必做

- [ ] 所有 Skill 文档显式要求输入 `novel_id`
- [ ] 所有工作区路径改为 `projects/{{novel_id}}/workspace/...`
- [ ] 增加 `manifest.yaml` 作为小说级元信息入口
- [ ] 建立 GitHub → clawskills.sh → OpenClaw 的版本流

### 强烈建议

- [ ] 引入发布锁文件记录 package version
- [ ] 在运行日志中打印 `novel_id`、`git_commit`、`package_version`
- [ ] 给每本小说分配独立 package 名或独立 channel
- [ ] 把“缺少 `novel_id`”视为阻断错误而非警告

---

## 六、什么时候不需要上 clawskills.sh

以下情况可以先不发布：

- 你只在本地单机试验技能草稿。
- 你还处于快速迭代 prompt 的阶段。
- 你不需要别人复现当前环境。

但一旦进入以下场景，就建议发版：

- 多人协作
- 多环境调试
- 线上运行
- 需要稳定回滚
- 需要区分不同小说或不同实验分支
