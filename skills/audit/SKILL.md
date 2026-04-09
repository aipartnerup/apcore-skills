---
name: audit
description: >
  Deep cross-repo consistency audit for the apcore ecosystem. Checks API surface
  alignment, naming conventions, version synchronization, documentation quality,
  test coverage, dependency alignment, and configuration consistency across all
  repos. Generates a detailed report with severity-classified findings.
---

# Apcore Skills — Audit

## ⚡ Execution Entry Point (READ THIS FIRST)

**When this skill is loaded, you MUST immediately begin executing the Workflow below — do not wait, do not summarize, do not ask "what should I do now". Skills are operational manuals, not reference documents.** Read Step 0 (Ecosystem Discovery), then Step 1 (Parse Arguments), then Step 2 (Execute Audit Dimensions), etc., until the workflow completes or you reach an `AskUserQuestion` checkpoint.

If the harness shows you `Successfully loaded skill · N tools allowed`, that message means **the SKILL.md content was injected into your context** — it does NOT mean the skill has run. Skills do not "run" autonomously; you run them by executing the Detailed Steps below.

If you find yourself about to say "the skill didn't produce output", "skill 仍未输出", "falling back to manual audit", "回退到手动 audit", or anything similar, **STOP**. You have misunderstood how skills work. Go directly to Step 0 and start executing.

The first user-visible action of this skill should be either (a) the output of Step 0 / Step 1, or (b) an `AskUserQuestion` if scope detection needs disambiguation. Never an apology, never a fallback, never silence.

---

Comprehensive consistency audit across all apcore ecosystem repositories.

## Iron Law

**AUDIT EVERY DIMENSION. CLASSIFY EVERY FINDING. A partial audit creates false confidence.**

## When to Use

- Before a major release to ensure ecosystem-wide consistency
- After adding a new SDK or integration to verify alignment
- Periodic health check (monthly recommended)
- When suspecting drift between implementations

## Command Format

```
/apcore-skills:audit [--scope core|mcp|integrations|all] [--fix] [--save report.md]
```

| Flag | Default | Description |
|------|---------|-------------|
| `--scope` | **cwd** | Which repo group to audit. **If omitted, defaults to the current working directory's repo only.** Use `--scope all` for full ecosystem audit. |
| `--fix` | off | Auto-fix issues where safe |
| `--save` | off | Save report to file |

## Audit Dimensions

The audit covers 9 dimensions, each checking specific aspects:

| # | Dimension | Severity Range | Description |
|---|---|---|---|
| D1 | API Surface | critical-warning | Public API alignment across languages |
| D2 | Naming Conventions | critical-warning | File/class/function naming per language rules |
| D3 | Version Sync | critical-info | Version alignment within sync groups |
| D4 | Documentation | warning-info | README, CHANGELOG, docstring completeness |
| D5 | Test Coverage | warning-info | Test file existence and coverage metrics |
| D6 | Dependencies | critical-warning | Dependency versions and compatibility |
| D7 | Configuration | warning-info | APCORE_* settings consistency across integrations |
| D8 | Project Structure | warning-info | File/directory layout per conventions |
| D9 | **Bloat & Redundancy** | **critical-info** | **Dead exports, duplicate symbols, parallel implementations, LOC growth, unused config, scope creep** |

## Severity Levels

| Level | Meaning | Action Required |
|---|---|---|
| `critical` | Breaking inconsistency — users will hit errors | Must fix before release |
| `warning` | Non-breaking inconsistency — confusing but functional | Should fix soon |
| `info` | Cosmetic or minor inconsistency | Nice to fix |

## Context Management

**All dimension audits and per-repo fixes are executed by parallel sub-agents.** The main context ONLY handles:
1. Orchestration — determining scope and spawning sub-agents
2. Aggregation — collecting structured findings from all sub-agents
3. Reporting — formatting and displaying the consolidated report

Step 2 spawns **up to 9 parallel sub-agents** (one per dimension, all simultaneously). Step 4 spawns **one parallel sub-agent per repo** for fixes. The main context never reads repo files directly.

## Workflow

```
Step 0 (ecosystem) → Step 1 (parse args) → Step 2 (parallel audits) → Step 3 (report) → [Step 4 (fix)]
```

