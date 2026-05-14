---
name: gitcode-security-check
description: |
  对目标 GitCode 仓库进行代码级安全审查：扫描硬编码凭证、注入漏洞、不安全依赖、
  敏感数据泄露、权限绕过、加密缺陷等。生成安全审计报告含整改建议。

  TRIGGER when: 用户要求做代码安全检查、安全审计、漏洞扫描、检查代码安全、
  扫描硬编码密钥、审计依赖安全、检查敏感信息、security audit、code security、
  或对仓库/PR/代码做安全方面的深度检查。只要是审查目标仓库代码本身的安全，
  即应触发。
---

# GitCode 代码仓库安全审查

对目标仓库进行代码级安全扫描，发现潜在安全问题并给出整改建议。

## 审查维度

### 1. 硬编码凭证和密钥

扫描源码中是否硬编码了敏感凭证：

```bash
# 搜索硬编码 token/key/password
grep -rnI "token\s*[:=]\s*['\"]\w" --include="*.go" --include="*.py" --include="*.js" --include="*.ts" --include="*.java" --include="*.yaml" --include="*.json" --include="*.toml" . 2>/dev/null

# 搜索密码赋值
grep -rnI "password\s*[:=]\s*['\"]" --include="*.go" --include="*.py" --include="*.js" --include="*.ts" . 2>/dev/null

# 搜索 API key
grep -rnI "api[_-]?key\s*[:=]\s*['\"]" --include="*.go" --include="*.py" --include="*.yaml" . 2>/dev/null

# 搜索私钥（PEM/RSA）
grep -rnI "\-\-\-\-\-BEGIN (RSA |EC )?PRIVATE KEY" . 2>/dev/null

# 搜索敏感文件未在 .gitignore
git ls-files | grep -iE "\.env$|\.pem$|\.key$|credentials\.json$"
```

### 2. 注入漏洞

SQL/命令/模板注入模式检测：

```bash
# SQL 字符串拼接（Go 示例）
grep -rnI "Sprintf.*SELECT\|Sprintf.*INSERT\|Sprintf.*UPDATE\|Sprintf.*DELETE" --include="*.go" . 2>/dev/null

# 命令拼接执行
grep -rnI "exec\.Command\|os\.Exec\|subprocess\.\|popen" --include="*.go" --include="*.py" . 2>/dev/null | grep -v "_test"

# 动态拼接含用户输入的参数
grep -rnI "fmt\.Sprintf.*%s.*cmd\|Sprintf.*%s.*exec\|\"\+\s*.*\s*\"" --include="*.go" . 2>/dev/null
```

### 3. 认证授权缺陷

```bash
# 搜索 auth 相关函数检查有无绕过
grep -rnI "func.*auth\|func.*login\|func.*permit\|func.*access" --include="*.go" . 2>/dev/null

# 搜索 token/session 验证缺失信号
grep -rnI "TODO.*auth\|FIXME.*auth\|skip.*auth\|bypass\|disable.*auth" --include="*.go" --include="*.py" . 2>/dev/null

# 搜索权限检查函数
grep -rnI "func.*can\|func.*allowed\|func.*hasPerm" --include="*.go" . 2>/dev/null
```

### 4. 敏感信息泄露

内部地址、调试信息、详细错误暴露：

```bash
# 内网地址暴露
grep -rnI "192\.168\.\|10\.\d{1,3}\.\|172\.(1[6-9]\|2\d\|3[01])\." --include="*.go" --include="*.py" --include="*.yaml" . 2>/dev/null

# 调试打印泄漏数据
grep -rnI "fmt\.Print.*password\|log\.Print.*token\|fmt\.Print.*secret\|console\.log.*password" --include="*.go" --include="*.js" . 2>/dev/null

# 详细错误信息返回到客户端
grep -rnI "return.*err\.Error()\|w\.Write.*err.Error\|\.Error()\s*\)" --include="*.go" . 2>/dev/null
```

### 5. 不安全依赖

```bash
# Go：检查已知漏洞
grep -rnI "replace\s" go.mod 2>/dev/null  # 是否用了本地/私有替换（绕过审计）

# 检查依赖版本是否过期（go list -u -m all）
cat go.mod 2>/dev/null && echo "---依赖列表---" && grep "^\s" go.mod 2>/dev/null | head -20

# Python：检查 requirements
cat requirements.txt 2>/dev/null && echo "---" && grep -i "==" requirements.txt 2>/dev/null
```

### 6. 不安全加密

