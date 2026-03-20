---
name: novel-verify
description: 小说质量验证 - 检查结构完整性、OOC、Canon 冲突、信息泄露、文风漂移。
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
        "description": "小说质量验证工具",
      },
  }
---

# novel-verify

验证小说内容的质量：先检查结构完整性，再检查角色一致性（OOC）、Canon 冲突、信息泄露、文风漂移。

> 执行前提：必须先锁定 `novel_id`，审核输入与评审输出都必须绑定当前小说工作区。

## When to Use

✅ **USE this skill when:**

- 场景/章节写完后需要审核
- 怀疑有 OOC（Out Of Character）
- 需要检查是否泄露伏笔
- 检查文风是否漂移
- 需要防止“串章、混稿、标题错配却仍被通过”

## 验证类型

### 0. 结构完整性检查（新增，必须优先执行）

检查文档是否具备最基本的发布资格。

**检查点：**
- 文件名、标题、`chapter_id`、`scene_id` 是否一致
- 文档是否只服务一个章节
- 是否混入多个 Scene / 多个章节的内容
- 是否存在正文、摘要、下章预告混写
- 所引用的 Scene Beat 是否存在且状态为 `approved`

### 1. OOC 检查 (Out Of Character)

检查角色行为/语言是否与其性格设定不符。

**检查点：**
- 语言风格是否一致
- 行为动机是否合理
- 性格特质是否保持

### 2. Canon 冲突检查

检查是否违背核心设定。

**检查点：**
- 世界观规则是否一致
- 时间线是否冲突
- 已确定的事实是否被改写

### 3. 信息泄露检查

检查是否过早揭示应隐藏的信息。

**检查点：**
- 是否在预期章节前揭示了伏笔
- 秘密是否被不应该知道的人知晓
- 悬念是否被意外破坏

### 4. 文风漂移检查

检查是否偏离既定文风。

**检查点：**
- 句式结构是否一致
- 描写密度是否保持
- 叙事视角是否统一

## 验证流程

### 手动验证步骤

1. **先执行结构检查**
   ```text
   检查: 文件名 / 标题 / chapter_id / scene_id / source_beat
   ```
2. **读取相关状态文件**
   ```text
   read: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
   read: projects/{{novel_id}}/workspace/states/open_loops/index.md
   ```
3. **读取 Canon**
   ```text
   read: projects/{{novel_id}}/workspace/canon/bible.md
   ```
4. **对比检查**
   - 对照角色状态，检查行为/语言是否一致
   - 对照 Canon，检查是否冲突
   - 对照伏笔表，检查是否泄露
5. **输出评审报告**
   ```text
   write: projects/{{novel_id}}/workspace/reviews/review-{{场景ID}}.md
   ```

## 一票否决条件

若出现以下任一情况，直接判 `需重写` 或 `需拆分`：

- 一个文档出现多个不同 `chapter_id`
- 一个文档出现多个不连续或不同归属的 `scene_id`
- 章节标题与元信息不一致
- 场景稿中混入章节摘要、下章预告或其他审批产物
- 结构完整性检查未通过

## 评审报告模板

```markdown
# 评审报告 - {{场景ID}}

## 审核对象
- review_target: {{scene_id}}
- chapter_id: {{chapter_id}}
- source_beat: {{source_beat}}

## Gate A - 结构完整性
- [ ] pass
- [ ] fail

### Gate A 问题
| 问题 | 位置 | 建议 |
|------|------|------|

## Gate B - 内容质量
- [ ] pass
- [ ] fail

### Fatal（必须修复）
| 问题 | 位置 | 建议 |
|------|------|------|

### Major（建议修复）
| 问题 | 位置 | 建议 |
|------|------|------|

### Minor（可选优化）
| 问题 | 位置 | 建议 |
|------|------|------|

## 决策
- decision: approved / revise / rewrite / split
- required_action: {{下一步动作}}
```

## 注意事项

1. **独立审核**: Critic 应该独立于 Writer，确保客观
2. **结构先行**: 未过结构闸门不得写“通过”
3. **分级处理**: Fatal 必须修复，Minor 可选
4. **禁止宽松放行**: 只要存在章节身份不清或混稿，就不能进入归档