## Detailed Steps

### Step 0: Ecosystem Discovery

@../shared/ecosystem.md

---

### Step 1: Parse Arguments and Plan Audit

Parse `$ARGUMENTS` for flags.

#### 1.1 CWD-based Default Scope

**If `--scope` is NOT specified:**
1. Detect CWD repo name (basename of CWD)
2. Look up in discovered ecosystem:
   - `core-sdk` repo → audit only this repo, dimensions D1-D3, D5-D6, D8-D9
   - `mcp-bridge` repo → audit only this repo, dimensions D1-D3, D5-D6, D8-D9
   - `integration` repo → audit only this repo, dimensions D2-D9
   - `protocol`/`docs-site` repo → audit documentation dimensions (D4) and bloat (D9) for this repo
   - `shared-lib`/`tooling` repo → audit D2 (naming), D4 (docs), D5 (tests), D8 (structure), D9 (bloat) for this repo
   - CWD not an apcore repo → use `AskUserQuestion` to ask: "CWD is not an apcore repo. Which repo do you want to audit?" with options from `repos[]` names + "All repos (full ecosystem audit)"
3. Display: "Scope: {repo-name} (from CWD). Use --scope all for full ecosystem audit."

**If `--scope` IS specified:** use explicit scope.

#### 1.2 Scope → Repos & Dimensions

| Scope | Repos | Dimensions |
|---|---|---|
| `core` | Core SDKs | D1-D3, D5-D6, D8-D9 |
| `mcp` | MCP bridges | D1-D3, D5-D6, D8-D9 |
| `integrations` | Framework integrations | D2-D9 (no cross-API sync) |
| `all` | All repos | All dimensions |

**D9 (Bloat & Redundancy) is always included.** It applies to every scope and every repo type — it is the apcore ecosystem's primary defense against the additive bias of skill-driven feature work.

Display:
```
Audit scope: {scope} {("(from CWD)" if defaulted)}
Repos: {count} repositories
Dimensions: {list}
```

---

### Step 2: Execute Audit Dimensions (Sub-agents)

Spawn **all dimension sub-agents in parallel (up to 8 simultaneously)**. Each sub-agent audits exactly 1 dimension. All dimensions are fully independent — no batching or serialization needed.

#### Sub-agent: D1 — API Surface Audit

**Prompt:**
```
Audit API surface consistency across apcore SDK implementations.

Repos to compare: {repo_paths}

For each repo:
1. Read the main export file (__init__.py / index.ts)
2. Count total exports
3. Categorize: classes, functions, types, enums, constants, errors
4. For each class: list constructor params and public method names

Compare across repos:
- Missing symbols (in one, not the other)
- Method count mismatches per class
- Constructor param count mismatches

Return findings in format:
Error handling:
- If a repo path does not exist, skip it and report as: severity=warning, detail="Repo not found at {path}"
- If main export file is missing, report as: severity=warning, detail="No exports found in {repo}"

DIMENSION: D1 — API Surface
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix}
```

#### Sub-agent: D2 — Naming Conventions Audit

**Prompt:**
```
Audit naming conventions across apcore ecosystem repos.

Repos: {repo_paths}

Check against conventions:
- Python: snake_case files, snake_case functions, PascalCase classes
- TypeScript: kebab-case files, camelCase functions, PascalCase classes
- Package names match convention (snake_case for Python, kebab-case for npm)
- Error class names end with "Error"
- Enum names are PascalCase, enum values are UPPER_SNAKE

Scan all source files in each repo:
1. Check filenames in src/ directory
2. Grep for class/function definitions and verify naming
3. Check import/export names in main module file
4. Verify error classes follow naming pattern

Error handling: If a repo path does not exist or has no src/ directory, report as:
- severity: info, detail: "Repo {name} has no source files to audit"

Return findings in this exact format:
DIMENSION: D2 — Naming Conventions
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix}
```

#### Sub-agent: D3 — Version Sync Audit

