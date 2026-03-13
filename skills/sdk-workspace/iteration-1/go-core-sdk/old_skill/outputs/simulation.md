# Simulation: /apcore-skills:sdk go --type core (OLD Skill)

Simulated execution of the OLD version of the SDK bootstrap skill.

**Command:** `/apcore-skills:sdk go --type core`
**Working Directory:** `/Users/tercel/WorkSpace/aipartnerup/`
**Date:** 2026-03-13

---

## Step 0: Ecosystem Discovery

### 0.1 Detect Ecosystem Root

Search strategy applied:
1. Check CWD for `.apcore-skills.json` -- not found.
2. Search for `apcore/` subdirectory containing `PROTOCOL_SPEC.md` -- **FOUND** at `/Users/tercel/WorkSpace/aipartnerup/apcore/PROTOCOL_SPEC.md`.

**Result:** `ecosystem_root = /Users/tercel/WorkSpace/aipartnerup/`

### 0.2 Discover Repositories

Scanned `ecosystem_root` for known patterns. Key discoveries:

| Type | Repo | Lang | Version | Status |
|------|------|------|---------|--------|
| protocol | apcore | -- | -- | clean |
| core-sdk | apcore-python | Python | 0.13.0 | clean |
| core-sdk | apcore-typescript | TypeScript | (detected) | (detected) |
| core-sdk | apcore-rust | Rust | (detected) | (detected) |
| mcp-bridge | apcore-mcp-python | Python | (detected) | (detected) |
| mcp-bridge | apcore-mcp-typescript | TypeScript | (detected) | (detected) |
| integration | django-apcore | Python | (detected) | (detected) |
| integration | flask-apcore | Python | (detected) | (detected) |
| integration | nestjs-apcore | TypeScript | (detected) | (detected) |
| integration | tiptap-apcore | TypeScript | (detected) | (detected) |
| integration | express-apcore | TypeScript | placeholder | (detected) |
| integration | comfyui-apcore | Python | placeholder | (detected) |
| tooling | apcore-studio | -- | placeholder | (detected) |
| shared-lib | apcore-toolkit-python | Python | (detected) | (detected) |
| shared-lib | apcore-toolkit-typescript | TypeScript | (detected) | (detected) |

### 0.3 Detect CWD Repo

CWD is `/Users/tercel/WorkSpace/aipartnerup/` -- this is the ecosystem root itself, not an individual repo.
`cwd_repo = null`

### 0.4 Load Configuration

System defaults applied:
- `protocol_repo = "apcore"`
- `reference_sdk.python = "apcore-python"`
- `reference_sdk.typescript = "apcore-typescript"`

### 0.6 Display Discovery Summary

```
apcore-skills -- Ecosystem Dashboard

Ecosystem root: /Users/tercel/WorkSpace/aipartnerup/
Repos discovered: 15+

  Type          | Repo                    | Lang       | Version | Status
  protocol      | apcore                  | --         | --      | clean
  core-sdk      | apcore-python           | Python     | 0.13.0  | clean
  core-sdk      | apcore-typescript       | TypeScript | ...     | ...
  core-sdk      | apcore-rust             | Rust       | ...     | ...
  mcp-bridge    | apcore-mcp-python       | Python     | ...     | ...
  mcp-bridge    | apcore-mcp-typescript   | TypeScript | ...     | ...
  integration   | django-apcore           | Python     | ...     | ...
  integration   | flask-apcore            | Python     | ...     | ...
  integration   | nestjs-apcore           | TypeScript | ...     | ...
  ...
```

---

## Step 1: Parse Arguments

### Raw Input

`$ARGUMENTS = "go --type core"`

### Parsed Values

