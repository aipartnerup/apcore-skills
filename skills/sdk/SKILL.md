---
name: sdk
description: >
  Bootstrap a new language SDK for the apcore ecosystem. Scaffolds the project
  structure with source stubs, test stubs (TDD red phase), and runnable examples
  ported from the reference implementation. Extracts the full API contract from
  the protocol spec and configures code-forge for implementation planning.
---

# Apcore Skills — SDK

Bootstrap a new apcore project in a new language. The project type is auto-discovered from the reference implementation — no hardcoded type list.

## Iron Law

**EVERY NEW PROJECT MUST IMPLEMENT THE FULL API CONTRACT. No partial implementations — if you ship it, it must cover all exported symbols from the reference implementation.**

## Anti-Rationalization Table

| Thought | Reality |
|---------|---------|
| "I'll start with just the core classes" | Start with a complete project skeleton. Feature implementation order is code-forge's job. |
| "Copy the Python structure exactly" | Use idiomatic target-language patterns. Same concepts, different structure. |
| "Tests can come later" | TDD is mandatory. Test infrastructure is set up in scaffolding. |
| "I'll figure out the naming as I go" | Naming is defined by conventions.md. Apply language rules from day one. |
| "Examples can be added after the API works" | Examples are ported from the reference implementation during scaffolding. Users need runnable code from day one. |

## When to Use

- Starting any new apcore project in a new language (any `--type` — auto-discovered from reference)
- Re-scaffolding an existing project that needs restructuring

## Command Format

```
/apcore-skills:sdk <language> [--type <type>] [--ref <repo>]
```

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `<language>` | Yes | — | Target language: `go`, `rust`, `java`, `csharp`, `kotlin`, `swift`, `php` |
| `--type` | No | `core` | Project type (e.g., `core`, `mcp`, `a2a`, `toolkit`). Any string — new types are auto-supported via reference discovery. |
| `--ref` | No | auto-detect | Reference implementation to extract API from |

**Example:**
```
/apcore-skills:sdk go --type mcp
→ Discovers apcore-mcp-python as reference
→ Extracts API contract (server/, auth/, adapters/, converters/, cli, explorer)
→ Scaffolds apcore-mcp-go/ with Go source stubs, test stubs, and ported examples
```

## Context Management

Steps 2 and 4 use sub-agents. Step 2 analyzes the reference implementation. Step 4 generates the project skeleton. The main context orchestrates and retains only summaries.

## Workflow

```
Step 0 (ecosystem) → 1 (parse args) → 2 (extract API contract) → 3 (tech stack) → 4 (scaffold) → 5 (feature specs) → 6 (plan generation) → 7 (summary)
```

## Bundled References

- `references/extract-api-contract.md` — Sub-agent prompt template for Step 2
- `references/scaffold-project.md` — Sub-agent prompt template for Step 4

## Detailed Steps

### Step 0: Ecosystem Discovery

@../shared/ecosystem.md

**Data flow:** Step 0 produces the following variables used by subsequent steps:
- `ecosystem_root` — absolute path to the parent directory containing all apcore repos
- `protocol_path` — path to the `apcore` protocol spec repo (e.g., `{ecosystem_root}/apcore/`)
- `repos[]` — list of discovered repos with metadata
- `config` — merged configuration object

---

### Step 1: Parse Arguments

Parse `$ARGUMENTS`:

1. Extract `<language>` — required, ask the user if missing
2. Extract `--type` — default `core`
3. Extract `--ref` — resolve reference repo (priority order):
   - If `--ref` explicitly specified: use that
   - **If CWD is a same-type apcore repo** (e.g., in `apcore-python/` and `--type core`): use CWD repo as reference
   - Otherwise auto-detect: look for `apcore-{type}-python` in ecosystem root (for `core` type, look for `apcore-python`)

Derive target repo name using the naming pattern:
- `core` type → `apcore-{lang}` (special case: no type infix)
- All other types → `apcore-{type}-{lang}` (e.g., `apcore-mcp-go`, `apcore-a2a-rust`, `apcore-toolkit-java`)

Derive target path: `{ecosystem_root}/{target-repo-name}/`

**Data flow:** Step 1 adds: `lang`, `type`, `ref_path`, `target-repo-name`, `target-path`

Check if target directory already exists:
- If exists with source files: warn and ask — "Update scaffolding" / "Use as-is" / "Cancel"
- If exists but empty: proceed

Display:
```
SDK Bootstrap:
  Language:   {lang}
  Type:       {type}
  Reference:  {ref-repo} ({ref-lang})
  Target:     {target-path}
```

---

### Step 2: Extract API Contract (Sub-agent)

Spawn `Agent(subagent_type="general-purpose")` with the prompt from `references/extract-api-contract.md`, filling in: `{lang}`, `{ref_path}`, `{type}`, `{protocol_path}`.