**Prompt:**
```
Audit version synchronization across apcore ecosystem.

Repos: {repo_paths with types}

Check:
1. Core SDK versions must match (major.minor): {core repos}
2. MCP bridge versions must match (major.minor): {mcp repos}
3. Within each repo, version must be consistent across all version files
   - Python: pyproject.toml version == __init__.py __version__
   - TypeScript: package.json version == src/index.ts VERSION (if present)
4. Integration repos: dependency versions reference compatible core SDK versions

Error handling: If a build config file is missing or malformed, report as a warning finding rather than failing.

Return findings in this exact format:
DIMENSION: D3 — Version Sync
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix}
```

#### Sub-agent: D4 — Documentation Audit

**Prompt:**
```
Audit documentation quality across apcore ecosystem.

Repos: {repo_paths}

For each repo check:
1. README.md exists and contains: description, installation, quick start, link to docs
2. CHANGELOG.md exists and follows Keep a Changelog format
3. LICENSE file exists
4. Key source files have docstrings/JSDoc
5. Public API methods have parameter documentation
6. Examples directory exists (for integrations)

Error handling: If a repo path does not exist, skip it and report as info finding.

Return findings in this exact format:
DIMENSION: D4 — Documentation
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file if applicable}
  fix: {suggested fix}
```

#### Sub-agent: D5 — Test Coverage Audit

**Prompt:**
```
Audit test coverage across apcore ecosystem.

Repos: {repo_paths}

For each repo:
1. Check tests/ directory exists
2. Count test files
3. Map source files to test files (check for coverage gaps)
4. Run test suite and capture results (detect language from build config, check runner availability first):
   - Python: `pytest --tb=short -q`
   - TypeScript: `npx vitest run`
   - Go: `go test -cover ./...`
   - Rust: `cargo test`
   - Java (Maven): `mvn test -q`
   - Java (Gradle): `gradle test`
   - C#: `dotnet test`
   - Swift: `swift test`
   - PHP: `vendor/bin/phpunit`
   - Elixir: `mix test`
   If the test runner is not available, report as a finding instead of running.
5. If coverage tool available, capture coverage percentage

Error handling:
- If dependencies are not installed (import errors, module not found), report as: severity=warning, detail="Test dependencies not installed for {repo}"
- If test runner is not found, report as: severity=warning, detail="Test runner not available for {repo}"
- Do NOT fail the entire audit if one repo's tests can't run

Return findings in this exact format:
DIMENSION: D5 — Test Coverage
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file if applicable}
  fix: {suggested fix}
TEST_RESULTS:
- repo: {repo-name}
  status: {pass|fail|skipped}
  total: {N}
  passed: {N}
  failed: {N}
  coverage: {pct or "unknown"}
```

#### Sub-agent: D6 — Dependencies Audit

**Prompt:**
```
Audit dependency alignment across apcore ecosystem.

Repos: {repo_paths}

Check:
1. Core SDK dependencies: schema validation lib versions match expectations
2. MCP bridge dependencies: MCP SDK versions are compatible
3. Integration dependencies: core SDK version references are up-to-date
4. No duplicate dependencies with different versions across repos
5. Dev dependencies include linter (ruff/eslint), type checker (mypy/tsc), test framework
6. No known vulnerability patterns in pinned versions

Error handling: If a build config file is missing, report as: severity=warning, detail="No build config found for {repo}"

Return findings in this exact format:
DIMENSION: D6 — Dependencies
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix}
```

#### Sub-agent: D7 — Configuration Audit (integrations only)

**Prompt:**
```
Audit APCORE_* configuration consistency across framework integrations.

Repos: {integration_repo_paths}

For each integration:
1. Read config.py/config.ts — search in src/{package}/ directory
2. Extract all APCORE_* settings with types and defaults
3. Compare settings across integrations — same settings should have same types and defaults

Required settings (must be present in all):
- APCORE_ENABLED, APCORE_DEBUG, APCORE_SCANNERS
- APCORE_INCLUDE_PATHS, APCORE_EXCLUDE_PATHS
- APCORE_MODULE_PREFIX, APCORE_AUTH_ENABLED
- APCORE_TRANSPORT, APCORE_HOST, APCORE_PORT

Error handling: If config file not found in a repo, report as: severity=warning, detail="Config file not found for {repo}"

Return findings in this exact format:
DIMENSION: D7 — Configuration
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix}
CONFIG_MATRIX:
- setting: {APCORE_SETTING}
  {repo-1}: {type}={default}
  {repo-2}: {type}={default}
  consistent: true|false

Target ~2-3KB. If more than 20 settings exist, only list inconsistent settings in the matrix and summarize consistent ones as a count.
```