1. **`<language>`:** `go` -- extracted as first positional argument.
2. **`--type`:** `core` -- explicitly provided.
3. **`--ref`:** Not specified. Auto-detect logic:
   - CWD (`/Users/tercel/WorkSpace/aipartnerup/`) is NOT a same-type apcore repo (it is the ecosystem root).
   - For `core` type, prefer `apcore-python`.
   - **Resolved:** `apcore-python` at `/Users/tercel/WorkSpace/aipartnerup/apcore-python/`

### Derived Values

- **Target repo name:** `apcore-go` (pattern: `apcore-{lang}`)
- **Target path:** `/Users/tercel/WorkSpace/aipartnerup/apcore-go/` (pattern: `{ecosystem_root}/apcore-{lang}/`)
- **Target directory exists?** No -- directory does not exist. Proceed.

### Display Output

```
SDK Bootstrap:
  Language:   go
  Type:       core
  Reference:  apcore-python (Python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-go/
```

---

## Step 2: Extract API Contract (Sub-agent)

The skill instructs spawning a sub-agent (`Task(subagent_type="general-purpose")`) to extract the complete public API contract from `apcore-python`.

### What the sub-agent would read

1. **`src/apcore/__init__.py`** -- all public exports (the `__all__` list)
2. **Each exported class source file** -- constructor + all public methods
3. **`src/apcore/errors.py`** -- error classes and codes
4. **`src/apcore/middleware/`** -- middleware interfaces
5. **`src/apcore/registry/`** -- registry interfaces
6. **`src/apcore/schema/`** -- schema interfaces
7. **`src/apcore/observability/`** -- observability interfaces
8. **`apcore/PROTOCOL_SPEC.md`** -- authoritative definitions

### Extracted API Contract (sample from real data)

