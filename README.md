# optflow

A Claude Code skill that discovers optimization opportunities in any codebase and executes fixes end to end — with structured prioritization, behavior preservation, and per-step validation.

## What it does

1. **Discover** — Two-layer scan: automated tools (clippy, tsc, ruff, swiftlint, etc.) + agent code analysis for patterns like error swallowing, string-interpolated SQL, N+1 queries, excessive complexity.
2. **Prioritize** — Structured Finding Contract with severity, confidence, and applicability. Priority formula: `impact × confidence × applicability / (effort × regression_risk)`.
3. **Execute** — Step-by-step implementation with mandatory behavior guards, per-step testing, and atomic commits.
4. **Validate** — Test matrix per step. BDD scenarios when requirements are ambiguous.

### Categories

| Category | Examples |
|----------|----------|
| Performance | N+1 queries, sync I/O in async, unbounded collections |
| Reliability | Swallowed errors, missing error context, INSERT OR IGNORE without UPSERT |
| Security | String-interpolated SQL, unsanitized input, hardcoded secrets |
| Maintainability | Cyclomatic complexity >10, functions >80 lines, duplicated blocks |
| DX | Missing type exports, undocumented public API, stale TODOs |
| Cost | Unnecessary allocations, redundant API calls |

### Language support (Layer 1 tool scan)

| Language | Tools |
|----------|-------|
| Rust | `cargo clippy`, `cargo audit`, `cargo deny` |
| TypeScript | `tsc --noEmit`, `biome check` / `eslint` |
| Swift | `swiftlint`, `xcodebuild analyze` |
| Objective-C/C | `scan-build` / `clang-tidy` |
| Python | `ruff check`, `pip-audit` |

Missing tools are skipped gracefully — the skill never blocks on unavailable tooling.

## Install

### Claude Code (recommended)

Copy the skill directory into your Claude Code skills path:

```bash
# User-level install (available in all projects)
mkdir -p ~/.claude/skills/optflow/references
cp SKILL.md ~/.claude/skills/optflow/
cp references/*.md ~/.claude/skills/optflow/references/

# Then add to ~/.claude/settings.json under "skills"
```

Or symlink from wherever you cloned this repo:

```bash
git clone https://github.com/ZHOUXIN-PM/optflow-skill.git ~/optflow-skill
ln -s ~/optflow-skill ~/.claude/skills/optflow
```

### Verify installation

In Claude Code, type `/optflow` or say "find optimization opportunities" — the skill should activate.

## Usage

### Discover optimizations

```
/optflow
```

Or use natural language:

- "What can be improved in this repo?"
- "Find optimization opportunities"
- "Scan the code for issues"

### Trigger words

English: `optflow`, `optimization discovery`, `find optimization opportunities`, `what can be improved`

Chinese: `有什么可以优化的`, `找优化点`, `扫描代码`, `代码扫描`, `全面扫描`, `代码有什么能改`

### Commit policies

| Policy | Behavior | When to use |
|--------|----------|-------------|
| `per_step` (default) | Commit after each step passes tests | Most cases |
| `final_only` | Single commit after all validation | Small batch fixes |
| `milestone` | Commit at defined milestone boundaries | Large refactors |

### BDD mode

Say "use BDD" or the skill auto-activates BDD when requirements are ambiguous. Scenarios follow Given/When/Then format with a quality checklist.

## Key design decisions

**Two-layer discovery**: Tool output is fact (high confidence). Agent-only findings are hypotheses (medium/low confidence) until tool-verified. This prevents false-positive noise.

**Finding Contract**: Every finding follows a structured YAML schema with `id`, `category`, `source`, `confidence`, `applicability`, and `behavior_risk`. This makes findings portable — you can pipe them to external audit tools or dashboards.

**Behavior preservation guards**: Before every code change, the skill answers 4 questions: Why does this code exist? What behavior must be preserved? Is this a platform workaround? Can we verify the change? Red flags (removing version-specific code, changing error handling in I/O paths, modifying unsafe blocks) require explicit user confirmation.

**No opportunistic refactoring**: The skill fixes what it found and nothing more. Adjacent "cleanup" is how regressions happen.

## File structure

```
optflow-skill/
├── SKILL.md                      # Main skill definition
├── references/
│   ├── delivery-base.md          # Shared delivery workflow (DoR, plan, execute, validate, commit)
│   └── bdd-guide.md              # BDD Lite reference (Given/When/Then, Scenario Outline)
├── README.md
└── LICENSE
```

## Extending

**Add a language to Layer 1**: Edit the tool scan table in `SKILL.md` § "Layer 1 — Automated tool scan". Add the language, commands, and what they catch.

**Add patterns to Layer 2**: Edit the agent analysis section. Group by category (Reliability, Security, etc.).

**Custom Finding Contract fields**: The YAML schema is open — add fields like `cwe_id`, `owasp_category`, or `auto_fixable` for your workflow.

## License

[MIT](LICENSE)