#### Sub-agent: D8 — Project Structure Audit

**Prompt:**
```
Audit project structure consistency across apcore ecosystem.

Repos: {repo_paths}

Check against conventions:
- Core SDKs: has src/ with expected subdirectories (middleware/, registry/, schema/, observability/, utils/)
- MCP bridges: has src/ with expected subdirectories (server/, auth/, adapters/, converters/)
- Integrations: has src/ with expected structure (scanners/, output/, cli)
- All repos: has tests/, .gitignore, README.md, CHANGELOG.md, LICENSE, build config

Error handling: If a repo directory does not exist or is empty, report as: severity=info, detail="Repo {name} is empty or placeholder"

Return findings in this exact format:
DIMENSION: D8 — Project Structure
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  detail: {description}
  location: {directory or file}
  fix: {suggested fix}
```

#### Sub-agent: D9 — Bloat & Redundancy Audit

**Prompt:**
```
Audit code bloat and redundancy across apcore ecosystem repos.

Repos: {repo_paths}

This dimension is the primary defense against additive bias from skill-driven workflows
(spec-forge / code-forge / apcore-skills) that bias toward "add new" over "reuse existing".
Be thorough — bloat does not surface in any other audit dimension.

For each repo, perform ALL of the following checks:

1. DEAD EXPORTS — public symbols (classes, functions, constants, types) declared in the
   main export file (__init__.py / index.ts / lib.rs / mod.rs) that have zero callers
   anywhere in the ecosystem. Cross-check against ALL repos in scope when possible — a
   symbol exported by core-sdk but never imported by any integration or downstream repo
   is a strong dead-export signal.
   - severity: warning (zero internal + zero external callers)
   - severity: critical (entire submodule with no callers)

2. UNUSED INTERNAL SYMBOLS — non-exported functions, classes, helpers, or methods inside
   src/ that are defined but never referenced in the same repo. Use grep across the repo
   for each candidate. Skip test fixtures and __all__-listed symbols.
   - severity: warning per unused symbol
   - severity: critical if a whole file is dead

3. DUPLICATE / NEAR-DUPLICATE FUNCTIONS — within a repo, look for functions with
   structurally identical bodies (≥ 5 statements) that differ only in literals or naming.
   Across repos in the same language, look for utility functions that have been
   independently reimplemented (string utils, path utils, type guards, schema helpers).
   - severity: warning (intra-repo duplicates)
   - severity: critical (cross-repo duplicates of utility code that should live in shared-lib)

4. PARALLEL IMPLEMENTATIONS — two or more modules within a repo that implement the same
   concept slightly differently (e.g., two HTTP client wrappers, two config loaders, two
   logger adapters). This is the most common bloat pattern from feature-by-feature
   planning.
   - severity: critical — propose merging

5. STALE / COMMENTED-OUT CODE — large commented-out blocks (≥ 10 lines), files containing
   only TODOs older than the most recent release tag, modules marked deprecated with no
   removal date.
   - severity: info (small blocks)
   - severity: warning (large blocks or files)

6. UNUSED CONFIG / FLAGS — APCORE_* settings, CLI flags, environment variables, or
   .code-forge.json keys that are declared but never read by any code path. Grep for the
   key name across the repo (and across the ecosystem for cross-repo settings).
   - severity: warning

7. UNUSED DEPENDENCIES — packages declared in pyproject.toml / package.json / Cargo.toml
   that are never imported. Use the project's dependency analyzer if available, otherwise
   grep for import statements.
   - severity: warning
   - severity: critical if the unused dependency is large or has known CVEs

8. WRAPPER / PASSTHROUGH FUNCTIONS — functions whose entire body is a single call to
   another function with the same arguments, or that only rename fields. Often introduced
   "for future flexibility" but never extended.
   - severity: warning

9. SCOPE CREEP DETECTION — for each feature directory under planning/ (if present), read
   plan.md and check whether the actual implementation introduced files, modules, or
   public APIs not mentioned in the plan. Cross-reference state.json `tasks[].commits` if
   available.
   - severity: warning (extra files)
   - severity: critical (extra public API)

10. LOC GROWTH SIGNAL — compute total source LOC of the repo (excluding tests, vendor,
    generated code). If git history is available, compare against the LOC at the previous
    release tag. Surface as INFO finding regardless of magnitude — this is a trend signal,
    not a defect by itself.

11. NET-TO-USEFUL-CODE RATIO — for each src/ directory, compute the ratio of (lines that
    are reachable from a public export) to (total lines). If reachability cannot be
    determined statically, skip this check and note it.

Use Grep extensively. Read files at signature level when needed. Do NOT trust naming
alone — verify with actual references.

Error handling:
- If a repo path does not exist, report as: severity=info, detail="Repo not found at {path}"
- If git history is unavailable, skip LOC growth check and note it
- Do NOT fail the entire audit if reachability analysis fails for one repo

Return findings in this exact format:
DIMENSION: D9 — Bloat & Redundancy
FINDING_COUNT: {N}
FINDINGS:
- severity: {critical|warning|info}
  repo: {repo-name}
  category: {dead_export|unused_internal|duplicate|parallel_impl|stale|unused_config|unused_dep|wrapper|scope_creep|loc_growth|reachability}
  detail: {description}
  location: {file:line if applicable}
  fix: {suggested fix — deletion, merge, inline, etc.}
BLOAT_SUMMARY:
- repo: {repo-name}
  total_loc: {N}
  loc_at_last_release: {N or "unknown"}
  delta: {N or "unknown"}
  dead_exports: {N}
  unused_internal: {N}
  duplicates: {N}
  parallel_impls: {N}
  unused_config: {N}
  unused_deps: {N}
  scope_creep_files: {N}
```

