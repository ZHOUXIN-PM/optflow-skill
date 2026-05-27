---
name: optflow
description: "Discover and deliver repository optimization work end to end: identify performance/reliability/maintainability/security/dx/cost optimization points, prioritize by impact-effort-risk, then execute fixes step by step with continuous testing and explicit commit policy. Default to per_step."
user-invocable: true
triggers:
  - optflow
  - optimization discovery
  - find optimization opportunities
  - what can be improved
  - 有什么可以优化的
  - 找优化点
  - 扫描代码
  - 代码扫描
  - 全面扫描
  - 代码有什么能改
---

# Optflow

## Overview

Use this skill to discover repository optimization opportunities and execute selected optimizations end to end:

- Discover optimization points first (performance, reliability, maintainability, security, cost, DX).
- Prioritize by impact/effort/risk.
- Execute in strict sequence with validation and explicit commit policy.
- Add BDD behavior specs when requested or when requirements are ambiguous.

## Trigger Cues

Trigger this skill when the user asks for one or more of:

- Ask to find optimization opportunities in a repository/library.
- Ask for optimization roadmap + implementation.
- Require test-first optimization delivery and commit (final or per step).
- Require behavior-driven delivery (BDD) or acceptance scenarios.

## Workflow

### 0. Discover Optimization Backlog

Scan the repository before planning changes. Use **two layers**:

**Layer 1 — Automated tool scan (run first, results are facts):**

Detect project language from manifest files, then run the matching checklist:

| Language | Commands | What they catch |
|----------|----------|-----------------|
| Rust | `cargo clippy -- -W clippy::all 2>&1`, `cargo audit 2>&1`, `cargo deny check 2>&1` | lint warnings, known CVEs, license/supply-chain |
| TypeScript | `npx tsc --noEmit 2>&1`, `npx biome check . 2>&1` or `npx eslint . 2>&1` | type errors, style/logic lint |
| Swift | `swiftlint lint --quiet 2>&1`, `xcodebuild analyze 2>&1` (if .xcodeproj exists) | style, static analysis |
| Objective-C/C | `scan-build make 2>&1` or `clang-tidy 2>&1` (if available) | memory, null deref, logic |
| Python | `ruff check . 2>&1`, `pip-audit 2>&1` | lint, known CVEs |

If a tool is not installed, skip it and note "skipped: {tool} not found" — never block on missing tools.

**Layer 2 — Agent analysis (read code, apply judgment):**

After tool scan, read source files and look for these specific patterns:

- **Reliability**: `let _ =` (Rust error swallowing), empty `catch {}`, `INSERT OR IGNORE` without UPSERT, missing `.context()` on `?`
- **Security**: string-interpolated SQL, unsanitized IPC input, hardcoded secrets, `unsafe` blocks without justification
- **Maintainability**: cyclomatic complexity >10, nesting >4 levels, functions >80 lines, duplicated logic blocks >10 lines
- **Performance**: N+1 query patterns, synchronous I/O in async context, unbounded collection growth
- **DX**: missing type exports, undocumented public API, stale TODO/FIXME older than 6 months

Classify findings into: performance, reliability, maintainability, security, developer experience, and cost.

For each finding, record using the **Finding Contract** format:

```yaml
- id: "optflow-{category}-{seq}"
  category: <performance|reliability|maintainability|security|dx|cost>
  source: <tool name or "agent-analysis">
  file: <path from repo root>
  line_start: <number>
  line_end: <number>
  severity: <critical|high|medium|low>
  confidence: <high|medium|low>
  title: <one-line summary>
  evidence: <what was observed>
  impact: <what breaks or degrades>
  suggested_fix: <concrete action>
  applicability: <applicable|needs-review|not-applicable>
  behavior_risk: <none|low|high>
```

**Post-scan processing (mandatory before ranking):**

1. **Dedup**: if multiple findings point to the same root cause, merge into one finding
2. **Applicability check**: mark `not-applicable` for findings that don't match the actual framework/language/platform
3. **Confidence correction**: tool-reported findings = `high` confidence; agent-only findings without tool evidence = `medium` or `low`

Build a prioritized backlog using:

```
priority = impact × confidence × applicability / (effort × regression_risk)
```

Explicitly mark low-confidence findings as hypotheses.

### 1. Define Ready Criteria (DoR)

> See shared delivery base: `references/delivery-base.md`