```
API_CONTRACT:
  type: core
  source: apcore-python
  source_version: 0.13.0
  export_count: ~170 (from __all__)
  module_count: ~25 source files/modules

MODULES:
- module: executor
  file: src/apcore/executor.py
  classes:
    - Executor:
        constructor: (registry: Registry, config: Config | None, acl: ACL | None,
                       approval_handler: ApprovalHandler | None,
                       middleware_manager: MiddlewareManager | None)
        methods:
          - execute(module_id, inputs, context) -> dict[str, Any] [async]
          - validate(module_id, inputs, context) -> PreflightResult
          - stream(module_id, inputs, context) -> AsyncIterator[dict[str, Any]] [async]
  functions:
    - redact_sensitive(data, sensitive_keys) -> Any
  constants:
    - REDACTED_VALUE: str = "***REDACTED***"

- module: context
  file: src/apcore/context.py
  classes:
    - Context:
        constructor: (module_id, inputs, identity, call_chain, config, trace_context, ...)
        methods: (attribute access, to_dict, etc.)
    - ContextFactory:
        methods:
          - create(module_id, inputs, identity, ...) -> Context [static]
    - Identity:
        constructor: (user_id, roles, metadata)

- module: module
  file: src/apcore/module.py
  classes:
    - Module:
        methods:
          - execute(context) -> dict [async] [abstract]
          - validate(context) -> ValidationResult
          - describe() -> str
    - ModuleAnnotations: (dataclass with approval_required, requires_human, cost_tier, etc.)
    - ModuleExample: (dataclass with description, inputs, expected_output)
    - ValidationResult: (dataclass)
    - PreflightCheckResult: (dataclass)
    - PreflightResult: (dataclass)

- module: config
  file: src/apcore/config.py
  classes:
    - Config:
        constructor: (config_path, config_dict, ...)
        methods:
          - get(key, default) -> Any
          - get_module_config(module_id) -> dict
          - merge(other) -> Config

- module: errors
  file: src/apcore/errors.py
  classes:
    - ModuleError: (base, code, message, details, cause, trace_id, retryable, ai_guidance, ...)
    - ErrorCodes: (immutable class with ~30 error code constants)
    - ErrorCodeRegistry: (register, unregister, all_codes)
    - (36 specific error classes -- see ERROR_HIERARCHY below)

- module: acl
  file: src/apcore/acl.py
  classes:
    - ACL: (check, add_rule, remove_rule, ...)
    - ACLRule: (dataclass)
    - AuditEntry: (dataclass)

- module: approval
  file: src/apcore/approval.py
  classes:
    - ApprovalHandler: (protocol/interface)
    - ApprovalRequest: (dataclass)
    - ApprovalResult: (dataclass)
    - AlwaysDenyHandler: (implements ApprovalHandler)
    - AutoApproveHandler: (implements ApprovalHandler)
    - CallbackApprovalHandler: (implements ApprovalHandler)

- module: async_task
  file: src/apcore/async_task.py
  classes:
    - AsyncTaskManager: (submit, get_status, cancel, ...)
    - TaskInfo: (dataclass)
    - TaskStatus: (enum)

- module: bindings
  file: src/apcore/bindings.py
  classes:
    - BindingLoader: (load_file, load_dir, ...)

- module: cancel
  file: src/apcore/cancel.py
  classes:
    - CancelToken: (cancel, is_cancelled, ...)
    - ExecutionCancelledError: (error class)

- module: decorator
  file: src/apcore/decorator.py
  classes:
    - FunctionModule: (wraps functions as modules)
  functions:
    - module(...) -> decorator

- module: extensions
  file: src/apcore/extensions.py
  classes:
    - ExtensionManager: (register_point, get_extensions, ...)
    - ExtensionPoint: (dataclass/descriptor)

- module: client
  file: src/apcore/client.py
  classes:
    - APCore:
        constructor: (config, registry, executor, ...)
        methods:
          - call(module_id, inputs, context) -> dict
          - call_async(module_id, inputs, context) -> dict [async]
          - stream(module_id, inputs, context) -> AsyncIterator [async]
          - validate(module_id, inputs, context) -> PreflightResult
          - register(module_id, module_obj) -> None
          - describe(module_id) -> str
          - use(middleware) -> APCore
          - use_before(callback) -> APCore
          - use_after(callback) -> APCore
          - remove(middleware) -> bool
          - discover() -> int
          - list_modules(tags, prefix) -> list[str]
          - on(event_type, handler) -> EventSubscriber
          - off(subscriber) -> None
          - disable(module_id, reason) -> dict
          - enable(module_id, reason) -> dict
          - module(...) -> decorator

- module: trace_context
  file: src/apcore/trace_context.py
  classes:
    - TraceContext: (trace_id, span_id, parent_span_id, ...)
    - TraceParent: (parse, serialize, ...)

- module: version
  file: src/apcore/version.py
  classes:
    - VersionIncompatibleError: (error class)
  functions:
    - negotiate_version(client_version, server_version) -> str

- module: middleware (subdirectory)
  files: __init__.py, base.py, adapters.py, manager.py, logging.py, retry.py, error_history.py, platform_notify.py
  classes:
    - Middleware: (protocol/interface -- before, after, on_error)
    - MiddlewareManager: (add, remove, run_before, run_after)
    - BeforeMiddleware: (simplified before-only)
    - AfterMiddleware: (simplified after-only)
    - LoggingMiddleware: (built-in)
    - RetryMiddleware: (built-in with RetryConfig)
    - ErrorHistoryMiddleware: (built-in)
    - PlatformNotifyMiddleware: (built-in)
    - MiddlewareChainError: (error class)

- module: registry (subdirectory)
  files: __init__.py, registry.py, types.py, scanner.py, validation.py, dependencies.py, conflicts.py, entry_point.py, metadata.py, schema_export.py, version.py
  classes:
    - Registry: (register, unregister, get, list, discover, ...)
    - ModuleDescriptor: (dataclass)
    - DiscoveredModule: (dataclass)
    - DependencyInfo: (dataclass)
    - Discoverer: (protocol)
    - ModuleValidator: (protocol)
  constants:
    - MODULE_ID_PATTERN, MAX_MODULE_ID_LENGTH, RESERVED_WORDS, REGISTRY_EVENTS

- module: schema (subdirectory)
  files: __init__.py, loader.py, validator.py, exporter.py, ref_resolver.py, strict.py, types.py, annotations.py
  classes:
    - SchemaLoader: (load, load_file)
    - SchemaValidator: (validate)
    - SchemaExporter: (export)
    - RefResolver: ($ref resolution)
    - SchemaStrategy: (enum or type)
    - ExportProfile: (enum or type)
  functions:
    - to_strict_schema(schema) -> dict

- module: observability (subdirectory)
  files: __init__.py, tracing.py, metrics.py, context_logger.py, error_history.py, usage.py
  classes:
    - TracingMiddleware, Span, SpanExporter, StdoutExporter, InMemoryExporter, OTLPExporter, create_span
    - MetricsCollector, MetricsMiddleware
    - ContextLogger, ObsLoggingMiddleware
    - ErrorEntry, ErrorHistory
    - UsageCollector, UsageMiddleware

- module: events (subdirectory)
  files: __init__.py, emitter.py, subscribers.py
  classes:
    - ApCoreEvent, EventEmitter, EventSubscriber, WebhookSubscriber, A2ASubscriber

- module: sys_modules
  functions:
    - register_sys_modules, register_subscriber_type, unregister_subscriber_type, reset_subscriber_registry

- module: utils (subdirectory)
  functions:
    - match_pattern, guard_call_chain, propagate_error, normalize_to_canonical_id, calculate_specificity

ERROR_HIERARCHY:
  base: ModuleError
  codes: ErrorCodes (30 constants -- CONFIG_NOT_FOUND, CONFIG_INVALID, ACL_RULE_ERROR,
         ACL_DENIED, MODULE_NOT_FOUND, MODULE_DISABLED, MODULE_TIMEOUT,
         MODULE_LOAD_ERROR, MODULE_EXECUTE_ERROR, RELOAD_FAILED,
         EXECUTION_CANCELLED, SCHEMA_VALIDATION_ERROR, SCHEMA_NOT_FOUND,
         SCHEMA_PARSE_ERROR, SCHEMA_CIRCULAR_REF, CALL_DEPTH_EXCEEDED,
         CIRCULAR_CALL, CALL_FREQUENCY_EXCEEDED, GENERAL_INVALID_INPUT,
         GENERAL_INTERNAL_ERROR, FUNC_MISSING_TYPE_HINT, FUNC_MISSING_RETURN_TYPE,
         BINDING_INVALID_TARGET, BINDING_MODULE_NOT_FOUND,
         BINDING_CALLABLE_NOT_FOUND, BINDING_NOT_CALLABLE,
         BINDING_SCHEMA_MISSING, BINDING_FILE_INVALID, CIRCULAR_DEPENDENCY,
         MIDDLEWARE_CHAIN_ERROR, APPROVAL_DENIED, APPROVAL_TIMEOUT,
         APPROVAL_PENDING, VERSION_INCOMPATIBLE, ERROR_CODE_COLLISION,
         GENERAL_NOT_IMPLEMENTED, DEPENDENCY_NOT_FOUND)
  classes:
    - ConfigNotFoundError(code=CONFIG_NOT_FOUND, parent=ModuleError)
    - ConfigError(code=CONFIG_INVALID, parent=ModuleError)
    - ACLRuleError(code=ACL_RULE_ERROR, parent=ModuleError)
    - ACLDeniedError(code=ACL_DENIED, parent=ModuleError)
    - ApprovalError(parent=ModuleError) -- base for approval errors
    - ApprovalDeniedError(code=APPROVAL_DENIED, parent=ApprovalError)
    - ApprovalTimeoutError(code=APPROVAL_TIMEOUT, parent=ApprovalError)
    - ApprovalPendingError(code=APPROVAL_PENDING, parent=ApprovalError)
    - ModuleNotFoundError(code=MODULE_NOT_FOUND, parent=ModuleError)
    - ModuleDisabledError(code=MODULE_DISABLED, parent=ModuleError)
    - ModuleTimeoutError(code=MODULE_TIMEOUT, parent=ModuleError)
    - SchemaValidationError(code=SCHEMA_VALIDATION_ERROR, parent=ModuleError)
    - SchemaNotFoundError(code=SCHEMA_NOT_FOUND, parent=ModuleError)
    - SchemaParseError(code=SCHEMA_PARSE_ERROR, parent=ModuleError)
    - SchemaCircularRefError(code=SCHEMA_CIRCULAR_REF, parent=ModuleError)
    - CallDepthExceededError(code=CALL_DEPTH_EXCEEDED, parent=ModuleError)
    - CircularCallError(code=CIRCULAR_CALL, parent=ModuleError)
    - CallFrequencyExceededError(code=CALL_FREQUENCY_EXCEEDED, parent=ModuleError)
    - InvalidInputError(code=GENERAL_INVALID_INPUT, parent=ModuleError)
    - FuncMissingTypeHintError(code=FUNC_MISSING_TYPE_HINT, parent=ModuleError)
    - FuncMissingReturnTypeError(code=FUNC_MISSING_RETURN_TYPE, parent=ModuleError)
    - BindingInvalidTargetError(code=BINDING_INVALID_TARGET, parent=ModuleError)
    - BindingModuleNotFoundError(code=BINDING_MODULE_NOT_FOUND, parent=ModuleError)
    - BindingCallableNotFoundError(code=BINDING_CALLABLE_NOT_FOUND, parent=ModuleError)
    - BindingNotCallableError(code=BINDING_NOT_CALLABLE, parent=ModuleError)
    - BindingSchemaMissingError(code=BINDING_SCHEMA_MISSING, parent=ModuleError)
    - BindingFileInvalidError(code=BINDING_FILE_INVALID, parent=ModuleError)
    - CircularDependencyError(code=CIRCULAR_DEPENDENCY, parent=ModuleError)
    - ModuleLoadError(code=MODULE_LOAD_ERROR, parent=ModuleError)
    - ModuleExecuteError(code=MODULE_EXECUTE_ERROR, parent=ModuleError)
    - ReloadFailedError(code=RELOAD_FAILED, parent=ModuleError)
    - InternalError(code=GENERAL_INTERNAL_ERROR, parent=ModuleError)
    - FeatureNotImplementedError(code=GENERAL_NOT_IMPLEMENTED, parent=ModuleError)
    - DependencyNotFoundError(code=DEPENDENCY_NOT_FOUND, parent=ModuleError)
    - ErrorCodeCollisionError(code=ERROR_CODE_COLLISION, parent=ModuleError)

EXTENSION_POINTS:
  - Middleware: before(context) -> context, after(context, result) -> result, on_error(context, error) -> error
  - ApprovalHandler: check(request) -> ApprovalResult [async]
  - Discoverer: discover(paths) -> list[DiscoveredModule]
  - ModuleValidator: validate(module) -> list[str]
  - Module: execute(context) -> dict [async], validate(context) -> ValidationResult
  - SpanExporter: export(spans) -> None
  - EventSubscriber: on_event(event) -> None [async]
```