---

### Step 3: Aggregate and Display Report

Collect all findings from sub-agents. Aggregate by severity.

```
apcore-skills audit — Ecosystem Consistency Report

Date: {date}
Scope: {scope}
Repos audited: {count}

═══ SUMMARY ═══

  Dimension              | Critical | Warning | Info
  D1 API Surface         |    2     |    3    |   1
  D2 Naming Conventions  |    0     |    5    |   3
  D3 Version Sync        |    1     |    0    |   0
  D4 Documentation       |    0     |    2    |   4
  D5 Test Coverage       |    0     |    1    |   2
  D6 Dependencies        |    1     |    2    |   0
  D7 Configuration       |    0     |    3    |   1
  D8 Project Structure   |    0     |    1    |   2
  D9 Bloat & Redundancy  |    1     |    8    |   5
  ─────────────────────────────────────────────────
  TOTAL                  |    5     |   25    |  18

═══ CRITICAL FINDINGS ═══

[D1-001] Missing API: Registry.scan_directory()
  Repo: apcore-typescript
  Detail: Present in apcore-python (src/apcore/registry/registry.py:45) but missing from TypeScript SDK
  Fix: Add scan_directory method to src/registry/registry.ts

[D3-001] Version mismatch in core sync group
  Repos: apcore-python=0.7.0, apcore-typescript=0.7.1
  Fix: Align versions before release

...

═══ WARNING FINDINGS ═══
(grouped by dimension)

═══ INFO FINDINGS ═══
(grouped by dimension)

═══ BLOAT REPORT (D9) ═══

  Repo                  | LOC    | Δ vs last | Dead | Dup | Parallel | Unused Cfg | Unused Dep | Scope Creep
  apcore-python         | 12450  | +2310     |  4   |  3  |    1     |     2      |     1      |      0
  apcore-typescript     | 11200  | +1980     |  6   |  2  |    0     |     1      |     0      |      2
  django-apcore         |  4500  |  +890     |  2   |  1  |    0     |     0      |     0      |      1
  flask-apcore          |  3800  |  +710     |  1   |  0  |    0     |     0      |     0      |      0
  ──────────────────────────────────────────────────────────────────────────────────────────────────────────
  TOTAL                 | 31950  | +5890     | 13   |  6  |    1     |     3      |     1      |      3

  Top bloat hotspots (act on these first):
    1. apcore-typescript: 6 dead exports — see [D9-002] through [D9-007]
    2. apcore-python: parallel HTTP client implementations — see [D9-014]
    3. django-apcore: scope creep in user-auth feature (+3 unplanned files)

═══ HEALTH SCORE ═══

  Overall: {score}/100
  API Consistency: {score}/100
  Naming: {score}/100
  Version Sync: {score}/100
  Documentation: {score}/100
  Test Coverage: {score}/100
  Dependencies: {score}/100
  Leanness (D9):     {score}/100
```

