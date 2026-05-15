# GitCode CLI Skills

GitCode 全生命周期 skills 集合，结合 [Superpowers](https://github.com/obra/superpowers)（本地开发循环）构成完整的 AI 编程工程体系。

## 架构

```
需求阶段              开发阶段              交付阶段
(远端 GitCode)        (本地 Superpowers)    (远端 GitCode)
    │                     │                     │
    ├─ issue-create       ├─ writing-plans      ├─ pr-create
    ├─ issue-review       ├─ TDD                ├─ pr-review
    └─ issue-triage       ├─ subagent-dev       └─ release-helper
                          └─ requesting-review
            ↑                     ↑                     ↑
            └─────── gitcode-dev-workflow（编排层）───────┘
```

- **Superpowers**：需求澄清 / 原子任务拆解 / TDD / 子代理隔离 / 任务间审查 / 收尾
- **gitcode skills**：Issue 管理 / PR 创建审查 / 安全审计 / Release 发布
- **`gitcode-dev-workflow`**：两者胶水层，知道每个阶段触发哪个 skill、何时交接

## Skill 矩阵

### 编排

| Skill | 做什么 |
|-------|--------|
| `gitcode-dev-workflow` | 全流程编排：idea → 需求 → 开发 → PR → 合并 → 发布 |

### Issue 流水线

| Skill | 时机 | 做什么 |
|-------|------|--------|
| `gitcode-issue-create` | 提交前 | 引导写 issue → 查重 → 模板填充 → 提交 |
| `gitcode-issue-review` | 开发前 | 需求分析 → 任务分解 → 实现方案 |
| `gitcode-issue-triage` | 提交后 | 分类 → 标签 → 优先级 → 分流 |

### PR 流水线

| Skill | 时机 | 做什么 |
|-------|------|--------|
| `gitcode-pr-create` | 提交时 | 检查变更 → PR 标题描述 → 提交 |
| `gitcode-pr-review` | 提交后 | 6维审查 + 行内评论 + 整体报告 + 发布 |

### 辅助技能

| Skill | 时机 | 做什么 |
|-------|------|--------|
| `gitcode-security-check` | 审查时 | 代码安全审计：凭证/注入/认证/依赖/加密 |
| `gitcode-release-helper` | 发布时 | Release note + 资产 + 清单 + 执行 |
| `gitcode-repo-onboarding` | 入门时 | 仓库结构 → 构建 → 测试 → 贡献流程 |

## 快速开始

### 前提

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- 已安装并配置 `gc` CLI（GitCode 命令行工具）

### Step 1：安装 Superpowers（本地开发循环）

```bash
# 在 Claude Code 中执行（从官方插件市场安装）
/plugin install superpowers
```

这会自动加载 13 个 skill，包括 brainstorming、writing-plans、TDD、subagent-driven-development、requesting-code-review、finishing-a-development-branch 等。

验证：
```bash
ls ~/.claude/skills/ | grep -E "brainstorming|writing-plans|test-driven"
```

### Step 2：安装 gitcode skills（远端协作）

```bash
git clone git@gitcode.com:gitcode-cli/gitcode-cli-skills.git /tmp/gitcode-skills
cp -r /tmp/gitcode-skills/gitcode-* ~/.claude/skills/
```

验证：
```bash
ls ~/.claude/skills/ | grep gitcode
# 应看到 9 个 gitcode-* 目录
```

### Step 3：验证

```bash
# 重启 Claude Code 或在会话中查看可用 skills
# 应包含 Superpowers 和 gitcode skills 的全部 skill
```

开始使用：
```
启动开发工作流，我要做 X
```

## 典型工作流

```
"I have an idea"           → brainstorming → issue-create → issue-review
"Let me implement"         → writing-plans → TDD → subagent-dev
"Code is ready, ship it"   → pr-create → pr-review → merge → release-helper
```

## 设计原则

- **步骤化工作流**：每个 skill 有明确的步骤顺序，不跳步
- **先验证再行动**：PR 描述中的声明先 grep 验证，Issue 创建前先查重
- **模板驱动**：PR/Issue 使用固定模板，确保内容完整
- **确认后操作**：涉及远端写操作先预览后执行
- **生态互补**：本地 (Superpowers) + 远端 (gitcode) 明确分工，不重叠

## 贡献

新增或优化 skill 后，使用 `/skill-creator` 进行测试验证。
