# 案例演示：openlibing-platform-release 全流程 AI 交付

> 演示如何使用 gitcode skills + Superpowers 完成从需求到交付的全生命周期开发

## 项目背景

**仓库**: `git@gitcode.com:aflyingto/openlibing-platform-release.git`

OpenLiBing Platform Release — 基于 Spring Boot 3.x + Java 21 + MongoDB 的企业级 OpenEuler Linux 发行版发布管理平台。

**技术栈**: Spring Boot / MongoDB / Redis / Apollo / Jenkins / 华为云 OBS / Docker

---

## 场景

平台上已有 CVE 管理功能，但每次只能手动逐条录入。运维团队每次安全扫描出来几十个 CVE，需要一个**批量导入**功能。

---

## 阶段一：需求（想法 → 清晰 Issue）

### Step 1 — 触发工作流

```
用户: 启动开发工作流，我有个需求：openlibing-platform-release 的 CVE 管理模块
      需要支持批量导入，现在每次只能手动录入一条，运维每次巡检扫出来几十个
      CVE，逐条录入太慢了

→ gitcode-dev-workflow 自动识别为"需求阶段"，触发 brainstorming
```

### Step 2 — brainstorming（苏格拉底式提问）

```
AI: 批量导入的具体场景是怎样的？用什么格式的文件？
用户: 运维执行完安全扫描，导出一个 CSV，里面有几十条 CVE 记录

AI: 每条 CVE 记录包含什么字段？
用户: CVE编号、影响组件、严重等级、描述、修复版本、发现时间

AI: 导入时如果 CVE 已存在怎么处理？覆盖还是跳过？
用户: 存在就跳过，不要覆盖已有数据

AI: 验收标准是什么？
用户: 上传一个 50 行的 CSV，30 秒内完成，显示成功/跳过/失败数量
```

### Step 3 — gitcode-issue-create（查重 + 提交）

```bash
# AI 自动执行
gc issue list -R aflyingto/openlibing-platform-release --search "CVE批量导入" --state all
# → 无重复

gc issue create -R aflyingto/openlibing-platform-release \
  --title "feat(CVE): 支持 CSV 批量导入 CVE 记录" \
  --body-file issue-body.md \
  --label "type/feature,scope/CVE,priority-medium"
```

生成的 Issue：

```markdown
## 需求背景
运维团队每次安全扫描产生数十条 CVE，目前只能逐条手动录入，
一次扫描结果需要 30 分钟以上才能全部录入完成。

## 功能描述
- 上传 CSV 文件（CVE编号、组件、级别、描述、修复版本、发现时间）
- 已存在的 CVE 自动跳过，不覆盖
- 返回导入结果：成功 X 条 / 跳过 X 条 / 失败 X 条（含失败原因）
- 支持预览：先解析 CSV 展示将要导入的记录，用户确认后再执行

## 验收标准
- [ ] 50 行 CSV 30 秒内完成
- [ ] 重复 CVE 正确跳过
- [ ] 格式错误的行给出具体错误信息
- [ ] API 文档 (Swagger) 自动生成
```

### Step 4 — gitcode-issue-review（需求分析 + 任务分解）

AI 自动搜索代码库：

```bash
cd /tmp/openlibing-platform
grep -r "CVE\|CveEntity\|CveController\|CveService" --include="*.java" src/
```

生成技术评审报告：

