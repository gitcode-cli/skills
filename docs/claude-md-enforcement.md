# 项目 CLAUDE.md 强制工作流模板

将以下内容追加到项目根目录的 `CLAUDE.md`。CLAUDE.md 始终在 Claude 上下文中，
这些规则会作为持续生效的约束。

---

## 🚨 开发工作流强制规则

### 优先级

以下规则**覆盖**所有其他行为偏好。任何情况下不得跳过。

### 阶段闸门

进入下一阶段前必须向用户展示产出物并获得确认：

```
Phase 1 (需求)    → Gate: Issue URL + 需求分析报告
Phase 2 (开发)    → Gate: 原子任务清单全绿 + 测试报告
Phase 3 (质量)    → Gate: 5项检查全绿 (build/test/coverage/format/static)
Phase 4 (交付)    → Gate: PR URL + Review Report
```

未通过闸门时：
- 不允许写下一阶段的代码
- 不允许跳过闸门
- 如果用户说"先跳过"，提醒后果并再次确认

### TDD 强制

每个原子任务必须按 RED → GREEN → REFACTOR 执行。

| | 禁止 | 要求 |
|---|------|------|
| ❌ | 先写实现再补测试 | 先写测试，确认失败 |
| ❌ | 合并多个任务一次完成 | 每次只做一个任务 |
| ❌ | 跳过 REFACTOR | 绿了之后优化结构 |

### 子代理强制

每个原子任务启动独立子代理：

```
Agent(
  team_name: "workflow-<task-id>",
  prompt: "任务描述 + 相关代码 + 测试要求",
  mode: "auto"
)
```

单会话不得执行超过 1 个原子任务。

### 质量门禁

提交前必须运行 gitcode-dev-quality。流程：

```
build → test → coverage(≥80%) → format → static-analysis
  ↓        ↓         ↓               ↓           ↓
 任何一项失败 = 停止，不允许 commit
```

### Issue 审计日志

**Issue 是全流程唯一的审计日志源。** 每个阶段产出物以评论发布到关联 Issue：

```
Phase 1 完成 → gc issue comment <number> --body "需求分析报告"
Phase 2 完成 → gc issue comment <number> --body "原子任务清单 + 测试结果"
Quality Gate → gc issue comment <number> --body "质量报告 (build/test/coverage/format/static)"
Phase 3 完成 → gc issue comment <number> --body "PR #N + 审查结论"
```

不看对话记录，只看 Issue 评论区就能完整追溯全流程。

### 审计追踪

每次开发启动时记录：

```
## Workflow Audit: <feature-name>

| Phase | 步骤 | 状态 | 证据 |
|-------|------|------|------|
| 需求 | brainstorming | □ | - |
| 需求 | issue-create | □ | Issue URL |
| 需求 | issue-review | □ | Comment: <link> |
| 开发 | writing-plans | □ | Comment: <link> |
| 开发 | TDD (per task) | □ | - |
| 开发 | subagent (per task) | □ | - |
| 开发 | requesting-review | □ | - |
| 质量 | build/test/coverage/format/static | □ | Comment: <link> |
| 交付 | pr-create | □ | PR URL |
| 交付 | pr-review | □ | Review comment |
| 交付 | finishing | □ | - |

缺失步骤: <自动填充>
```