```bash
# 弱哈希（MD5/SHA1）
grep -rnI "md5\.Sum\|md5\.New\|sha1\.Sum\|sha1\.New\|MD5\|SHA1" --include="*.go" --include="*.py" --include="*.js" . 2>/dev/null

# 弱加密算法（DES/RC4）
grep -rnI "crypto/des\|crypto/rc4\|des\.\|rc4" --include="*.go" . 2>/dev/null

# 硬编码盐/IV
grep -rnI "salt\s*[:=]\s*['\"]|iv\s*[:=]\s*['\"]" --include="*.go" --include="*.py" . 2>/dev/null
```

### 7. 路径遍历和不安全文件操作

```bash
# 路径拼接含用户输入
grep -rnI "path\.Join.*\.\.\|filepath\.Join.*\.\.\|os\.Open.*\+" --include="*.go" . 2>/dev/null

# 文件权限过于宽松
grep -rnI "os\.Mkdir\|os\.Create\|os\.OpenFile.*0777\|0o777\|os.ModePerm" --include="*.go" . 2>/dev/null
```

### 8. 输入校验缺失

```bash
# 检查输入校验函数存在但可能被绕过的地方
grep -rnI "strconv\.Atoi\|strconv\.Parse\w*\(" --include="*.go" . 2>/dev/null | grep -v "_test" | grep -v "if err"

# 类型断言无 ok 检查（Go panic 风险）
grep -rnI "\.\([a-zA-Z]+\*?\)$" --include="*.go" . 2>/dev/null | grep -v "_test" | grep -v ", ok"
```

## 工作流

### 第一步：确定审查范围

从用户输入明确审查目标：
- 仓库（owner/repo）
- 具体目录或文件（如 `api/`、`pkg/cmd/`）
- 特定关注点（如"重点看 SQL 注入"）

如需 clone 仓库：
```bash
gc repo clone <owner/repo>
```

### 第二步：执行扫描

按上述 8 个维度逐项扫描。根据仓库使用的语言调整 grep 模式：
- Go：`--include="*.go"`
- Python：`--include="*.py"`
- JavaScript/TypeScript：`--include="*.js" --include="*.ts"`
- Java：`--include="*.java"`

### 第三步：分类和评估

对每个发现进行分类和风险评估：

| 严重度 | 定义 | 示例 |
|--------|------|------|
| **Critical** | 可直接利用、造成重大损失 | 硬编码生产 token、SQL 注入 |
| **High** | 高概率、中影响或中概率、高影响 | 认证绕过、敏感信息暴露 |
| **Medium** | 有条件利用 | 不安全加密、错误信息泄露 |
| **Low** | 最佳实践问题 | 调试日志残留、弱文件权限 |
| **Info** | 信息性建议 | 依赖版本建议、代码风格 |

### 第四步：生成报告

```markdown
## Security Audit: <repo_name>

### 审查范围
- 仓库：<owner/repo>
- 路径：<dir>
- 时间：<timestamp>
- 语言：<languages>

### 总体评估
- 安全等级：[A/B/C/D/F]
- 总发现问题：<n>
- Critical: <n> / High: <n> / Medium: <n> / Low: <n> / Info: <n>

### 发现详情

#### #1 [Critical] <标题>
- **文件**: `path/to/file.go:42`
- **问题**: <具体描述>
- **代码**:
  ```go
  // 问题代码片段
  ```
- **影响**: <可能的攻击场景>
- **修复**: <具体修复方案>
- **参考**: <CWE/OWASP 链接如相关>

#### #2 [High] ...

### 统计

| 维度 | 发现数 | Critical/High |
|------|--------|--------------|
| 硬编码凭证 | 2 | 2/0 |
| 注入漏洞 | 1 | 1/0 |
| 认证授权 | 0 | 0/0 |
| 敏感信息 | 1 | 0/1 |
| ... | ... | ... |

### 整改优先级

1. [Critical] <标题> — 立即修复
2. [High] <标题> — 本周内修复
3. [Medium] <标题> — 下个迭代
4. ...
```

## 示例用法

```
扫描 gitcode-cli/cli 仓库的 api/ 目录有没有硬编码 token 或 SQL 注入
```
```
对当前项目做完整安全审计
```
```
检查 go.mod 依赖有没有已知漏洞
```
```
审查 PR #42 的代码变更有没有安全问题
```

## 注意事项

- 安全报告中的敏感值**必须脱敏**（显示前几位或打码）
- 报告不对外公开（仅给用户）
- 发现硬编码凭证后优先建议：环境变量 → vault → 配置文件（.gitignore 保护）
- 不是所有 grep 命中都是真实问题，需要人工判断上下文
- 对误报（false positive）标注为 Info 并说明原因