---

## Step 3: Confirm Tech Stack

The skill instructs using `AskUserQuestion` with Go-specific options. The questions would be:

### Questions Presented to User

**Go SDK Tech Stack Configuration:**

1. **Go version:** "1.21+ (Recommended)" / "1.22+"
2. **Module path:** default `github.com/aipartnerup/apcore-go`
3. **Test extras:** "Standard testing (Recommended)" / "testify"
4. **Schema validation:** "go-jsonschema (Recommended)" / "gojsonschema" / "Other"

### Simulated Answers (assuming defaults)

```
tech_stack:
  go_version: "1.21+"
  module_path: "github.com/aipartnerup/apcore-go"
  test_framework: "Standard testing"
  schema_validation: "go-jsonschema"
```

---

## Step 4: Scaffold Project (Sub-agent)

The skill instructs spawning a sub-agent to create the project skeleton. Based on the SKILL.md Step 4 template for **core SDK**, here is the COMPLETE project structure that would be scaffolded:

### Complete File and Directory Listing

```
/Users/tercel/WorkSpace/aipartnerup/apcore-go/
├── go.mod                              # Build config: module github.com/aipartnerup/apcore-go, go 1.21
├── .gitignore                          # Go-appropriate patterns (*.exe, /vendor/, etc.)
├── README.md                           # Project name, description, installation, link to docs
├── CHANGELOG.md                        # Empty "## [Unreleased]" section
├── LICENSE                             # Apache-2.0 (detected from apcore-python pyproject.toml)
├── src/                                # NOTE: see analysis below
│   ├── apcore.go                       # Main module file -- package exports (empty stubs with TODO)
│   ├── executor.go                     # Stub -- Executor struct + Execute/Validate/Stream methods
│   ├── context.go                      # Stub -- Context, ContextFactory, Identity structs
│   ├── module.go                       # Stub -- Module interface, ModuleAnnotations, etc.
│   ├── config.go                       # Stub -- Config struct + Get/GetModuleConfig/Merge
│   ├── errors.go                       # Stub with ErrorCode constants and base ModuleError
│   ├── acl.go                          # Stub -- ACL, ACLRule, AuditEntry
│   ├── approval.go                     # Stub -- ApprovalHandler interface, ApprovalRequest, etc.
│   ├── async_task.go                   # Stub -- AsyncTaskManager, TaskInfo, TaskStatus
│   ├── bindings.go                     # Stub -- BindingLoader
│   ├── decorator.go                    # Stub -- FunctionModule, module decorator equivalent
│   ├── extensions.go                   # Stub -- ExtensionManager, ExtensionPoint
│   ├── cancel.go                       # Stub -- CancelToken, ExecutionCancelledError
│   ├── trace_context.go               # Stub -- TraceContext, TraceParent
│   ├── middleware/                      # Stub directory
│   ├── registry/                       # Stub directory
│   ├── schema/                         # Stub directory
│   ├── observability/                  # Stub directory
│   └── utils/                          # Stub directory
└── tests/
    └── (test runner config)            # For Go: no separate config needed, `go test ./...`
```

