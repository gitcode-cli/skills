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

本 skill 依赖以下两个体系：

- **Superpowers**：通过 `/plugin install superpowers` 从 Claude 官方插件市场安装
- **gitcode skills**：`git clone` + `cp -r gitcode-* ~/.claude/skills/`

两个体系必须都已安装才能使用完整工作流。详见 `references/integration-guide.md`。

## 工作流总览

```
需求阶段 ──── Superpowers                   → gitcode skills
  ├─ 头脑风暴      brainstorming          (本地问答)
  ├─ 创建 Issue                             → issue-create
  └─ 需求评审                               → issue-review

开发阶段 ──── Superpowers
  ├─ 任务拆解      writing-plans          (原子任务)
  ├─ TDD 循环      TDD skill              (RED-GREEN-REFACTOR)
  ├─ 子代理执行    subagent-driven-dev    (每任务独立会话)
  └─ 任务间审查    requesting-code-review (快速审查)

交付阶段 ──── Superpowers                   → gitcode skills
  ├─ 提交 PR                                → pr-create
  ├─ 多维审查                               → pr-review
  ├─ 收尾          finishing              (测试验证 + MR 选项)
  └─ 发布                                   → release-helper
```

## 阶段详解

### 阶段一：需求（想法 → 清晰规格）

**触发**：用户说 "我想做一个 X" 或 "我有个需求"

**本地 — Superpowers brainstorming**：
```
用苏格拉底式提问澄清：
- 这个功能为谁解决什么问题？
- 成功的验收标准是什么？
- 什么场景下使用？什么人使用？
- 有没有现有方案可以参考？
- 边界在哪里（做什么 vs 不做什么）？
```
输出：非正式的需求阐述 + 功能边界 + 验收标准

**远端 — gitcode skills**：
```
# 提交为正式 Issue
→ gitcode-issue-create（查重 + 模板 + 提交）

# 技术深度评审
→ gitcode-issue-review（需求分析 + 任务分解 + 实现方案）
```

**交接条件**：Issue 已创建 + 需求分析报告已有 + 明确可开始编码

### 阶段二：开发（需求 → 可运行代码）

**本地 — Superpowers writing-plans**：
- 将需求拆为 2-5min 可完成的原子任务
- 每个任务独立的验证标准
- 按依赖关系排序

**本地 — Superpowers TDD**（每个原子任务循环）：
```
1. RED   —— 写失败测试（先定义预期行为）
2. GREEN —— 写最小实现让测试通过
3. REFACTOR —— 优化代码结构但保持测试绿色
```

**本地 — Superpowers subagent-driven-dev**：
- 每个原子任务由独立 AI 会话处理
- 避免上下文膨胀和漂移
- 每次会话只看到：任务描述 + 相关代码 + 测试要求

**本地 — Superpowers requesting-code-review**：
- 每个原子任务完成后触发快速审查
- 报告问题严重性（blocker / major / minor）
- 必须通过才能进入下一个任务

**交接条件**：所有原子任务完成 + 全部测试绿色 + 任务间审查全部通过

### 阶段三：交付（代码 → 合并发布）

**远端 — gitcode skills**：
```
# 生成 PR 标题和描述
→ gitcode-pr-create（检查变更 + conventional commit + 提交）

# 全面代码审查
→ gitcode-pr-review（6维度 + 行内评论 + 整体报告）
```

**本地 — Superpowers finishing-a-development-branch**：
- 验证所有测试通过
- 检查 CI 状态
- 提供选项：merge / PR / discard

**远端 — gitcode skills**：
```
# 如需发布
→ gitcode-release-helper（release note + checklist + 执行）
```

## 每个阶段的进入条件

| 阶段 | 进入条件 | 退出条件 |
|------|---------|---------|
| 需求 | 用户有模糊想法 | Issue + 需求分析报告 |
| 开发 | 验收标准明确，可行 | 全测试绿 + 审查通过 |
| 交付 | 代码 ready，commit push | PR merged + release 发布(可选) |

## 快速开始

**完整流程**：
```
启动 gitcode-dev-workflow，我有个需求：给 gc CLI 加 pr label 子命令
```

**从中间进入**：
```
当前在 PR #180 审查阶段，帮我把剩下的交付流程走完
```

## 注意事项

- 每个阶段完成后显式确认，再进入下一阶段
- 阶段一和二严格串行（需求不清不编码），阶段二内可并行（独立原子任务）
- Superpowers 负责本地开发和执行决策，gitcode skills 负责远端平台操作和正式记录
- 如果用户需求已经很清晰（已有 Issue + 分析报告），可以跳过阶段一直接从阶段二开始
- 详细的分工和技能映射参考 `references/integration-guide.md`