```markdown
## 需求分析

**核心诉求**: CVE 管理从单条操作升级为批量操作

**技术可行性**: 完全可行
- 现有 CveEntity + CveMapper 已有单条写入能力
- Java 有成熟的 CSV 解析库 (OpenCSV / Apache Commons CSV)
- Spring Boot 的文件上传 (MultipartFile) 可直接复用

**影响范围**: ~5 文件
- 新增: business/controller/CveBatchController.java
- 新增: business/dto/CveBatchImportResult.java
- 修改: business/service/CveService.java（加批量写入方法）
- 新增: business/service/CveBatchService.java
- 新增: common/utils/CsvParserUtil.java

## 需求分解

| # | 子任务 | 涉及文件 | 预估 | 依赖 |
|---|--------|----------|------|------|
| 1 | CSV 解析工具类 | CsvParserUtil.java | S (~3h) | - |
| 2 | 批量写入 Service | CveBatchService.java | S (~3h) | #1 |
| 3 | REST API 端点 | CveBatchController.java + DTO | S (~3h) | #2 |
| 4 | 单元测试 + Swagger | *Test.java | S (~2h) | #3 |
| 5 | 集成验证 | - | XS (~1h) | #4 |
```

**此时产出**：
- [x] GitCode Issue 已创建
- [x] 需求分析报告已有
- [x] 任务分解明确 → **可以进入开发阶段**

---

## 阶段二：开发（分析报告 → 可运行代码）

### Step 5 — Superpowers writing-plans（原子任务拆解）

将小时级任务拆为 2-5min 完成的原子单位：

```
原子任务 #1:  在 pom.xml 添加 OpenCSV 依赖                 (2min)
原子任务 #2:  创建 CsvParserUtil.java，实现 parseCveCsv()    (5min)
原子任务 #3:  写 CsvParserUtilTest，验证解析 50 行 CSV       (3min)
原子任务 #4:  创建 CveBatchImportResult.java DTO             (3min)
原子任务 #5:  在 CveService.java 添加 batchCreate()           (5min)
原子任务 #6:  写 CveServiceTest，验证重复跳过逻辑            (3min)
原子任务 #7:  创建 CveBatchService.java 编排解析+导入       (5min)
原子任务 #8:  创建 CveBatchController.java REST 端点         (5min)
原子任务 #9:  写 ControllerTest，验证文件上传和响应          (3min)
原子任务 #10: 运行全量测试确保无回归                        (2min)
```

### Step 6 — TDD 循环（以原子任务 #2 为例）

```
┌─ RED ─────────────────────────────────────────┐
│  @Test                                        │
│  void parseCveCsv_50rows_shouldReturn50() {   │
│      MultipartFile csv = mockCsv(50);         │
│      var records = CsvParserUtil.parse(csv); │
│      assertThat(records).hasSize(50);         │
│  }                                            │
│  → 编译失败（类不存在）— RED ✓                │
└───────────────────────────────────────────────┘
                    ↓
┌─ GREEN ───────────────────────────────────────┐
│  public static List<CveRecord> parse(         │
│      MultipartFile file) {                    │
│      try (CSVReader r = new CSVReader(...)) { │
│          return r.readAll().stream()           │
│              .map(CveRecord::fromRow)          │
│              .toList();                        │
│      }                                        │
│  }                                            │
│  → 测试通过 — GREEN ✓                         │
└───────────────────────────────────────────────┘
                    ↓
┌─ REFACTOR ────────────────────────────────────┐
│  // 提取常量: CSV_HEADERS                     │
│  // 添加行号追踪: AtomicInteger lineNum       │
│  // 优化异常处理: 捕获 CsvException 转业务异常 │
│  → 测试仍绿 — REFACTOR ✓                      │
└───────────────────────────────────────────────┘
```

### Step 7 — subagent-driven-dev（并行执行）

```
子代理 A: 原子任务 #1, #4           ─┐
子代理 B: 原子任务 #2, #3, #5, #6   ─┤ 并行
子代理 C: 原子任务 #7, #8, #9       ─┘

每个子代理:
- 只看自己的任务描述 + 相关代码片段
- 执行 TDD 循环
- 完成后触发 requesting-code-review
- 审查通过后任务标记为 done
```

---

## 阶段三：交付（代码 → PR → 审查 → 合并）

### Step 8 — gitcode-pr-create