Additional for optflow:

- If implementation is requested and commit style is not explicitly specified, use `per_step` (default).
- If user says "per step optimize and test then commit", use `per_step`.

### 2. Build Complete Plan Before Editing

> See shared delivery base: `references/delivery-base.md`

### 3. Add BDD Layer When Needed

Use BDD if user requests it, or if requirements are unclear.

For BDD Lite, Scenario Quality Checklist, Scenario Outline, and Test Layer mapping, see the shared reference:

> `references/bdd-guide.md`

### 4. Execute Step by Step

> See shared delivery base: `references/delivery-base.md`

**Pre-step behavior guard (mandatory before every code change):**

Before modifying or deleting any code, answer these 4 questions. If any answer is unclear, write a characterization test first.

1. **Why does this code exist?** — Read git blame, comments, related tests. Do not assume "it looks wrong" means "it is wrong".
2. **What behavior must be preserved?** — Identify observable side effects, error paths, platform-specific handling.
3. **Is this a platform workaround?** — iOS jailbreak paths, TrollStore entitlements, Tauri IPC boundaries, Filza/WebDAV quirks, rootless adaptations — these must NOT be "simplified" or "cleaned up" unless proven obsolete for all supported devices.
4. **Can we verify the change didn't break behavior?** — If no existing test covers it, write one before changing.

**Red flags that block immediate execution (require user confirmation):**
- Removing code that references specific iOS versions, device models, or jailbreak tools
- Changing error handling in device I/O paths (SSH, WebDAV, USB)
- Modifying `unsafe` blocks in Rust or swizzled methods in Objective-C
- Deleting code older than 1 year with no test coverage

- Treat one planned step as one feature boundary whenever possible.
- If `commit_policy = per_step` (default for implementation):
  - Stage only files for current step/feature.
  - Run step-level checks first.
  - Commit immediately after step checks pass.
  - Record step -> commit hash mapping.

### 5. Apply No-Backward-Compatibility Mode (When Requested)

> See shared delivery base: `references/delivery-base.md`

### 6. Validate with Test Matrix

> See shared delivery base: `references/delivery-base.md`

Per-step mandatory loop (when `commit_policy = per_step`):

1. Implement current feature step.
2. Run mapped step checks (at least one automated command).
3. If checks pass, commit this step immediately.
4. Move to next feature step.

### 7. Commit and Handoff

> See shared delivery base: `references/delivery-base.md`

- `per_step`: each step must already be committed before next step starts.

## Output Templates

### Optimization Backlog Template

```
Finding:
Category: <performance|reliability|maintainability|security|dx|cost>
Evidence: <file/metric/log>
Impact: <high|medium|low>
Effort: <high|medium|low>
Risk: <high|medium|low>
Priority score: <...>
Decision: <implement now|defer>
```

### Optimization Plan Template

```
Selected Findings:
1. <finding>
2. <finding>

Execution Steps:
1. <step> (done condition: <...>)
2. <step> (done condition: <...>)

Expected Gains:
- <metric or qualitative gain>
```

## Guardrails

### Execution discipline
- Do not stop at planning when implementation is expected.
- Do not leave partially completed plan steps.
- Do not defer required testing when it can be run now.
- Do not move to the next plan step before committing when `commit_policy = per_step`.
- Do not merge multiple completed feature steps into one commit when `commit_policy = per_step`.
- Do not claim compatibility if user explicitly requested no compatibility work.
- Do not include unrelated pre-existing dirty files in commits.
- Do not hand off without concrete validation evidence.

### Behavior preservation
- Do not delete or simplify code without first understanding why it exists (git blame + tests + comments).
- Do not remove platform-specific workarounds (jailbreak, TrollStore, Tauri IPC, WebDAV) unless proven obsolete for ALL supported devices.
- Do not make "opportunistic refactors" beyond the scope of the current finding.
- Do not change error handling behavior — propagate errors, don't swallow or reshape them without justification.
- If existing tests must be modified for the fix to pass, the fix is changing behavior — stop and confirm with user.

### Discovery quality
- Do not rank findings without running Layer 1 tool scan first (when tools are available).
- Do not treat agent-only findings as high confidence — they are hypotheses until tool-verified.
- Do not execute fixes for `not-applicable` or `needs-review` findings without user approval.
- Do not propose performance optimizations without a measurable baseline or benchmark command.
