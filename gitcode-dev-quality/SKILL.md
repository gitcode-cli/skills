---
name: gitcode-dev-quality
description: |
  本地代码质量关卡：构建验证、全量单元测试执行、覆盖率检查、代码格式化、
  静态分析、pre-commit 检查。在提交 PR 之前作为最后一道质量门禁。

  TRIGGER when: 用户要检查代码质量、跑测试、构建验证、lint检查、
  覆盖率检查、提交前检查、或说 "质量检查"、"跑一下测试"、"check quality"、
  "pre-commit check"、"验证构建"、"跑覆盖率"、"代码规范检查"。
---

# 本地代码质量门禁

PR 提交前的最后一道防线：构建 → 测试 → 覆盖率 → lint → 分析，全绿才能提交。

## 质量管道

```
构建验证 ──→ 全量单元测试 ──→ 覆盖率检查 ──→ 代码格式化 ──→ 静态分析 ──→ 通过
   ↓ fail        ↓ fail         ↓ fail        ↓ fail        ↓ fail
  立即停止      立即停止       立即停止      立即停止      立即停止
```

**任何一步失败都不允许进入下一步，更不允许提交 PR。**

## 检查步骤

### 1. 构建验证

确保代码能编译通过：

```bash
# Go
go build ./...

# Java / Maven
mvn compile -q

# Python
python -m compileall src/ -q

# Node.js / TypeScript
npm run build 2>/dev/null || npx tsc --noEmit
```

判断标准：exit code = 0 且无 error 输出。

### 2. 全量单元测试

**不只是当前变更的测试，是全部测试：**

```bash
# Go（含 race 检测）
go test ./... -race -count=1

# Java / Maven
mvn test

# Python
python -m pytest --tb=short

# Node.js
npm test
```

检查项：
- 全部测试通过（exit code = 0）
- 无 skipped 测试（除非有明确理由）
- 无 panic / segfault / timeout

### 3. 覆盖率检查

```bash
# Go
go test ./... -coverprofile=coverage.out
go tool cover -func=coverage.out | grep total

# Java / Maven (需 jacoco 插件)
mvn verify -Pcoverage

# Python
python -m pytest --cov=src/ --cov-report=term-missing
```

判断标准：
- 新增代码的增量覆盖率 ≥ 80%
- 整体覆盖率不下降（相比上次运行）

### 4. 代码格式化

```bash
# Go
gofmt -l .          # 检查是否有未格式化的文件
gofmt -w .          # 自动修复

# Java
mvn spotless:check 2>/dev/null || mvn formatter:format

# Python
ruff format --check . 2>/dev/null || black --check .

# 通用
pre-commit run --all-files 2>/dev/null
```

### 5. 静态分析

```bash
# Go
go vet ./...
golangci-lint run ./... 2>/dev/null || staticcheck ./...

# Java
mvn pmd:check 2>/dev/null || mvn spotbugs:check

# Python
ruff check . 2>/dev/null || pylint src/

# 通用：检查 TODO/FIXME 残留
grep -rn "TODO\|FIXME\|HACK" --include="*.go" --include="*.java" --include="*.py" . \
  | grep -v "_test\|test_" | grep -v "vendor/"
```

### 6. Pre-commit 检查（如项目已配置）

```bash
# 运行项目的 pre-commit 配置
pre-commit run --all-files 2>/dev/null

# 或手动检查 .pre-commit-config.yaml
cat .pre-commit-config.yaml 2>/dev/null
```

## 输出报告

```markdown
## Quality Gate Report

### 构建
- 状态: ✅/❌
- 耗时: <time>
- 错误: <errors if any>

### 全量单元测试
- 通过: N / 总计: N
- 耗时: <time>
- 失败: <list if any>

### 覆盖率
- 当前: X%
- 增量: Y%
- 状态: ✅/⚠️/❌

### 格式化
- 状态: ✅/❌
- 未格式化文件: <count>

### 静态分析
- 状态: ✅/⚠️/❌
- Warnings: <count>
- Errors: <count>

### 最终判定
[ ] 全部通过 — 可提交 PR
[ ] 有警告 — 建议修复后提交
[ ] 有错误 — 必须修复才能提交
```

## 注意事项

- 质量门禁是**本地操作**，不涉及远端仓库
- 任何一项失败就停止，不要跳到下一项
- 如果项目没有配置某个检查工具，标记为 N/A 而不是强制安装
- 覆盖率目标是 80%（增量），不是绝对值（旧项目可能整体覆盖率低）
- 通过质量门禁后，下一步是 `gitcode-pr-create`
- 本 skill 是 `gitcode-dev-workflow` 中开发阶段→交付阶段的必过关卡