Store result as `api_contract`. If the sub-agent returns STATUS: NOT_FOUND or NO_EXPORTS, display error and ask the user to either provide a different reference or abort.

---

### Step 3: Confirm Tech Stack

Ask the user to confirm the target language tech stack.

@../shared/conventions.md (refer to "Testing Conventions" and "Dependency Conventions" sections)

**For Go:**
- Go version: "1.21+ (Recommended)" / "1.22+"
- Module path: default `github.com/aipartnerup/{target-repo-name}`
- Test extras: "Standard testing (Recommended)" / "testify"
- Schema validation: "go-jsonschema (Recommended)" / "gojsonschema" / "Other"

**For Rust:**
- Rust edition: "2021 (Recommended)" / "2024"
- Async runtime: "tokio (Recommended)" / "async-std" / "None (sync only)"
- Serialization: "serde (Recommended)" / "Other"
- Schema: "schemars (Recommended)" / "Other"

**For Java:**
- Java version: "17+ (Recommended)" / "21+"
- Build tool: "Gradle (Recommended)" / "Maven"
- Schema validation: "Jackson (Recommended)" / "Gson"
- Test framework: "JUnit 5 (Recommended)" / "TestNG"

**For other languages:** Single open-ended question about tech stack preferences.

Store `tech_stack` decisions.

---

### Step 4: Scaffold Project (Sub-agent)

Spawn `Agent(subagent_type="general-purpose")` with the prompt from `references/scaffold-project.md`, filling in: `{target-repo-name}`, `{target-path}`, `{lang}`, `{type}`, `{tech_stack}`, `{package_name}`, `{api_contract}`, `{ref_path}`, `{conventions_path}` (= resolved path to `../shared/conventions.md`).

After sub-agent completes, verify:
- Build config file exists
- Main module file exists with exports
- At least 3 source files exist
- Tests directory exists with test stubs
- Test helper/fixture file exists
- Examples directory exists (only if reference had examples/)
- README.md exists

---

### Step 5: Generate Feature Specs

Check if feature specs already exist at `{protocol_path}/docs/features/*.md`.

If they exist:
- Link to them via `.code-forge.json` configuration
- Display: `Feature specs found: {N} specs in {protocol_path}/docs/features/`

If they don't exist:
- Extract module list from the API contract
- Generate lightweight feature specs at `{target-path}/docs/features/`:
  - One per module (executor, registry, schema, etc.)
  - Each spec contains: module purpose, public API surface, acceptance criteria
- Display: `Feature specs generated: {N} specs in docs/features/`

---

### Step 6: Generate .code-forge.json and Trigger Planning

Write `{target-path}/.code-forge.json`:
```json
{
  "directories": {
    "base": "./",
    "input": "{relative-path-to-feature-specs}",
    "output": "planning/"
  },
  "port": {
    "source_docs": "{relative-path-to-protocol}",
    "reference_impl": "{relative-path-to-ref}",
    "target_lang": "{lang}"
  },
  "execution": {
    "default_mode": "ask",
    "auto_tdd": true,
    "task_granularity": "medium"
  }
}
```

Optionally add `reference_docs.sources` if the reference repo has `planning/` output. After writing, resolve each relative path (`input`, `reference_impl`, `source_docs`) and warn if any don't exist yet. Missing paths are acceptable (they may be created later) but should be noted.

**Git initialization is left to the user.** Display:
```
Project scaffolded. To initialize git:
  cd {target-path}
  git init
  git add <files...>
  git commit -m "chore: initialize {target-repo-name} project skeleton"
```

---

### Step 7: Display Summary and Next Steps

```
apcore-skills:sdk — SDK Bootstrap Complete

Target: {target-path}
Language: {lang}
Type: {type}
Modules: {N} source files scaffolded
Tests: {N} test stubs (TDD red phase)
Examples: {N} runnable examples
Feature specs: {N} specs available
API contract: {N} public symbols to implement

Project structure:
  {tree output of key files}

Next steps:
  cd {target-path}
  /code-forge:port @{protocol-path} --ref {ref-repo} --lang {lang}    Generate implementation plans
  /code-forge:impl {first-feature}                                      Start implementing
  /apcore-skills:sync --lang {lang},{ref-lang}                           Verify API consistency
```

## Coordination with Other Skills

- **After sdk:** Use `code-forge:port` to generate implementation plans for each feature
- **During implementation:** Use `code-forge:impl` to execute TDD tasks
- **After implementation:** Use `apcore-skills:sync` to verify cross-language consistency
- **Before release:** Use `apcore-skills:audit` for comprehensive check
- **For release:** Use `apcore-skills:release` for coordinated version bump
