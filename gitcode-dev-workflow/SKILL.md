---
name: gitcode-dev-workflow
description: |
  全流程工程体系编排器：串联 Superpowers（本地开发循环）和 gitcode skills
  （远端协作），从需求澄清、原子任务拆解、TDD 编码，到 PR 提交、多维审查、
  发布交付。覆盖从 idea 到 merge 的完整软件开发生命周期。

  TRIGGER when: 用户要开始一个新需求、启动开发工作流、走完整工程流程、
  或说 "start workflow"、"开始开发"、"新功能开发"、"完整流程"、"从需求到交付"、
  "走一遍全流程"。当用户有模糊想法但不知道从哪步开始时，这应该是第一个触发的 skill。
---

# GitCode 完整开发工作流

串联 **Superpowers（本地开发）** + **gitcode skills（远端协作）**，覆盖从需求到交付的全生命周期。

## 前提

- **Superpowers**：`/plugin install superpowers`
- **gitcode skills**：`git clone` + `cp -r gitcode-* ~/.claude/skills/`

## 🚨 强制执行规则

**以下规则不可跳过，任何一步不满足就不能进入下一步。**

### 阶段闸门

进入下一阶段前，必须**出示产出物证据**给用户确认：

| 阶段转换 | 必须出示的证据 | 未完成则 |
|----------|--------------|---------|
| 需求 → 开发 | Issue URL + 需求分析报告 | 🔒 不允许写一行代码 |
| 开发 → 质量关卡 | 全部原子任务清单（全部标记 done） | 🔒 不允许跑质量检查 |
| 质量关卡 → 交付 | 5 项检查全部绿色（build/test/coverage/format/static） | 🔒 不允许创建 PR |

### Issue 审计日志（强制）

**Issue 是全流程唯一的审计日志源。** 每个阶段的关键产出物必须以评论形式发布到关联 Issue 上：

```
Phase 1 完成 → gc issue comment <number> --body "需求分析 + 任务分解"
Phase 2 完成 → gc issue comment <number> --body "原子任务清单 + 测试报告"
Quality Gate → gc issue comment <number> --body "质量门禁报告 (build/test/coverage/format/static)"
Phase 3 完成 → gc issue comment <number> --body "PR #N 已创建 + 审查结论"
```

任何人不看对话记录，仅通过 Issue 评论区，就能完整追溯从需求到交付的全过程。

### TDD 强制规则

```
每个原子任务 = RED → GREEN → REFACTOR，缺一不可。

RED:   先写测试，确认测试失败（验证测试有效）
GREEN: 写最小实现让测试通过（不写多余代码）
REFACTOR: 测试通过后优化结构（不改行为）

禁止: 先写实现再补测试
禁止: 合并多个原子任务一次完成
```

### 子代理隔离规则

```
每个原子任务由独立 AI 子代理执行：
- 子代理只看到：任务描述 + 相关代码 + 测试要求
- 子代理不得访问：其他任务上下文、全局设计决策

强制使用 Subagent 工具，组名: workflow-<task-id>
```

## 工作流总览

```
Phase 1: 需求               Phase 2: 开发               🔒 Quality Gate            Phase 3: 交付
──────────────────────────────────────────────────────────────────────────────────────────
brainstorming               writing-plans               build      ✅/❌            pr-create
issue-create    →闸门→      TDD (per task)              test       ✅/❌  →闸门→    pr-review
issue-review                subagent-dev (isolated)     coverage   ✅/❌            finishing
                            requesting-review (per)     format     ✅/❌            release-helper
     ↓                             ↓                   static     ✅/❌                  ↓
产出: Issue URL              产出: 全绿原子任务清单      产出: Quality Report       产出: Merged PR
```

## 阶段详解

### 阶段一：需求

1. **brainstorming** — 最少问清 4 个问题：为谁解决什么 / 验收标准 / 边界（做什么不做什么） / 参考方案
2. **issue-create** — 查重 → 模板填充 → 提交
3. **issue-review** — 独立验证 PR 描述声明 + 代码库搜索 + 需求分解 + 实现方案

**闸门**：向用户展示 Issue URL + 需求分析报告，确认后**将报告作为 Issue 评论发布**，然后进入阶段二。

### 阶段二：开发

1. **writing-plans** — 拆为 2-5min 原子任务（不是小时级，不是天级）
2. **TDD** — 每个任务严格 RED→GREEN→REFACTOR
3. **subagent-driven-dev** — 每个任务独立子代理执行，组名 `workflow-<task-id>`
4. **requesting-code-review** — 每个任务完成后触发快速审查

**闸门**：展示原子任务清单 + 测试结果，确认后**将清单作为 Issue 评论发布**，然后进入质量关卡。

### 阶段三：质量关卡

按顺序执行，**任何一项失败立即停止**：
1. `mvn compile` / `go build ./...`
2. `mvn test` / `go test ./... -race -count=1`
3. 覆盖率（增量 ≥ 80%）
4. 格式化（`gofmt -l .` / `mvn spotless:check`）
5. 静态分析（`go vet` / `golangci-lint run` / `mvn pmd:check`）

**闸门**：5 项全部绿色，出示 Quality Report，确认后**将报告作为 Issue 评论发布**，然后进入阶段四。

### 阶段四：交付

1. **pr-create** — 检查变更 → conventional commit 标题 → PR 描述 → 创建
2. **pr-review** — 6 维审查 + **行内评论**（必须，不是可选的）
3. **finishing** — 全测试通过 + merge 决策

## 合规检查清单

每个阶段结束后，用户确认前，输出此清单：

```
Phase N 合规检查:
  [ ] 步骤 1: <step> — <evidence>
  [ ] 步骤 2: <step> — <evidence>
  ...
  [ ] Issue 评论已发布: <comment_link>
  
  缺失项: <list or "无">
  是否允许进入下一阶段: [ ]
```
