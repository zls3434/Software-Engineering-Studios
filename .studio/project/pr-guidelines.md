<!-- Software Engineering Studios -->
# PR/提交规范

本文件是平台无关的 PR 和提交规范源，定义提交信息格式、分支命名、代码审查流程和门禁检查要求。

## 1. Conventional Commits 格式

所有 Git 提交信息必须遵循 Conventional Commits 规范。

### 1.1 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 1.2 类型（type）

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): 添加 OAuth2 登录支持` |
| `fix` | 缺陷修复 | `fix(api): 修复分页参数 off-by-one 错误` |
| `docs` | 文档变更 | `docs(api): 更新用户接口文档` |
| `refactor` | 重构（不改变外部行为） | `refactor(auth): 提取令牌验证逻辑` |
| `test` | 测试相关 | `test(user): 添加用户注册单元测试` |
| `chore` | 构建/工具/杂项 | `chore(deps): 升级依赖至最新版本` |

### 1.3 规则

- **subject 使用简体中文**，不超过 50 字
- **subject 使用祈使语气**，首字母不大写，结尾不加句号
- **scope** 为可选项，表示影响的模块或领域
- **body** 说明变更原因和背景，每行不超过 72 字
- **footer** 标注 BREAKING CHANGE 或关联 issue（如 `Closes #123`）
- 提交前由 `validate-commit.sh` Hook 自动校验格式

### 1.4 示例

**标准提交：**

```
feat(auth): 添加 OAuth2 登录支持

集成 Google 和 GitHub OAuth2 提供商，
支持用户通过第三方账号登录。

Closes #42
```

**带 BREAKING CHANGE 的提交：**

```
feat(api): 重构用户接口返回结构

将用户信息从嵌套对象改为扁平结构，
提升前端消费便利性。

BREAKING CHANGE: /users 接口返回结构变更，
前端需同步更新数据解析逻辑
```

## 2. 分支命名

采用基于主干的开发模式。

| 分支类型 | 格式 | 用途 | 示例 |
|----------|------|------|------|
| 主干 | `main` | 生产分支，受保护 | `main` |
| 功能分支 | `feature/[简短描述]` | 新功能开发 | `feature/user-auth` |
| 修复分支 | `fix/[简短描述]` | 缺陷修复 | `fix/pagination-off-by-one` |
| 热修复分支 | `hotfix/[简短描述]` | 生产紧急修复 | `hotfix/login-crash` |
| 发布分支 | `release/vX.Y.Z` | 发布准备 | `release/v1.2.0` |

### 分支规则

- `main` 和 `release/*` 为受保护分支，禁止直接推送
- `validate-push.sh` Hook 拦截对受保护分支的强制推送
- 功能分支从 `main` 检出，开发完成后通过 PR 合并回 `main`
- 热修复分支从 `main` 检出，修复后同时合并到 `main` 和当前发布分支

## 3. 代码审查流程

### 3.1 审查要求

- 所有代码变更必须通过 Pull Request（或等效流程）合并
- PR 必须至少获得一名审查者的 approval 才能合并
- 审查由 `code-reviewer` Agent 或人工审查者执行
- 涉及架构变更的 PR 需 `chief-architect` 额外审批

### 3.2 审查内容

审查者应关注以下方面：

| 审查维度 | 检查项 |
|----------|--------|
| 功能正确性 | 代码是否实现了预期功能 |
| 编码规范 | 是否符合命名约定和禁止模式 |
| 测试覆盖 | 是否包含充分的单元测试和集成测试 |
| 安全性 | 是否存在安全风险（硬编码密钥、SQL 注入等） |
| 性能 | 是否有明显性能问题 |
| 可维护性 | 代码是否清晰、可读、可扩展 |
| 文档 | 公共 API 是否有完整文档注释 |

### 3.3 审查流程

1. **提交 PR**：开发者提交 PR，填写描述和关联 issue
2. **自动检查**：CI 流水线自动运行 lint、测试、安全扫描
3. **人工审查**：审查者审查代码，提出修改建议或 approval
4. **修改迭代**：开发者根据反馈修改代码
5. **最终审批**：审查者给出最终 approval
6. **合并**：通过所有门禁后合并到主干

## 4. 门禁检查要求

### 4.1 合并前必须通过的门禁

| 门禁 | 检查项 | 负责人 | 强制级别 |
|------|--------|--------|----------|
| 代码审查 | 至少一名审查者 approval | `code-reviewer` | BLOCKING |
| 代码静态检查 | Lint 无错误 | CI 自动 | BLOCKING |
| 单元测试 | 全部通过，覆盖率达标 | `qa-lead` | BLOCKING |
| 集成测试 | 关键路径全部通过 | `qa-lead` | BLOCKING |
| 安全扫描 | 无严重漏洞 | `security-lead` | BLOCKING |
| 构建验证 | 构建成功，产物完整 | `devops-lead` | BLOCKING |

### 4.2 CI 流水线阶段

```
提交 → Lint → 单元测试 → 集成测试 → 构建验证 → 安全扫描 → 合并
```

- 每次提交自动触发 CI 流水线
- 任一阶段失败禁止合并到主干
- CI 配置文件纳入版本控制

### 4.3 发布门禁

生产部署需通过额外的发布门禁：

- 运行 `/gate-check` 命令进行发布前检查
- `release-manager` 确认发布清单完整
- `security-lead` 确认安全审计通过
- `devops-lead` 确认部署策略（蓝绿部署或滚动部署）

## 5. 版本管理

### 5.1 语义化版本

采用语义化版本号：`MAJOR.MINOR.PATCH`

- **MAJOR**：不兼容的 API 修改
- **MINOR**：向下兼容的新功能
- **PATCH**：向下兼容的缺陷修复

### 5.2 发布标签

- 发布标签格式：`vX.Y.Z`（如 `v1.2.0`）
- 变更日志由 `/changelog` 自动生成
- 补丁说明由 `/patch-notes` 生成

## 6. 提交前检查清单

提交 PR 前开发者应确认：

- [ ] 提交信息符合 Conventional Commits 格式
- [ ] subject 不超过 50 字
- [ ] 分支命名符合规范
- [ ] 代码通过本地 lint 检查
- [ ] 本地测试全部通过
- [ ] 无硬编码密钥或敏感信息
- [ ] 无 `console.log` 或调试输出
- [ ] 新功能包含对应的测试用例
- [ ] 公共 API 包含文档注释
- [ ] PR 描述清晰，关联相关 issue