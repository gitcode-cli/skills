# Superpowers + gitcode skills 集成指南

## 体系架构

```
gitcode-dev-workflow（编排层）
    ├── 需求阶段
    │   ├── Superpowers: brainstorming
    │   └── gitcode: issue-create, issue-review
    ├── 开发阶段
    │   ├── Superpowers: writing-plans, TDD, subagent-dev, in-branch-review
    │   └── gitcode: (无 — 全部本地)
    └── 交付阶段
        ├── gitcode: pr-create, pr-review, release-helper
        └── Superpowers: finishing-a-development-branch
```

## 两个体系的分工

### Superpowers 的能力（本地开发循环）

| Skill | 输入 | 输出 | 触发频率 |
|-------|------|------|---------|
| brainstorming | 模糊想法 | 设计文档 + 验收标准 | 每个需求一次 |
| writing-plans | 规格说明 | 2-5min 原子任务列表 | 每需求一次 |
| TDD | 原子任务 | RED-GREEN-REFACTOR 循环 | 每任务多次 |
| subagent-driven-dev | 原子任务 + 代码上下文 | 通过测试的实现 | 每任务一次 |
| requesting-code-review | 完成的原子任务 | 问题报告 | 每任务一次 |
| finishing | 完成的开发分支 | merge/PR/discard 决策 | 每分支一次 |

### gitcode skills 的能力（远端协作）

| Skill | 输入 | 输出 | 触发频率 |
|-------|------|------|---------|
| issue-create | 需求描述 | GitCode Issue | 每需求一次 |
| issue-review | Issue 编号 | 需求分析 + 任务分解 + 实现方案 | 每 Issue 一次 |
| pr-create | 当前分支变更 | GitCode PR | 每 PR 一次 |
| pr-review | PR 编号 | 6维度审查报告 + 行内评论 | 每 PR 一次 |
| release-helper | 版本号 | release note + checklist + release | 每版本一次 |

## 关键交接点

### 1. brainstorming → issue-create

触发条件：Superpowers 完成了需求澄清，输出包含"做什么 + 验收标准"

操作：
1. 将 brainstorming 的输出整理为 gitcode-issue-create 需要的格式
2. 如果 gitcode-issue-create 查到重复 issue，回到 brainstorming 重新评估范围
3. 创建 issue 后，将 GitCode issue URL 交给 issue-review

### 2. issue-review → writing-plans

触发条件：issue-review 输出包含"技术可行 + 需求分解"

操作：
1. 将 issue-review 的 "需求分解" 节作为 writing-plans 的输入
2. Superpowers 进一步拆解为 2-5min 粒度（issue-review 的分解可能是小时级）
3. 超级原子任务保持与 gitcode issue 的子任务列表一致

### 3. TDD + subagent 完成 → pr-create

触发条件：所有原子任务通过 + 全部测试绿 + 审查通过

操作：
1. gitcode-pr-create 检查当前分支的 commit 历史和 diff
2. 自动生成 PR 标题（使用 conventional commit 格式）
3. 在 PR 描述中自动引用关联的 gitcode issue 编号

### 4. pr-create → pr-review

触发条件：PR 已创建

操作：
1. gitcode-pr-review 读取 PR diff 和已有评论
2. 按 6 个维度审查（代码质量/安全/架构/测试/文档/易用性）
3. 生成行内评论和整体报告并发布到 PR 评论区

### 5. merge → release-helper

触发条件：PR 已合并，需要发布新版本

操作：
1. gitcode-release-helper 收集 merge 后的 commits
2. 生成 release note 和 checklist
3. 创建 release 并上传资产

## 安装

```bash
# 1. 安装 Superpowers
git clone https://github.com/obra/superpowers ~/.claude/superpowers
# 按照 Superpowers 的 README 完成 skill 注册

# 2. 安装 gitcode skills
git clone git@gitcode.com:gitcode-cli/gitcode-cli-skills.git /tmp/gitcode-skills
cp -r /tmp/gitcode-skills/gitcode-* ~/.claude/skills/

# 3. 验证
# 输入 "启动开发工作流，我要做 X" 应该触发 gitcode-dev-workflow
```

## 不引入完整 Superpowers 时的替代方案

如果你只需要 Superpowers 的某些部分，可以只安装对应的 skill：

```bash
# 只装 TDD + 子代理开发
cp -r ~/.claude/superpowers/test-driven-development ~/.claude/skills/
cp -r ~/.claude/superpowers/subagent-driven-development ~/.claude/skills/
```

gitcode skills 自身已经覆盖了需求管理、PR 审查和发布，不需要 Superpowers 的全部 6 个 skill。