### What the Skill EXPLICITLY Creates -- Analysis

**Files the skill template lists (Step 4 "Project Structure" for core SDK):**

| File/Dir | Created? | Content |
|----------|----------|---------|
| `{build-config}` (go.mod) | YES | Module declaration, Go version |
| `.gitignore` | YES | Go patterns |
| `README.md` | YES | Name, description, installation, docs link |
| `CHANGELOG.md` | YES | `## [Unreleased]` |
| `LICENSE` | YES | Apache-2.0 |
| `src/{main-module-file}` | YES | apcore.go with exports |
| `src/executor.go` | YES | Stub |
| `src/context.go` | YES | Stub |
| `src/module.go` | YES | Stub |
| `src/config.go` | YES | Stub |
| `src/errors.go` | YES | Stub with ErrorCode enum and base error |
| `src/acl.go` | YES | Stub |
| `src/approval.go` | YES | Stub |
| `src/async_task.go` | YES | Stub |
| `src/bindings.go` | YES | Stub |
| `src/decorator.go` | YES | Stub |
| `src/extensions.go` | YES | Stub |
| `src/cancel.go` | YES | Stub |
| `src/trace_context.go` | YES | Stub |
| `src/middleware/` | YES | Stub directory |
| `src/registry/` | YES | Stub directory |
| `src/schema/` | YES | Stub directory |
| `src/observability/` | YES | Stub directory |
| `src/utils/` | YES | Stub directory |
| `tests/` | YES | Directory with test runner config |

