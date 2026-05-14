---
name: gitcode-issue-create
description: |
  帮助用户撰写并创建高质量 GitCode Issue。自动识别 issue 类型（bug/feature/enhancement），
  引导补充缺失信息，搜索已有 issue 查重，建议标签和优先级，预览确认后通过 gc CLI 提交。

  TRIGGER when: 用户要创建/提交 issue、报 bug、提 feature、写 issue、
  提交问题、创建需求、开 issue、report bug、create issue、file issue、
  或者描述了一个需要提交到 GitCode 的问题/需求。只要涉及"把问题/需求写成
  ticket 提交"，即应触发。
---

# GitCode Issue 创建助手

引导用户从零写出高质量 Issue，查重、填模版、建议标签优先级、确认后提交。

## 与现有 skill 的分工

| Skill | 时机 | 做什么 |
|-------|------|--------|
| **gitcode-issue-create（本 skill）** | Issue **创建前** | 从零撰写、查重、引导填充、提交 |
| `gitcode-issue-triage` | Issue **创建后** | 对已有 issue 分类贴标签分流 |
| `gitcode-issue-review` | 开发**动手前** | 深度需求分析、任务分解、实现建议 |

三个 skill 覆盖 issue 全生命周期：创建 → 分流 → 落地。

## 核心工作流

### 第一步：理解用户意图

从用户的描述中提取核心信息：
- **类型判断**：这是个什么问题？（bug / feature / enhancement / question）
- **目标仓库**：要提交到哪个 repo？（如未提供，根据上下文推断或询问）
- **已有素材**：用户已经提供了多少信息？（标题？描述？版本号？截图？）

类型判断规则：
- 描述含"报错/坏了/不工作/bug/异常/崩溃/crash" → bug
- 描述含"添加/新增/希望支持/能不能/需要 XXX 功能" → feature
- 描述含"改进/优化/重构/更好" → enhancement
- 不确定时直接问用户

### 第二步：查重

**这一步必须执行。** 在写 issue 之前先搜索是否已有相同/相似 issue。

```bash
# 用关键词搜索已有 issue（open 和 closed 都要搜）
gc issue list -R <owner/repo> --search "<关键词1>" --state all --json
gc issue list -R <owner/repo> --search "<关键词2>" --state open --json

# 如果 repo 有已知的长期 issue（如"统一支持 stdin"），也应提及
```

查重结果处理：
- **找到精确重复** → 告知用户已有 issue 编号和状态，询问是否继续提交还是直接参与已有讨论
- **找到相关但不相同** → 在 issue 描述中引用关联 issue 编号
- **未找到重复** → 继续下一步

### 第三步：按模板引导填充

根据 issue 类型使用对应模板。不要一次性问所有问题——先让用户确认已说清的部分，再逐条追问缺失项。

#### Bug Report 模板

```markdown
## 问题现象
[简要描述发生了什么]

## 复现步骤
1.
2.
3.

## 期望行为
[期望的正确行为是什么]

## 实际行为
[实际发生了什么，包含错误信息/日志/截图]

## 环境信息
- gc 版本: [如 0.5.0]
- OS: [如 macOS 14 / Ubuntu 22.04 / Windows 11]
- Shell: [如 zsh / bash / PowerShell]

## 影响范围
[影响了哪些功能或场景，严重程度]

## 补充信息
[截图、日志、关联 issue/PR]
```

#### Feature Request 模板

```markdown
## 需求背景
[为什么需要这个功能？解决什么问题？]

## 功能描述
[具体要做什么？描述交互方式和预期效果]

## 使用场景
[什么情况下会用到？]

## 验收标准
- [ ] [描述完成标准1]
- [ ] [描述完成标准2]

## 参考实现
[其他工具/平台的类似功能，如有]

## 关联 issue/PR
[相关的 issue 或 PR 编号，如有]
```

#### Enhancement 模板

```markdown
## 当前现状
[现在的实现/行为是什么]

## 改进方向
[希望改成什么样，为什么更好]

## 影响范围
[涉及哪些模块、是否破坏性变更]

## 验收标准
- [ ] [描述完成标准]
```

### 第四步：建议标签和元数据

基于 issue 内容建议：

- **类型标签**：`type/bug` / `type/feature` / `type/enhancement` / `type/docs`
- **范围标签**：如 `scope/api` / `scope/pr` / `scope/issue` / `scope/cli`
- **优先级**：
  - P0/Critical — 生产阻塞、安全漏洞、数据丢失
  - P1/High — 主要功能阻断
  - P2/Medium — 重要但非紧急
  - P3/Low — 边缘场景、可延后
- **优先级标签**：`priority-high` / `priority-medium` / `priority-low`

建议前先查看仓库有哪些现有标签：
```bash
gc label list -R <owner/repo>
```

### 第五步：预览确认

组装完整 issue 内容，展示预览给用户确认：

```
## Issue 预览

**仓库**: <owner/repo>
**标题**: <title>
**标签**: <label1>, <label2>

---
<body content>
---

确认提交？可修改:
- 标题 / 描述 / 标签
- 或回复 "直接提交" 确认
```

### 第六步：提交

```bash
gc issue create -R <owner/repo> \
  --title "<title>" \
  --body "<body>" \
  --label "<label1>,<label2>"
```

提交后展示 issue 链接，方便用户查看。

## 关键原则

- **查重先于提交**：宁可多搜一次，不重复提 issue。这是对维护者最大的尊重
- **引导优于填充**：缺信息时先问用户，不做猜测性填充。错误的猜测比缺失信息更糟糕
- **模板是起点不是枷锁**：简单问题允许精简模板，关键信息不缺即可
- **确认后提交**：绝不未经确认直接创建 issue，尤其是涉及远程仓库操作
- **仓库标签为准**：标签建议基于仓库实际标签体系，不做空中楼阁

## 注意事项

- 如果用户没有指定仓库，必须先问清楚再继续
- 搜索查重时注意搜索已关闭的 issue（问题可能已被修复）
- 如果用户提供的截图/日志含敏感信息（密码、token），提醒用户脱敏
- issue 标题要简洁但自描述：`bug(scope): 简短描述` 或 `feat(scope): 简短描述`
- 描述中避免评价性语言（"这太糟糕了"），保持事实陈述

## 示例用法

**报 Bug**：
```
我发现在 gitcode-cli/cli 上 gc issue edit --body-file - 不接受 stdin，
但帮助里写了支持。帮我提个 issue
```

**提 Feature**：
```
gc pr 没有 label 子命令，但 gc issue 有。我想给 gitcode-cli/cli 提个 feature request
```

**不确定类型**：
```
gc milestone view 返回结果有时候 JSON 里字段是 0 但实际没有 issue，
不确定算 bug 还是 enhancement，帮我整理一下提交
```
