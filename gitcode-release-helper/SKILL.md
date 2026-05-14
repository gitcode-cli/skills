---
name: gitcode-release-helper
description: |
  Assist with GitCode release preparation: generate release notes from commit
  history and merged PRs, manage release assets, create release checklist, and
  execute the release via gc CLI.

  TRIGGER when: user asks to prepare GitCode release, generate release notes,
  check release PRs, create release checklist, manage release assets, or phrases
  like "发布release", "生成release note", "release准备", "版本发布", "发布清单",
  "上传release", "下载release".
---

# GitCode Release Helper

辅助版本发布全流程：收集变更 → 生成 Release Note → 管理资产 → 创建清单 → 执行发布。

## 核心工作流

### 第一步：确定版本信息

```bash
# 查看已有 release 序列
gc release list -R <owner/repo> --json

# 查看里程碑（如有关联）
gc milestone list -R <owner/repo> --json

# 查看贡献统计
gc repo stats --branch main -R <owner/repo> --json
```

版本号规则：
- Major（X.0.0）：破坏性变更、架构调整
- Minor（0.X.0）：新功能、向后兼容增强
- Patch（0.0.X）：Bug 修复、小改进
- 预发布：alpha/beta/rc

### 第二步：收集变更内容

```bash
# 查看两个版本之间的 commit（最准确）
git log <previous_tag>..HEAD --oneline --no-merges

# 查看某个里程碑关联的 PR
gc milestone view <number> -R <owner/repo> --json

# 列出最近合并的 PR
gc pr list -R <owner/repo> --state merged --limit 50 --json

# 按时间范围查看
git log --since="2026-04-01" --until="2026-05-01" --oneline --no-merges

# 贡献者统计（用于 credit）
gc repo stats --branch main --since <date> --until <date> -R <owner/repo> --json
```

### 第三步：生成 Release Note

根据 commit 历史和合并 PR 分类生成：

```markdown
## Release v<version> (<date>)

### 新功能
- **<名称>**: <描述> (PR #<n>)

### 增强
- <描述> (PR #<n>)

### 修复
- **<模块>**: <描述> (PR #<n>)

### 文档
- <描述> (PR #<n>)

### 破坏性变更
- <变更描述及迁移指南>

### 贡献者
感谢 <名单> 对本版本的贡献
```

### 第四步：生成发布清单

```markdown
## Release v<version> Checklist

### 代码
- [ ] 所有计划 PR 已合并
- [ ] CI/CD 通过
- [ ] 代码审查完成

### 文档
- [ ] CHANGELOG 已更新
- [ ] README 更新（如有新功能）
- [ ] API 文档更新（如有变更）

### 测试
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 关键路径手动验证

### 发布
- [ ] 版本号确认
- [ ] Release Note 已定稿
- [ ] 发布时间确定

### 发布后
- [ ] Release 已创建
- [ ] 关联 milestone 已关闭
- [ ] 下一版本 planing
```

### 第五步：管理发布资产（可选）

```bash
# 构建产物
go build -o ./dist/gc ./cmd/gc

# 上传资产
gc release upload <tag> ./dist/gc -R <owner/repo>

# 下载已有资产
gc release download <tag> -R <owner/repo> --output ./downloads/

# 下载所有资产（含源码包）
gc release download <tag> -R <owner/repo> --all
```

### 第六步：执行发布

```bash
# 创建正式 release（从文件读取 notes）
gc release create v<version> -R <owner/repo> \
  --title "Release v<version>" \
  --notes-file RELEASE_NOTES.md \
  --target main

# 创建草稿（发布前 review）
gc release create v<version> -R <owner/repo> \
  --title "v<version>" \
  --notes-file RELEASE_NOTES.md \
  --draft

# 创建预发布
gc release create v<version>-rc1 -R <owner/repo> \
  --title "v<version>-rc1" \
  --notes-file RELEASE_NOTES.md \
  --prerelease

# 创建后需要修改
gc release edit v<version> -R <owner/repo> \
  --title "修正标题" \
  --notes-file UPDATED_NOTES.md
```

## 示例用法

```
为 gitcode-cli/cli 生成 v0.6.0 的 release note，基于最近一个月的合并 PR
```
```
检查 v1.0.0 发布前还有什么遗漏，生成 checklist
```
```
帮我把 v1.0.0 的构建产物上传到 release assets
```

## 注意事项

- Release Note 面向用户，避免过于技术化的实现细节
- 破坏性变更必须含升级/迁移指南
- 贡献者名单通过 `gc repo stats` 或 `git shortlog` 获取
- 使用 `--notes-file` 从文件读取长篇 notes，避免命令行长度限制
- `--draft` 和 `--prerelease` 可以组合使用
- 发布前确认 tag 已推送到远端
- 发布后关闭关联 milestone：`gc milestone edit <n> --state closed -R <owner/repo>`
