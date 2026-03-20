---
name: novel-state
description: 小说状态管理 - 角色状态、时间线、伏笔追踪。
metadata:
  {
    "openclaw":
      {
        "emoji": "🔄",
        "description": "小说动态状态管理",
      },
  }
---

# novel-state

管理角色状态、时间线、伏笔/承诺追踪。

> 执行前提：必须先锁定 `novel_id`，所有状态文件都应写入 `projects/{{novel_id}}/workspace/states/`。

## When to Use

✅ **USE this skill when:**

- 查询角色当前状态
- 更新角色状态（章节/场景结束后）
- 查看伏笔投放/回收情况
- 查询时间线
- 登记新的情节点

## 目录结构

```
projects/{{novel_id}}/workspace/states/
├── characters/         # 角色状态
│   ├── index.md       # 角色索引+关系矩阵
│   └── {{角色名}}.md  # 角色动态状态
├── timeline/          # 时间线
│   └── index.md
└── open_loops/        # 伏笔/承诺
    └── index.md
```

## 常用操作

### 查询角色状态

```bash
# 读取角色状态索引
read: projects/{{novel_id}}/workspace/states/characters/index.md

# 读取特定角色状态
read: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 更新角色状态

每次场景/章节结束后，需要更新：

1. 当前目标
2. 当前情绪
3. 关系变化
4. 新增承诺
5. 新增秘密

```bash
# 编辑角色状态文件
edit: projects/{{novel_id}}/workspace/states/characters/{{角色名}}.md
```

### 登记新情节点

```bash
# 编辑伏笔表
edit: projects/{{novel_id}}/workspace/states/open_loops/index.md
```

### 查看时间线

```bash
read: projects/{{novel_id}}/workspace/states/timeline/index.md
```

## 数据结构

### 角色状态结构

```markdown
# {{角色名}} - 动态状态

## 当前状态（第X章）
| 字段 | 内容 |
|------|------|
| 当前章节 | |
| 当前目标 | |
| 当前情绪 | |
| 当前地点 | |

### 最近变化
- 章节N: 

### 未兑现承诺

### 开放冲突
```

### 伏笔表结构

```markdown
# 伏笔与开放循环

## 索引
| 伏笔ID | 类型 | 内容 | 投放章 | 状态 |
|--------|------|------|--------|------|

## 开放承诺
| 承诺ID | 内容 | 承诺人 | 接收人 | 投放章 | 状态 |

## 待回收伏笔（高优先级）
| 伏笔ID | 预期回收章 | 风险 |
|--------|------------|------|

## 久未提及（需检查）
| 伏笔ID | 最后出现 | 当前章节 | 建议 |
```

## 更新时机

### 每场景结束后
- 更新角色当前状态
- 检查是否产生新承诺/秘密

### 每章节结束后
- 更新时间线
- 检查伏笔投放/回收
- 更新章节摘要

## 注意事项

1. **及时更新**: 状态必须在场景/章节结束后立即更新
2. **可追溯**: 每次变更应有记录
3. **一致性**: 状态变化必须与正文一致