### What about tests/ content?

The **SKILL.md Step 4 template** only specifies:

```
└── tests/
    └── {test-config}                # pytest.ini / vitest.config / test runner config
```

That is ALL the skill tells the scaffold sub-agent to create for tests. There is **no instruction to create individual test files**, test subdirectories (integration/, registry/, schema/, observability/), test helpers (conftest equivalent), or any test stubs. The conventions.md file describes a richer test structure, but the Step 4 sub-agent prompt does NOT reference conventions.md's "Project Structure Convention" -- it only includes the inline template shown above.

For Go specifically, `go test` does not need a separate config file, so the `tests/` directory would essentially be empty or contain a minimal placeholder.

### What about examples/ content?

The **SKILL.md Step 4 template does NOT include an `examples/` directory** at all. The template lists only `src/` and `tests/` as subdirectories. Despite conventions.md specifying an `examples/` directory with `simple_client.{ext}`, `bindings/`, and `modules/` subdirectories, the Step 4 sub-agent prompt does not instruct their creation.

**No examples/ directory or files would be created.**

### What about events/, sys_modules/, client.go, version.go?

The SKILL.md Step 4 template does NOT list these as files to create. They exist in `apcore-python` but are not part of the scaffold template. Missing from the template:
- `client.go` (the APCore facade class)
- `version.go` (version negotiation)
- `events/` directory
- `sys_modules/` directory
- `_docstrings.go` equivalent

