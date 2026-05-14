# GitCode CLI Skills

GitCode 全生命周期 skills 集合，覆盖 Issue、PR、Release、安全审计、仓库入门等场景。

## Skill 矩阵

### Issue 流水线

| Skill | 时机 | 做什么 | 状态 |
|-------|------|--------|------|
| `gitcode-issue-create` | 提交前 | 引导写 issue → 查重 → 模板填充 → 提交 | 已验证 |
| `gitcode-issue-triage` | 提交后 | 分类 → 标签 → 优先级 → 分流 | 已优化 |
| `gitcode-issue-review` | 开发前 | 需求分析 → 任务分解 → 实现建议 | 已验证 |

### PR 流水线

| Skill | 时机 | 做什么 | 状态 |
|-------|------|--------|------|
| `gitcode-pr-create` | 提交时 | 检查变更 → 生成 PR 标题描述 → 提交 | 已验证 |
| `gitcode-pr-review` | 提交后 | 多维代码审查 → 行内评论 → 整体报告 → 发布 | 已验证 |

### 辅助技能

| Skill | 时机 | 做什么 | 状态 |
|-------|------|--------|------|
| `gitcode-release-helper` | 发布时 | Release note → 资产管理 → 清单 → 执行 | 已优化 |
| `gitcode-security-check` | 审查时 | 代码安全审计：凭证/注入/认证/依赖/加密 | 已优化 |
| `gitcode-repo-onboarding` | 入门时 | 仓库结构 → 构建 → 测试 → 贡献流程 | 已优化 |

## 使用方式

```bash
# 克隆
git clone git@gitcode.com:gitcode-cli/gitcode-cli-skills.git /tmp/gitcode-skills

# 拷贝到 skills 目录
cp -r /tmp/gitcode-skills/gitcode-* ~/.claude/skills/
```

Claude Code 会自动加载 `~/.claude/skills/` 下的所有 skill 目录。

## 设计原则

- **步骤化工作流**：每个 skill 有明确的步骤顺序，不跳步
- **先验证再行动**：PR 描述中的声明先 grep 验证，Issue 创建前先查重
- **模板驱动**：PR/Issue 使用固定模板，确保内容完整
- **确认后操作**：涉及远端写操作（创建 Issue/PR、发布评论）先预览后执行
- **生态互补**：同领域 skill 明确分工边界，可链式触发

## 贡献

新增或优化 skill 后，使用 `/skill-creator` 进行测试验证。
