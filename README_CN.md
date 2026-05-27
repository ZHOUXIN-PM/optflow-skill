# optflow

[English](README.md) | 中文

一个 Claude Code skill，用于发现代码库中的优化机会并端到端执行修复 —— 结构化优先级排序、行为保护、逐步验证。

## 它做什么

1. **发现** — 双层扫描：自动化工具（clippy、tsc、ruff、swiftlint 等）+ AI 代码分析（错误吞没、SQL 注入、N+1 查询、过高复杂度等模式）
2. **排序** — 结构化 Finding Contract，含严重性、置信度、适用性。优先级公式：`impact × confidence × applicability / (effort × regression_risk)`
3. **执行** — 逐步实现，每步必须通过行为守卫检查、运行测试、原子提交
4. **验证** — 每步测试矩阵。需求不明确时自动启用 BDD 场景

## 扫描类别

| 类别 | 示例 |
|------|------|
| 性能 | N+1 查询、异步中的同步 I/O、无界集合增长 |
| 可靠性 | 吞没的错误、缺失错误上下文、INSERT OR IGNORE 没有 UPSERT |
| 安全 | 字符串拼接 SQL、未过滤的输入、硬编码密钥 |
| 可维护性 | 圈复杂度 >10、函数 >80 行、重复代码块 |
| 开发者体验 | 缺失类型导出、未文档化的公共 API、过期 TODO |
| 成本 | 不必要的分配、冗余 API 调用 |

## 语言支持（Layer 1 工具扫描）

| 语言 | 工具 |
|------|------|
| Rust | `cargo clippy`、`cargo audit`、`cargo deny` |
| TypeScript | `tsc --noEmit`、`biome check` / `eslint` |
| Swift | `swiftlint`、`xcodebuild analyze` |
| Objective-C/C | `scan-build` / `clang-tidy` |
| Python | `ruff check`、`pip-audit` |

工具未安装会自动跳过，不会阻塞流程。

## 安装

### 方式一：symlink（推荐）

```bash
git clone https://github.com/ZHOUXIN-PM/optflow-skill.git ~/optflow-skill
ln -s ~/optflow-skill ~/.claude/skills/optflow
```

### 方式二：手动复制

```bash
mkdir -p ~/.claude/skills/optflow/references
cp SKILL.md ~/.claude/skills/optflow/
cp references/*.md ~/.claude/skills/optflow/references/
```

安装后在 `~/.claude/settings.json` 的 `skills` 里添加路径。

## 使用

### 在 Claude Code 中

```
/optflow
```

或用自然语言：

- "有什么可以优化的"
- "找优化点"
- "扫描代码"
- "代码有什么能改"
- "find optimization opportunities"
- "what can be improved"

### 提交策略

| 策略 | 行为 | 适用场景 |
|------|------|----------|
| `per_step`（默认） | 每步测试通过后立即提交 | 大多数场景 |
| `final_only` | 全部验证通过后单次提交 | 小批量修复 |
| `milestone` | 在里程碑边界提交 | 大型重构 |

### BDD 模式

说"用 BDD"或在需求不明确时自动启用。场景遵循 Given/When/Then 格式。

## 核心设计

**双层发现**：工具输出是事实（高置信度），AI 独立分析是假设（中/低置信度），直到被工具验证。防止误报噪声。

**Finding Contract**：每个发现都有结构化 YAML schema（id、category、source、confidence、applicability、behavior_risk），可以对接外部审计工具或看板。

**行为保护守卫**：每次代码变更前必须回答 4 个问题：这段代码为什么存在？必须保留什么行为？是否为平台特定的 workaround？能否验证变更没破坏行为？高危操作（删除版本特定代码、改 I/O 路径错误处理、修改 unsafe 块）需要用户确认。

**不做机会性重构**：只修复发现的问题，不额外"顺手"改东西。

## 文件结构

```
optflow-skill/
├── SKILL.md                      # 主 skill 定义
├── references/
│   ├── delivery-base.md          # 共享交付流程（DoR、计划、执行、验证、提交）
│   └── bdd-guide.md              # BDD Lite 参考（Given/When/Then、Scenario Outline）
├── README.md                     # English
├── README_CN.md                  # 中文
└── LICENSE
```

## 扩展

**添加语言**：在 `SKILL.md` 的 "Layer 1 — Automated tool scan" 表格中添加语言、命令和检测内容。

**添加模式**：在 Agent Analysis 部分按类别（Reliability、Security 等）添加新的代码模式。

**自定义 Finding Contract 字段**：YAML schema 是开放的，可以添加 `cwe_id`、`owasp_category`、`auto_fixable` 等字段。

## 许可证

[MIT](LICENSE)
