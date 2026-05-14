---
name: gitcode-pr-create
description: |
  检查当前分支变更，生成规范 PR 标题（conventional commit）和描述（变更摘要、
  测试计划、关联 issue），建议标签和评审人，确认后通过 gc CLI 提交 PR。

  TRIGGER when: 用户要创建/提交 PR、提 pull request、开 PR、create PR、
  submit PR、想合并当前分支、或者代码改完了想提交到远端。只要涉及"把当前分支
  变更做成 PR 提交"，即应触发。
---

# GitCode PR 创建助手

检查当前分支变更，自动生成规范的 PR 标题和描述，确认后提交。

## 与现有 skill 的分工

| Skill | 时机 | 做什么 |
|-------|------|--------|
| **gitcode-pr-create（本 skill）** | 开发**完成后** | 检查变更 → 生成标题描述 → 提交 PR |
| `gitcode-pr-review` | PR **提交后** | 多维度代码审查 → 行内评论 |

## 核心工作流

### 第一步：检查当前状态

确认分支和变更情况：

```bash
# 当前分支和状态
git status

# 与 base 分支的差异统计
git diff --stat origin/main...HEAD

# 最近的提交（用于生成标题和描述）
git log --oneline origin/main..HEAD

# 远程分支是否已推送
git remote -v
```

从输出中提取：
- 当前分支名和 base 分支
- 变更文件数、新增/删除行数
- 提交数量和历史
- 是否已推送到远端
- 是否有未提交的变更

如果存在未提交的变更，先提醒用户提交或暂存。

### 第二步：确定 PR 类型和范围

根据 commit 历史和代码变更判断：

| 类型 | 适用场景 | 标题格式 |
|------|---------|---------|
| `feat` | 新功能 | `feat(scope): 描述` |
| `fix` | Bug 修复 | `fix(scope): 描述` |
| `refactor` | 重构（无功能变更） | `refactor(scope): 描述` |
| `docs` | 文档更新 | `docs: 描述` |
| `test` | 测试补充 | `test(scope): 描述` |
| `chore` | 构建/工具/依赖 | `chore: 描述` |

`scope` 根据变更文件路径推断（如 `api`、`cmd/issue`、`pkg/pr`）。

### 第三步：生成 PR 标题

格式：`<type>(<scope>): <简短描述>`

原则：
- 描述用中文或英文，与项目现有风格一致
- 控制在 50 字符以内
- 如有对应 issue，尾部加 `(#issue号)`
- 如多个 commit 涉及不同类型，取最主要变更的类型

使用 `--fill` 可以直接从最后一个 commit 填充标题作为起点：
```bash
gc pr create -R <repo> --fill
```

### 第四步：生成 PR 描述

根据变更内容生成规范描述。如果项目有 PR 模板（如 `.gitcode/PULL_REQUEST_TEMPLATE.md`），优先使用模板。

```markdown
## 变更说明
[简要说明本次 PR 做了什么、为什么这样做]

## 变更内容
- [主要变更点 1]
- [主要变更点 2]
- [主要变更点 3]

## 关联 Issue
- Fixes #<issue_number>
- Related #<issue_number>

## 测试计划
- [ ] 单元测试通过
- [ ] [手动验证点 1]
- [ ] [手动验证点 2]

## 截图/日志
[如有 UI 变更或关键输出，附上截图]

## 风险
[如有破坏性变更、性能影响或需要特别注意的地方]
```

**生成要点**：
- 从 commit 历史中提取变更摘要（不要简单复制 commit message）
- "为什么" 比 "改了什么" 更重要——解释动机和上下文
- 列出关联 issue（搜索分支名或 commit message 中的 issue 编号）
- 测试计划要具体，不要写 "已测" 就结束

### 第五步：建议元数据

- **Draft/WIP 判断**：如果用户说 "还在开发中" 或有未完成的标记，建议 `--draft`
- **标签建议**：基于变更类型和范围
- **Base 分支**：默认 `main`，如果项目有 release 分支策略，询问确认
- **跨仓库 PR**：如果检测到 from fork，用 `--fork`

### 第六步：预览确认

组装预览给用户确认：

```
## PR 预览

**仓库**: <owner/repo>
**分支**: <head> → <base>
**标题**: <title>
**Draft**: yes/no

---
<description body>
---

确认提交？可修改:
- 标题 / 描述 / base 分支
- 或回复 "直接提交" 确认创建
```

### 第七步：提交

```bash
gc pr create -R <owner/repo> \
  --head <branch> \
  --base <base> \
  --title "<title>" \
  --body-file pr-body.md

# 如需要 draft 模式
gc pr create ... --draft

# 确认无误后打开浏览器查看
gc pr create ... --web
```

提交后展示 PR 链接，后续可触发 `gitcode-pr-review` 进行代码审查。

## 注意事项

- 提交前确认有远端推送权限，必要时先 `git push`
- 如果分支与远端不同步，提醒用户先推送
- 变更统计数字要准确（不夸张不缩小），用于 reviewer 快速判断规模
- 描述中的 "关联 Issue" 使用 GitHub/GitCode 支持的关键词（Fixes/Closes/Resolves + 编号）实现自动关闭
- 如果检测到多个 unrelated 变更混在同一个 PR，建议拆分

## 示例用法

**正常提交**：
```
我的 bugfix/issue-242 分支已经改完了，帮我创建 PR
```

**快速提交**：
```
当前分支推到远端了，直接帮我生成 PR 标题和描述然后创建
```

**Draft PR**：
```
我还在开发中，想先建个 draft PR 让团队看看方向
```

**跨仓库**：
```
我这个 fork 分支要提 PR 到上游的 main 分支，帮我创建
```