These would need to be added during implementation via code-forge, but the scaffold does not create them.

### Stub File Content (per skill instructions)

Each stub file would contain:
1. Package header comment referencing the protocol spec section
2. Import of base types from the main module
3. Struct/interface stubs with correct signatures from the API contract
4. TODO comments indicating what needs to be implemented
5. Go type annotations (PascalCase for public, camelCase for private)

Example `executor.go` stub:
```go
// Package apcore - Executor implementation
// Reference: PROTOCOL_SPEC.md - Execution Pipeline
package apcore

// TODO: Implement Executor
// See apcore-python/src/apcore/executor.py for reference implementation

// Executor orchestrates module execution with middleware, ACL, approval, and schema validation.
type Executor struct {
    // TODO: fields
}

// Execute runs a module with the given inputs and context.
// TODO: Implement execution pipeline
func (e *Executor) Execute(moduleID string, inputs map[string]any, ctx *Context) (map[string]any, error) {
    panic("not implemented")
}

// Validate performs preflight validation on a module.
// TODO: Implement validation
func (e *Executor) Validate(moduleID string, inputs map[string]any, ctx *Context) (*PreflightResult, error) {
    panic("not implemented")
}
```

### Post-scaffold verification

The skill checks:
- [x] Build config file exists (go.mod)
- [x] Main module file exists with exports (src/apcore.go)
- [x] At least 5 source files exist (14 .go files)
- [x] Tests directory exists (tests/)
- [x] README.md exists

---

## Step 5: Generate Feature Specs

The skill checks for feature specs at `{protocol_path}/docs/features/*.md`.

**Result:** Feature specs FOUND -- 10 specs in `/Users/tercel/WorkSpace/aipartnerup/apcore/docs/features/`:
- acl-system.md
- approval-system.md
- core-executor.md
- decorator-bindings.md
- event-system.md
- middleware-system.md
- observability.md
- registry-system.md
- schema-system.md
- system-modules.md

The skill would link these via `.code-forge.json` and display:
```
Feature specs found: 10 specs in /Users/tercel/WorkSpace/aipartnerup/apcore/docs/features/
```

No new feature specs would be generated since they already exist.

---

## Step 6: Generate .code-forge.json

The skill would write `/Users/tercel/WorkSpace/aipartnerup/apcore-go/.code-forge.json`:

