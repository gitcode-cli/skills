---
name: gitcode-issue-triage
description: |
  Triage GitCode issues: classify issue type, suggest labels, identify
  duplicate issues, assess priority, and apply triage results.

  TRIGGER when: user asks to triage GitCode issues, classify issues, suggest
  issue labels, find duplicate issues, assess issue priority, or phrases like
  "issue分类", "处理issue", "issue标签", "重复issue", "issue优先级", "triage issue".
---

# GitCode Issue Triage

对 GitCode Issue 进行分类、查重、标签建议、优先级判断，并直接应用分类结果。

## 与其他 skill 的分工

| Skill | 时机 | 做什么 |
|-------|------|--------|
| `gitcode-issue-create` | Issue **创建前** | 从零撰写 issue → 查重 → 提交 |
| **gitcode-issue-triage（本 skill）** | Issue **创建后** | 分类 → 标签 → 优先级 → 分流 |
| `gitcode-issue-review` | 开发**动手前** | 深度需求分析 → 任务分解 → 实现建议 |

## 核心工作流

### 第一步：收集信息

```bash
# Issue 详情
gc issue view <number> --json

# Issue 评论（理解讨论上下文）
gc issue comments <number> --json

# 关联的 PR 和 issue 关系
gc issue relations <number>

# 仓库现有标签体系
gc label list -R <owner/repo> --json
```

### 第二步：查重

```bash
# 搜索相似 issue（open + closed）
gc issue list -R <owner/repo> --search "<关键词1>" --state all --json
gc issue list -R <owner/repo> --search "<关键词2>" --state open --json

# 搜索相同标签的已有 issue
gc issue list -R <owner/repo> --label <label> --state open --json
```

### 第三步：分类

| 维度 | 选项 | 判断依据 |
|------|------|---------|
| **类型** | bug/feature/enhancement/docs/question | 标题关键词 + 描述内容 |
| **模块** | api/pr/issue/release/milestone/... | 涉及的命令或模块 |
| **来源** | user-report/internal/regression/security | 报告上下文 |

### 第四步：标签和元数据建议

基于仓库**实际标签**建议（不是凭空推荐）：

```bash
# 如果需要的标签不存在，可先创建
gc label create "<name>" --color "#hex" -R <owner/repo>
```

建议维度：
- 类型标签：`type/bug`, `type/feature`, `type/enhancement`
- 范围标签：`scope/api`, `scope/pr`, `scope/issue`, `scope/cli`
- 优先级标签：`priority-high`, `priority-medium`, `priority-low`
- 状态标签：`status/triage`, `status/confirmed`, `needs-info`

### 第五步：优先级判断

| 级别 | 适用场景 |
|------|---------|
| **P0/Critical** | 生产阻塞、安全漏洞、数据丢失 |
| **P1/High** | 主要功能阻断、影响大量用户 |
| **P2/Medium** | 重要但非紧急、有 workaround |
| **P3/Low** | 边缘场景、可延后、nice-to-have |

### 第六步：应用结果

```bash
# 添加标签
gc issue label <number> --add <labels> -R <owner/repo>

# 移除标签
gc issue label <number> --remove <labels> -R <owner/repo>

# 设置受理人
gc issue edit <number> --assignee <username> -R <owner/repo>

# 添加 triage 说明
gc issue comment <number> --body "Triage 结果：..."

# 如果是重复 issue，建议关闭并引用原 issue
gc issue close <number> -R <owner/repo>
```

## 输出格式

```markdown
## Issue #<number> Triage

### 基本信息
- 标题/作者/状态/创建时间

### 分类
- 类型/模块/来源

### 查重结果
- 重复: yes/no，原始 issue #（如有）

### 标签建议
- 添加/移除

### 优先级
- P0-P3 + 依据

### 下一步
- 如：需要补充信息、可直接开发、建议转 gitcode-issue-review
```

## 示例用法

```
对 Issue #123 做 triage，看看是什么类型、该打什么标签
```
```
批量处理 gitcode-cli/cli 最近 10 个新 issue
```
```
查一下 Issue #456 是不是已经有重复的了
```

## 注意事项

- 标签建议必须基于 `gc label list` 的实际结果，不凭空猜测
- 查重搜索已关闭 issue（问题可能已修复）
- 重复判定谨慎：相似标题不一定是重复，需对比描述内容
- 信息不足时建议添加 `needs-info` 标签
- 应用标签前与用户确认
