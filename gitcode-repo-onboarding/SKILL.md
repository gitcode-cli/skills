---
name: gitcode-repo-onboarding
description: |
  Help onboard contributors to GitCode repositories: understand repo structure
  via gc CLI, guide build and test procedures, generate contribution paths, and
  set up local dev environment.

  TRIGGER when: user asks to understand a GitCode repo structure, how to build
  or test a repo, how to contribute to a repo, how to set up local dev, or
  phrases like "仓库入门", "怎么构建", "怎么测试", "贡献指南", "repo结构",
  "上手仓库", "clone仓库", "本地搭建", "开发环境".
---

# GitCode Repo Onboarding

用 `gc` CLI 帮助新贡献者快速上手：了解仓库、搭建环境、掌握开发流程。

## 核心工作流

### 第一步：获取仓库概览

```bash
# 仓库基本信息
gc repo view <owner/repo> --json

# 仓库活跃度
gc repo stats --branch main -R <owner/repo> --json

# 查看可用命令（了解项目 CLI 能力）
gc schema
```

关键信息：描述、语言、默认分支、许可证、活跃度。

### 第二步：Clone 和 Fork

```bash
# 方式1：直接 clone
gc repo clone <owner/repo>

# 方式2：Fork 后 clone（有贡献意图时推荐）
gc repo fork <owner/repo> --clone

# 指定 clone 目录和分支
gc repo clone <owner/repo> my-dir --branch develop
```

### 第三步：分析项目结构

进入仓库后，理解代码组织：

```
项目根目录/
├── README.md          # 项目说明 + 快速开始（先看这个）
├── CONTRIBUTING.md    # 贡献指南（重要）
├── CLAUDE.md          # AI 开发指引
├── cmd/               # 入口点
├── pkg/               # 核心库代码
├── api/               # API 客户端层
├── docs/              # 文档
├── scripts/           # 工具脚本
├── Makefile           # 构建命令（常用入口）
├── go.mod / go.sum    # Go 依赖
├── .gitcode/          # GitCode 特有配置
└── .github/workflows/ # CI/CD 配置
```

识别构建系统：
```bash
# 查看 Makefile 有哪些可用 target
head -50 Makefile 2>/dev/null || echo "无 Makefile"

# 检查语言和构建工具
ls go.mod 2>/dev/null && echo "Go 项目" && head -5 go.mod
ls package.json 2>/dev/null && echo "Node.js 项目"
ls pyproject.toml setup.py 2>/dev/null && echo "Python 项目"
ls Cargo.toml 2>/dev/null && echo "Rust 项目"
```

### 第四步：构建和测试

```bash
# 构建（优先用 Makefile）
make build 2>/dev/null || go build ./...

# 运行测试
make test 2>/dev/null || go test ./...

# 查看 CI 流程了解完整验证步骤
ls .github/workflows/ && head -80 .github/workflows/*.yml 2>/dev/null
```

### 第五步：理解开发流程

检查仓库规范：

- Issue 模板：`ls .gitcode/ISSUE_TEMPLATE/` 或 `ls .github/ISSUE_TEMPLATE/`
- PR 模板：`ls .gitcode/PULL_REQUEST_TEMPLATE/` 或 `ls .github/PULL_REQUEST_TEMPLATE/`
- 贡献指南：`head -100 CONTRIBUTING.md`
- 代码规范：检查 `.editorconfig`、`golangci-lint` 配置

### 第六步：生成 Onboarding 指引

输出格式：

```markdown
## <repo_name> Onboarding Guide

### 项目概述
- 描述、语言、许可证、活跃状态

### 快速开始
```bash
gc repo clone <owner/repo>
cd <repo_name>
# 安装依赖 + 构建 + 测试
```

### 项目结构
<关键目录树 + 说明>

### 开发流程
1. Fork → Clone → 创建分支
2. 编码 → 测试 → 提交
3. Push → 创建 PR (`gc pr create`)
4. Code Review → Merge

### 建议第一步
- 好的 first issue: <列表>
- 本项目还可用这些 skill：gitcode-issue-create、gitcode-pr-create、gitcode-pr-review
```

## 示例用法

```
帮我上手 gitcode-cli/cli，我想知道怎么构建和跑测试
```
```
分析 gitcode-cli/cli 的项目结构，告诉我核心模块
```
```
我想给 gitcode-cli/cli 提 PR，告诉我完整流程
```

## 注意事项

- 先读 README.md 和 CONTRIBUTING.md（如果有）
- 优先使用 `gc repo clone` 而非 `git clone`（自动处理认证）
- Fork + Clone 用于无写权限的仓库，直接 Clone 用于有权限的仓库
- 检查 CI 配置了解项目的完整验证流程
- 复杂项目建议从标记 `good first issue` 的 issue 开始
