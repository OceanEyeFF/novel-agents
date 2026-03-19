---
name: novel-verify
description: 小说质量验证 - 检查 OOC、Canon 冲突、信息泄露、文风漂移。
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

验证小说内容的质量：角色一致性（OOC）、Canon 冲突、信息泄露、文风漂移。

## When to Use

✅ **USE this skill when:**

- 场景/章节写完后需要审核
- 怀疑有 OOC（Out Of Character）
- 需要检查是否泄露伏笔
- 检查文风是否漂移

## 验证类型

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

1. **读取相关状态文件**
   ```bash
   read: states/characters/{{角色名}}.md
   read: states/open_loops/index.md
   ```

2. **读取 Canon**
   ```bash
   read: canon/bible.md
   ```

3. **对比检查**
   - 对照角色状态，检查行为/语言是否一致
   - 对照 Canon，检查是否冲突
   - 对照伏笔表，检查是否泄露

4. **输出评审报告**
   ```bash
   write: reviews/review-{{场景ID}}.md
   ```

## 评审报告模板

```markdown
# 评审报告 - {{场景ID}}

## 评审结果
- [ ] 通过
- [ ] 需修改
- [ ] 需重写

## 问题列表

### Fatal（必须修复）

| 问题 | 位置 | 建议 |
|------|------|------|

### Major（建议修复）

| 问题 | 位置 | 建议 |
|------|------|------|

### Minor（可选优化）

| 问题 | 位置 | 建议 |
|------|------|------|

## 问题分类

- OOC:
- Canon 冲突:
- 信息泄露:
- 文风:

## 建议动作
- [ ] patch scene
- [ ] rewrite scene
- [ ] replan next scenes
- [ ] escalate to architect
```

## 注意事项

1. **独立审核**: Critic 应该独立于 Writer，确保客观
2. **结构化输出**: 问题必须有明确位置和建议
3. **分级处理**: Fatal 必须修复，Minor 可选