```json
{
  "_tool": {
    "name": "code-forge",
    "description": "Transform documentation into actionable development plans",
    "url": "https://github.com/tercel/code-forge"
  },
  "directories": {
    "base": "./",
    "input": "../apcore/docs/features",
    "output": "planning/"
  },
  "reference_docs": {
    "sources": ["../apcore-python/planning/*/plan.md"]
  },
  "port": {
    "source_docs": "../apcore",
    "reference_impl": "../apcore-python",
    "target_lang": "go"
  },
  "execution": {
    "default_mode": "ask",
    "auto_tdd": true,
    "task_granularity": "medium"
  }
}
```

Git init is left to user:
```
Project scaffolded. To initialize git:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-go/
  git init
  git add <files...>
  git commit -m "chore: initialize apcore-go project skeleton"
```

---

## Step 7: Display Summary and Next Steps

```
apcore-skills:sdk -- SDK Bootstrap Complete

Target: /Users/tercel/WorkSpace/aipartnerup/apcore-go/
Language: go
Type: core
Modules: 14 source files scaffolded
Feature specs: 10 specs available
API contract: ~170 public symbols to implement

Project structure:
  apcore-go/
  ├── go.mod
  ├── .gitignore
  ├── README.md
  ├── CHANGELOG.md
  ├── LICENSE
  ├── .code-forge.json
  ├── src/
  │   ├── apcore.go
  │   ├── executor.go
  │   ├── context.go
  │   ├── module.go
  │   ├── config.go
  │   ├── errors.go
  │   ├── acl.go
  │   ├── approval.go
  │   ├── async_task.go
  │   ├── bindings.go
  │   ├── decorator.go
  │   ├── extensions.go
  │   ├── cancel.go
  │   ├── trace_context.go
  │   ├── middleware/
  │   ├── registry/
  │   ├── schema/
  │   ├── observability/
  │   └── utils/
  └── tests/

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-go/
  /code-forge:port @../apcore --ref apcore-python --lang go    Generate implementation plans
  /code-forge:impl core-executor                                Start implementing
  /apcore-skills:sync --lang go,python                          Verify API consistency
```

---

## Critical Observations: Gaps in the OLD Skill

### 1. tests/ is nearly empty
The Step 4 template only creates `tests/{test-config}`. For Go there is no test config file, so the directory is effectively empty. The conventions.md describes a rich test structure (per-module test files, integration/, registry/, schema/, observability/ subdirs, conftest/helpers) but the Step 4 sub-agent prompt does NOT reference this. **TDD is declared mandatory in the Iron Law section, but no test files are scaffolded.**

### 2. examples/ is completely missing
The conventions.md specifies `examples/` with `simple_client.{ext}`, `bindings/`, and `modules/` but the Step 4 scaffold template omits it entirely. The reference implementation (apcore-python) has `examples/simple_client.py`, `examples/global_client.py`, `examples/bindings/`, and `examples/modules/`.

### 3. Several source modules from the reference are not scaffolded
The template does not include:
- `client.go` (the APCore facade -- the primary user-facing API)
- `version.go` (version negotiation)
- `events/` directory (event system)
- `sys_modules/` directory (system modules)

### 4. Go project structure is not idiomatic
Using `src/` as the source directory is not idiomatic Go. Go projects typically place packages at the root or under `pkg/` or `internal/`. The skill template uses a generic `src/` directory for all languages, violating the Anti-Rationalization Table's own advice: "Use idiomatic target-language patterns."

### 5. Subdirectory stubs have no files
The template creates `middleware/`, `registry/`, `schema/`, `observability/`, and `utils/` as "stub directories" but does not specify what files go inside them. The reference has multiple files in each (e.g., registry has 11 files). These would be empty directories.

### 6. conventions.md is referenced but not fully used
Step 3 references conventions.md for "Testing Conventions" and "Dependency Conventions" but the Step 4 sub-agent prompt does not include the full Project Structure Convention from conventions.md.