```bash
# AI 检查变更
cd /tmp/openlibing-platform
git diff --stat origin/main...HEAD
# 5 files changed, +210/-5

git add -A
git commit -m "feat(CVE): add CSV batch import for CVE records"
git push origin feat/cve-batch-import

# 生成并创建 PR
gc pr create -R aflyingto/openlibing-platform-release \
  --title "feat(CVE): 支持 CSV 批量导入 CVE 记录" \
  --body-file pr-body.md \
  --head feat/cve-batch-import \
  --base main
```

自动生成的 PR 描述：

```markdown
## 变更说明
新增 CVE CSV 批量导入功能，支持一次上传 50+ 条 CVE 记录。
解决运维团队安全扫描后逐条录入的低效痛点。

## 变更内容
- 新增 CsvParserUtil.java：CSV 解析，含错误行号追踪
- 新增 CveBatchService.java：批量导入编排逻辑（解析→校验→写入）
- 新增 CveBatchController.java：POST /api/v1/cve/batch 端点
- 修改 CveService.java：新增 batchCreate() 批量写入方法
- 新增 CveBatchImportResult.java：导入结果 DTO
- 新增 5 个单元测试 + 1 个集成测试

## 关联 Issue
Fixes #1

## 测试计划
- [x] 解析 50 行 CSV < 1s
- [x] 重复 CVE 正确跳过
- [x] 格式错误行返回具体错误描述
- [x] 86 个原有测试全部通过
- [ ] 手动验证：上传真实扫描导出的 CSV 文件
```

### Step 9 — gitcode-pr-review

AI 执行 6 维度审查：

```bash
gc pr diff <number> -R aflyingto/openlibing-platform-release
gc pr comments <number> -R aflyingto/openlibing-platform-release --json
```

审查发现并发布行内评论：

```
[安全] CsvParserUtil.java:42
  文件大小未校验，恶意超大文件可能导致 OOM
  建议: 添加 @Size(max = 10MB) 注解

[代码质量] CveBatchService.java:28
  批量写入无事务保护，中途失败会留下部分数据
  建议: 添加 @Transactional(rollbackFor = Exception.class)

[测试] CveBatchService.java:batchCreate
  缺少空列表输入的边界测试
  建议: 添加 verify(batchCreate(emptyList)) 测试
```

PR 评论页面上可以看到 3 条行内评论 + 1 条整体报告。

### Step 10 — Superpowers finishing

```bash
mvn test
# Results: 91 tests passed (86 existing + 5 new)

git status
# nothing to commit, working tree clean
```

→ 选项：merge PR → main

### Step 11 — gitcode-release-helper（如需发布新版本）

```bash
# 生成 release note
gc release create v1.2 -R aflyingto/openlibing-platform-release \
  --title "v1.2 - CVE 批量导入" \
  --notes-file RELEASE.md

# 上传资产
gc release upload v1.2 target/openlibing-platform-release.jar \
  -R aflyingto/openlibing-platform-release
```

---

## 全流程总结

```
需求阶段 (AI 提问 + 自动创建 Issue)  → ~10min
开发阶段 (TDD + 子代理并行)          → ~2h
交付阶段 (PR + 审查 + merge)         → ~15min

总计: AI 编码 ~2.5h，人工审核确认 ~10min
```

### 你做了什么事

| 阶段 | 你的操作 | 耗时 |
|------|---------|------|
| 需求 | 回答 brainstorming 的 4 个问题，确认 Issue 和需求分析 | ~5min |
| 开发 | 审查 TDD 生成的测试用例和关键设计决策 | ~3min |
| 交付 | 确认 PR 描述，查看代码审查结果，点击 merge | ~2min |

### AI 做了什么事

| 阶段 | AI 自动完成 |
|------|-----------|
| 需求 | 提问澄清 → 查重 → 提交 Issue → 代码搜索 → 需求分析 → 任务分解 |
| 开发 | 原子任务拆解 → TDD 循环 → 3 子代理并行执行 → 任务间审查 |
| 交付 | 检查变更 → PR 标题描述 → 6维度代码审查 → 行内评论发布 |