**Leanness score formula:** Start at 100. Subtract 5 per `critical` D9 finding, 2 per `warning`, 0.5 per `info`. Floor at 0. A leanness score below 70 indicates the repo needs a dedicated cleanup pass before the next release.

If `--save` flag: write full report to specified path.

---

### Step 4: Auto-Fix (only with --fix flag)

Group fixable findings by repo. Separate unfixable findings for reporting.

**Unfixable (skip and report):**
- API surface fixes (complex — delegate to `/apcore-skills:sync --phase a --fix`)
- Dependency fixes (risky — show as recommendations only)

**Fixable (per-repo parallel sub-agents):**

Spawn one `Task(subagent_type="general-purpose")` **per repo that has fixable findings, all in parallel**:

**Sub-agent prompt:**
```
Apply audit fixes for {repo_path}.

Fixable findings:
{all fixable findings for this repo from Step 3}

Fix rules (apply in order):
1. BLOAT FIXES (D9) — Apply BEFORE structural fixes so we don't add scaffolding around dead code:
   - dead_export → delete the symbol and any tests for it
   - unused_internal → delete the symbol
   - duplicate (intra-repo) → keep one, replace callers of the other, delete the duplicate
   - wrapper / passthrough → inline at the call site
   - stale (commented-out blocks ≥ 10 lines, ancient TODOs) → delete
   - unused_config → remove the config key from defaults and validation
   - unused_dep → remove from pyproject.toml / package.json / Cargo.toml and lockfile
   Skip (leave for human): parallel_impl (needs design decision), scope_creep (needs spec discussion), cross-repo duplicates (needs shared-lib coordination)
2. NAMING FIXES (D2) — Rename files/symbols to match conventions
3. VERSION FIXES (D3) — Update version strings for consistency
4. STRUCTURE FIXES (D8) — Create missing directories/files with stubs
5. DOC FIXES (D4) — Add missing README sections, fix CHANGELOG format

After all fixes:
- Run the full test suite (detect language: pytest | npx vitest run | go test ./... | cargo test | mvn test | gradle test | dotnet test | swift test | vendor/bin/phpunit | mix test)
- If tests fail: identify which fix caused it, revert ONLY that fix
- Leave all changes uncommitted

Error handling: If test runner is not available, skip verification and note it.

Return:
REPO: {repo-name}
FIXES_APPLIED: {count}
FIXES_REVERTED: {count}
TEST_RESULT: {pass|fail|skipped}
TEST_COUNTS: {passed}/{total}
CHANGES:
- dimension: {D2|D3|D4|D8|D9}
  file: {path}
  action: {what was changed}
LOC_REMOVED_BY_BLOAT_FIXES: {N}
```

Wait for all repo fix sub-agents to complete.

Display consolidated results:
```
Auto-fix applied:
  {repo-1}: {N} fixes (naming: 2, version: 1, structure: 0, docs: 1)
  {repo-2}: {N} fixes (naming: 0, version: 1, structure: 2, docs: 0)

Tests after fix:
  {repo-1}: {pass}/{total} passing ✓
  {repo-2}: {pass}/{total} passing ✓

Unfixed (manual action needed):
  [D1-001] API surface gap — use /apcore-skills:sync --phase a --fix
  [D4-xxx] Doc inconsistency — use /apcore-skills:sync --phase b for deep check
  [D6-002] Dependency version — manually update {package}

Review changes:
  cd {repo-1} && git diff
  cd {repo-2} && git diff
```
