# SDK Bootstrap Simulation: `/apcore-skills:sdk go --type core`

**Simulated**: 2026-03-13
**Skill version**: SKILL.md (current)
**Working directory**: `/Users/tercel/WorkSpace/aipartnerup/`

---

## Step 0: Ecosystem Discovery

Ecosystem root detected at `/Users/tercel/WorkSpace/aipartnerup/` based on the presence of `apcore/`, `apcore-python/`, `apcore-typescript/`, and other ecosystem repos.

Protocol spec located at: `apcore/PROTOCOL_SPEC.md`

---

## Step 1: Parse Arguments

**Input**: `go --type core`

| Parameter | Parsed Value | Source |
|-----------|-------------|--------|
| `<language>` | `go` | Positional argument |
| `--type` | `core` | Explicit flag |
| `--ref` | `apcore-python` | Auto-detect: CWD is not a same-type repo, default for `core` is `apcore-python` |

**Derived values**:

- Target repo name: `apcore-go`
- Target path: `/Users/tercel/WorkSpace/aipartnerup/apcore-go/`
- Package name: `apcore` (Go module: `github.com/aipartnerup/apcore-go`)
- Reference path: `/Users/tercel/WorkSpace/aipartnerup/apcore-python/`

**Target directory check**: Does not exist -- proceed with scaffolding.

```
SDK Bootstrap:
  Language:   go
  Type:       core
  Reference:  apcore-python (Python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-go/
```

---

## Step 2: API Contract Extraction

Sub-agent would analyze the reference implementation at `apcore-python/`. Here is what it finds from real data:

### API_CONTRACT

```
API_CONTRACT:
  type: core
  source: apcore-python
  source_version: (from pyproject.toml / importlib.metadata)
  export_count: 108  (counted from __all__ in __init__.py)
  module_count: 20+  (source files + subdirectory packages)
```

### MODULES (key extracts from real source)

```
MODULES:
- module: executor
  file: src/apcore/executor.py
  classes:
    - Executor:
        constructor: (registry: Registry, middleware_manager: MiddlewareManager | None, ...)
        methods:
          - execute(module_id, inputs, context) -> dict[str, Any] [async variant]
          - execute_stream(module_id, inputs, context) -> AsyncIterator[dict] [async]
    - redact_sensitive(data) -> dict [function]
  constants:
    - REDACTED_VALUE: str = "***REDACTED***"

- module: client
  file: src/apcore/client.py
  classes:
    - APCore:
        constructor: (registry=None, executor=None, ...)
        methods:
          - call(module_id, inputs, context) -> dict
          - call_async(module_id, inputs, context) -> dict [async]
          - stream(module_id, inputs, context) -> AsyncIterator [async]
          - validate(module_id, inputs, context) -> PreflightResult
          - register(module_id, module_obj) -> None
          - describe(module_id) -> str
          - module(id, description, ...) -> decorator
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

- module: context
  file: src/apcore/context.py
  classes:
    - Context:
        constructor: (identity, trace_id, metadata, ...)
        methods: (getters/setters for identity, trace, services)
    - ContextFactory:
        methods:
          - create(identity, ...) -> Context [static]
    - Identity:
        fields: (id, role, permissions, ...)

- module: module
  file: src/apcore/module.py
  classes:
    - Module: (protocol/interface — input_schema, output_schema, execute, description)
    - ModuleAnnotations: (readonly, idempotent, destructive, open_world)
    - ModuleExample: (title, inputs, output, description)
    - PreflightCheckResult / PreflightResult / ValidationResult

- module: config
  file: src/apcore/config.py
  classes:
    - Config:
        constructor: (config_path | dict)
        methods: get(key, default), load(), ...

- module: errors
  file: src/apcore/errors.py
  classes: (see ERROR_HIERARCHY below)

- module: acl
  file: src/apcore/acl.py
  classes:
    - ACL: check(caller_id, target_id, context), add_rule(), remove_rule()
    - ACLRule: (caller_pattern, target_pattern, effect, priority)
    - AuditEntry: (timestamp, caller_id, target_id, decision, rule)

- module: approval
  file: src/apcore/approval.py
  classes:
    - ApprovalHandler (interface), ApprovalRequest, ApprovalResult
    - AutoApproveHandler, AlwaysDenyHandler, CallbackApprovalHandler

- module: async_task
  file: src/apcore/async_task.py
  classes:
    - AsyncTaskManager: submit(), get_status(), cancel(), wait()
    - TaskInfo: (task_id, status, result, error)
    - TaskStatus: (enum: PENDING, RUNNING, COMPLETED, FAILED, CANCELLED)

- module: bindings
  file: src/apcore/bindings.py
  classes:
    - BindingLoader: load_from_yaml(path) -> list[Module]

- module: cancel
  file: src/apcore/cancel.py
  classes:
    - CancelToken: cancel(), is_cancelled, on_cancel(callback)
    - ExecutionCancelledError(ModuleError)

- module: decorator
  file: src/apcore/decorator.py
  classes:
    - FunctionModule: (wraps a decorated function as a Module)
  functions:
    - module(id, description, ...) -> decorator

- module: extensions
  file: src/apcore/extensions.py
  classes:
    - ExtensionManager: register_point(), get_point(), list_points()
    - ExtensionPoint: register(handler), execute(context)

- module: trace_context
  file: src/apcore/trace_context.py
  classes:
    - TraceContext: inject(headers), extract(headers)
    - TraceParent: (trace_id, span_id, trace_flags)

- module: middleware (package)
  file: src/apcore/middleware/__init__.py
  classes:
    - Middleware (interface): before(), after(), on_error()
    - MiddlewareManager: add(), remove(), execute_chain()
    - BeforeMiddleware, AfterMiddleware (convenience bases)
    - LoggingMiddleware, RetryMiddleware, ErrorHistoryMiddleware, PlatformNotifyMiddleware
    - RetryConfig, MiddlewareChainError

- module: registry (package)
  file: src/apcore/registry/
  classes:
    - Registry: register(), get(), list(), discover(), resolve_dependencies()
    - ModuleDescriptor, DiscoveredModule, DependencyInfo (types)
    - Discoverer (protocol), ModuleValidator (protocol)
  constants:
    - MODULE_ID_PATTERN, MAX_MODULE_ID_LENGTH, RESERVED_WORDS, REGISTRY_EVENTS

- module: schema (package)
  file: src/apcore/schema/
  classes:
    - SchemaLoader, SchemaValidator, SchemaExporter, RefResolver
    - SchemaStrategy (enum), ExportProfile
  functions:
    - to_strict_schema(schema) -> dict

- module: observability (package)
  file: src/apcore/observability/
  classes:
    - TracingMiddleware, MetricsMiddleware, MetricsCollector
    - ContextLogger, ObsLoggingMiddleware
    - Span, SpanExporter (interface), StdoutExporter, InMemoryExporter, OTLPExporter
    - ErrorEntry, ErrorHistory
    - UsageCollector, UsageMiddleware
  functions:
    - create_span(name, ...) -> Span

- module: events (package)
  file: src/apcore/events/
  classes:
    - ApCoreEvent, EventEmitter, EventSubscriber
    - WebhookSubscriber, A2ASubscriber

- module: version
  file: src/apcore/version.py
  classes:
    - VersionIncompatibleError(ModuleError)
  functions:
    - negotiate_version(requested, supported) -> str

- module: utils (package)
  file: src/apcore/utils/
  functions:
    - match_pattern(pattern, value) -> bool
    - guard_call_chain(call_stack, max_depth) -> None
    - propagate_error(error, context) -> ModuleError
    - normalize_to_canonical_id(raw_id) -> str
    - calculate_specificity(pattern) -> int
```

### ERROR_HIERARCHY

```
ERROR_HIERARCHY:
  base: ModuleError(code, message, details, cause, trace_id, retryable, ai_guidance, user_fixable, suggestion)
  codes: ErrorCodes (constants class with 33 codes)
    CONFIG_NOT_FOUND, CONFIG_INVALID, ACL_RULE_ERROR, ACL_DENIED,
    MODULE_NOT_FOUND, MODULE_DISABLED, MODULE_TIMEOUT, MODULE_LOAD_ERROR,
    MODULE_EXECUTE_ERROR, RELOAD_FAILED, EXECUTION_CANCELLED,
    SCHEMA_VALIDATION_ERROR, SCHEMA_NOT_FOUND, SCHEMA_PARSE_ERROR, SCHEMA_CIRCULAR_REF,
    CALL_DEPTH_EXCEEDED, CIRCULAR_CALL, CALL_FREQUENCY_EXCEEDED,
    GENERAL_INVALID_INPUT, GENERAL_INTERNAL_ERROR,
    FUNC_MISSING_TYPE_HINT, FUNC_MISSING_RETURN_TYPE,
    BINDING_INVALID_TARGET, BINDING_MODULE_NOT_FOUND, BINDING_CALLABLE_NOT_FOUND,
    BINDING_NOT_CALLABLE, BINDING_SCHEMA_MISSING, BINDING_FILE_INVALID,
    CIRCULAR_DEPENDENCY, MIDDLEWARE_CHAIN_ERROR, APPROVAL_DENIED, ...
  classes:
    - ConfigNotFoundError(code=CONFIG_NOT_FOUND, parent=ModuleError)
    - ConfigError(code=CONFIG_INVALID, parent=ModuleError)
    - ACLRuleError(code=ACL_RULE_ERROR, parent=ModuleError)
    - ACLDeniedError(code=ACL_DENIED, parent=ModuleError)
    - ApprovalError(parent=ModuleError) -> ApprovalDeniedError, ApprovalTimeoutError, ApprovalPendingError
    - ModuleNotFoundError, ModuleDisabledError, ModuleTimeoutError
    - ModuleLoadError, ModuleExecuteError, ReloadFailedError
    - SchemaValidationError, SchemaNotFoundError, SchemaParseError, SchemaCircularRefError
    - CallDepthExceededError, CircularCallError, CallFrequencyExceededError
    - InvalidInputError, InternalError, FeatureNotImplementedError
    - FuncMissingTypeHintError, FuncMissingReturnTypeError
    - BindingInvalidTargetError, BindingModuleNotFoundError, BindingCallableNotFoundError
    - BindingNotCallableError, BindingSchemaMissingError, BindingFileInvalidError
    - CircularDependencyError, ErrorCodeCollisionError
    - MiddlewareChainError (in middleware package)
```

### EXTENSION_POINTS

```
EXTENSION_POINTS:
  - Middleware: before(context, module_id, inputs), after(context, module_id, result), on_error(context, module_id, error)
  - Discoverer: discover(path) -> list[DiscoveredModule]
  - ModuleValidator: validate(module) -> ValidationResult
  - ApprovalHandler: request_approval(request) -> ApprovalResult
  - SpanExporter: export(spans) -> None
  - EventSubscriber: handle(event) -> None
```

### EXAMPLES (from reference)

```
EXAMPLES:
  - simple_client.py: Create APCore client, register decorator-based module, call sync
  - global_client.py: Use default global apcore.call() without explicit client instantiation
  - modules/greet.py: Minimal duck-typed module class with input/output schemas
  - modules/get_user.py: Readonly module with ModuleAnnotations (readonly, idempotent)
  - modules/send_email.py: Full-featured module with annotations, examples, sensitive fields, ContextLogger
  - modules/decorated_add.py: Decorator-based module definition (@module decorator)
  - bindings/format_date/binding.yaml: YAML binding config mapping module_id to target callable
  - bindings/format_date/format_date.py: Target function for YAML binding
```

### TESTS (from reference)

```
TESTS:
  structure:
    - tests/ (root: 40+ test files)
    - tests/integration/ (9 test files: e2e_flow, acl_enforcement, async_flows, binding_modules, decorator_modules, error_propagation, full_lifecycle, infra_smoke, middleware_chain)
    - tests/registry/ (12 test files: registry, dependencies, entry_point, integration, metadata, types, validation, version_negotiation, scanner, schema_export, circular_dependency_error, module_execute_and_internal_errors)
    - tests/schema/ (8 test files: annotations, edge_cases, exporter, loader, ref_resolver, strict, types, validator + fixtures/)
    - tests/observability/ (6 test files: context_logger, error_history, metrics, observability_package, tracing, usage)
    - tests/events/ (2 test files: emitter, subscribers)
    - tests/sys_modules/ (5 test files: control, health, manifest, registration, usage)
    - tests/conformance/ (1 test file: conformance + fixtures/)
    - tests/examples/ (1 test file: test_example_modules)
  helpers:
    - conftest.py (root)
    - integration/conftest.py
    - registry/conftest.py
    - schema/conftest.py
  total_count: ~84 test files
```

---

## Step 3: Tech Stack Questions for Go

The skill specifies these questions for Go (would be presented via `AskUserQuestion`):

```
Go Tech Stack Configuration:
  1. Go version: "1.21+ (Recommended)" / "1.22+"
  2. Module path: default "github.com/aipartnerup/apcore-go"
  3. Test extras: "Standard testing (Recommended)" / "testify"
  4. Schema validation: "go-jsonschema (Recommended)" / "gojsonschema" / "Other"
```

**Simulated answers** (using recommended defaults):

```
tech_stack:
  go_version: "1.21"
  module_path: "github.com/aipartnerup/apcore-go"
  test_framework: "standard testing"
  schema_validation: "go-jsonschema"
```

---

## Step 4: Complete Scaffolded Project Structure

Below is the COMPLETE file and directory listing that the skill would generate. Every file is listed. The skill instructions specify Go naming: PascalCase for public, camelCase for private.

```
apcore-go/
├── go.mod                                    # module github.com/aipartnerup/apcore-go, go 1.21
├── go.sum                                    # empty (no deps yet)
├── .gitignore                                # Go-appropriate: bin/, *.exe, vendor/, .idea/, etc.
├── README.md                                 # project name, description, installation, link to docs
├── CHANGELOG.md                              # empty "## [Unreleased]" section
├── LICENSE                                   # MIT (matching ecosystem)
├── .code-forge.json                          # code-forge config for plan generation
│
├── apcore.go                                 # Package apcore — top-level exports, version const
├── executor.go                               # Executor struct + Execute(), ExecuteStream()
├── client.go                                 # APCore client struct: Call(), CallAsync(), Stream(), Validate(), Register(), etc.
├── context.go                                # Context, ContextFactory, Identity structs
├── module.go                                 # Module interface, ModuleAnnotations, ModuleExample, PreflightResult, ValidationResult
├── config.go                                 # Config struct: Load(), Get()
├── errors.go                                 # ModuleError base, ErrorCodes constants, all 33 error types
├── acl.go                                    # ACL, ACLRule, AuditEntry structs
├── approval.go                               # ApprovalHandler interface, ApprovalRequest, ApprovalResult, AutoApproveHandler, AlwaysDenyHandler, CallbackApprovalHandler
├── async_task.go                             # AsyncTaskManager, TaskInfo, TaskStatus
├── bindings.go                               # BindingLoader: LoadFromYAML()
├── cancel.go                                 # CancelToken, ExecutionCancelledError
├── decorator.go                              # FunctionModule struct, ModuleFunc() decorator-equivalent
├── extensions.go                             # ExtensionManager, ExtensionPoint
├── trace_context.go                          # TraceContext, TraceParent
├── version.go                                # negotiate_version(), VersionIncompatibleError
│
├── middleware/
│   ├── middleware.go                         # Middleware interface, MiddlewareManager, BeforeMiddleware, AfterMiddleware
│   ├── logging.go                            # LoggingMiddleware
│   ├── retry.go                              # RetryMiddleware, RetryConfig
│   ├── error_history.go                      # ErrorHistoryMiddleware
│   ├── platform_notify.go                    # PlatformNotifyMiddleware
│   └── errors.go                             # MiddlewareChainError
│
├── registry/
│   ├── registry.go                           # Registry struct: Register(), Get(), List(), Discover(), ResolveDependencies()
│   ├── types.go                              # ModuleDescriptor, DiscoveredModule, DependencyInfo
│   ├── constants.go                          # ModuleIDPattern, MaxModuleIDLength, ReservedWords, RegistryEvents
│   ├── discoverer.go                         # Discoverer interface
│   └── validator.go                          # ModuleValidator interface
│
├── schema/
│   ├── loader.go                             # SchemaLoader
│   ├── validator.go                          # SchemaValidator
│   ├── exporter.go                           # SchemaExporter, ExportProfile
│   ├── ref_resolver.go                       # RefResolver
│   ├── strategy.go                           # SchemaStrategy enum/constants
│   └── strict.go                             # ToStrictSchema()
│
├── observability/
│   ├── tracing.go                            # TracingMiddleware, Span, SpanExporter, StdoutExporter, InMemoryExporter, OTLPExporter, CreateSpan()
│   ├── metrics.go                            # MetricsMiddleware, MetricsCollector
│   ├── logging.go                            # ContextLogger, ObsLoggingMiddleware
│   ├── error_history.go                      # ErrorEntry, ErrorHistory
│   └── usage.go                              # UsageCollector, UsageMiddleware
│
├── events/
│   ├── events.go                             # ApCoreEvent, EventEmitter, EventSubscriber interface
│   ├── webhook.go                            # WebhookSubscriber
│   └── a2a.go                                # A2ASubscriber
│
├── sysmodules/
│   ├── registration.go                       # RegisterSysModules(), RegisterSubscriberType(), UnregisterSubscriberType(), ResetSubscriberRegistry()
│   ├── health.go                             # Health system module
│   ├── manifest.go                           # Manifest system module
│   ├── control.go                            # Control system module
│   └── usage.go                              # Usage system module
│
├── utils/
│   ├── pattern.go                            # MatchPattern(), CalculateSpecificity()
│   ├── call_chain.go                         # GuardCallChain()
│   ├── error_propagation.go                  # PropagateError()
│   └── normalize.go                          # NormalizeToCanonicalID()
│
├── tests/
│   ├── helpers_test.go                       # Shared test fixtures: mock executor, sample context, sample module config
│   ├── executor_test.go                      # Executor unit tests — stub per method: TestExecute, TestExecuteStream
│   ├── client_test.go                        # APCore client tests — TestCall, TestCallAsync, TestStream, TestValidate, TestRegister, TestDescribe, TestUse, TestDiscover, TestListModules, TestDisableEnable
│   ├── context_test.go                       # Context, ContextFactory, Identity tests
│   ├── module_test.go                        # Module interface compliance, ModuleAnnotations, ModuleExample, PreflightResult
│   ├── config_test.go                        # Config Load/Get tests
│   ├── errors_test.go                        # Error hierarchy: TestModuleError, TestErrorCodes, TestToDict, one test per error subtype
│   ├── acl_test.go                           # ACL check/add_rule/remove_rule tests, ACLRule tests, AuditEntry tests
│   ├── approval_test.go                      # ApprovalHandler, AutoApproveHandler, AlwaysDenyHandler, CallbackApprovalHandler tests
│   ├── async_task_test.go                    # AsyncTaskManager submit/status/cancel/wait, TaskStatus enum tests
│   ├── bindings_test.go                      # BindingLoader LoadFromYAML tests
│   ├── cancel_test.go                        # CancelToken cancel/is_cancelled/on_cancel, ExecutionCancelledError tests
│   ├── decorator_test.go                     # FunctionModule, ModuleFunc decorator tests
│   ├── extensions_test.go                    # ExtensionManager, ExtensionPoint register/execute tests
│   ├── trace_context_test.go                 # TraceContext inject/extract, TraceParent tests
│   ├── version_test.go                       # NegotiateVersion, VersionIncompatibleError tests
│   │
│   ├── middleware/
│   │   ├── middleware_test.go                # Middleware interface, MiddlewareManager add/remove/execute_chain
│   │   ├── logging_test.go                   # LoggingMiddleware tests
│   │   ├── retry_test.go                     # RetryMiddleware, RetryConfig tests
│   │   ├── error_history_test.go             # ErrorHistoryMiddleware tests
│   │   └── platform_notify_test.go           # PlatformNotifyMiddleware tests
│   │
│   ├── registry/
│   │   ├── registry_test.go                  # Registry Register/Get/List/Discover tests
│   │   ├── types_test.go                     # ModuleDescriptor, DiscoveredModule, DependencyInfo tests
│   │   ├── dependencies_test.go              # Dependency resolution tests
│   │   ├── validation_test.go                # ModuleValidator tests
│   │   ├── scanner_test.go                   # Entry point scanner tests
│   │   ├── schema_export_test.go             # Schema export from registry tests
│   │   ├── version_negotiation_test.go       # Version negotiation in registry context
│   │   ├── metadata_test.go                  # Module metadata tests
│   │   └── circular_dependency_test.go       # CircularDependencyError tests
│   │
│   ├── schema/
│   │   ├── loader_test.go                    # SchemaLoader tests
│   │   ├── validator_test.go                 # SchemaValidator tests
│   │   ├── exporter_test.go                  # SchemaExporter, ExportProfile tests
│   │   ├── ref_resolver_test.go              # RefResolver tests
│   │   ├── strict_test.go                    # ToStrictSchema tests
│   │   ├── annotations_test.go              # Schema annotation tests
│   │   ├── edge_cases_test.go               # Schema edge case tests
│   │   ├── types_test.go                     # Schema type tests
│   │   └── fixtures/
│   │       ├── simple.schema.yaml
│   │       ├── with_refs.schema.yaml
│   │       ├── circular_a.schema.yaml
│   │       ├── circular_b.schema.yaml
│   │       ├── empty.schema.yaml
│   │       ├── invalid_syntax.yaml
│   │       ├── nested_objects.schema.yaml
│   │       ├── llm_extensions.schema.yaml
│   │       └── common/
│   │           ├── address.schema.yaml
│   │           └── error.schema.yaml
│   │
│   ├── observability/
│   │   ├── tracing_test.go                   # TracingMiddleware, Span, SpanExporter tests
│   │   ├── metrics_test.go                   # MetricsMiddleware, MetricsCollector tests
│   │   ├── logging_test.go                   # ContextLogger, ObsLoggingMiddleware tests
│   │   ├── error_history_test.go             # ErrorEntry, ErrorHistory tests
│   │   ├── usage_test.go                     # UsageCollector, UsageMiddleware tests
│   │   └── package_test.go                   # Observability package-level export tests
│   │
│   ├── events/
│   │   ├── emitter_test.go                   # EventEmitter tests
│   │   └── subscribers_test.go               # WebhookSubscriber, A2ASubscriber tests
│   │
│   ├── sysmodules/
│   │   ├── registration_test.go              # RegisterSysModules, RegisterSubscriberType tests
│   │   ├── health_test.go                    # Health module tests
│   │   ├── manifest_test.go                  # Manifest module tests
│   │   ├── control_test.go                   # Control module tests
│   │   └── usage_test.go                     # Usage module tests
│   │
│   ├── integration/
│   │   ├── helpers_test.go                   # Integration test fixtures (conftest equivalent)
│   │   ├── e2e_flow_test.go                  # End-to-end: register -> execute -> verify
│   │   ├── acl_enforcement_test.go           # ACL + executor integration
│   │   ├── async_flows_test.go               # Async task flows
│   │   ├── binding_modules_test.go           # YAML binding -> execute
│   │   ├── decorator_modules_test.go         # Decorator module -> execute
│   │   ├── error_propagation_test.go         # Error flow through middleware chain
│   │   ├── full_lifecycle_test.go            # Register -> configure -> execute -> teardown
│   │   ├── infra_smoke_test.go               # Infrastructure smoke test
│   │   └── middleware_chain_test.go           # Multiple middleware composition
│   │
│   ├── conformance/
│   │   ├── conformance_test.go               # Protocol conformance tests
│   │   └── fixtures/                          # Conformance test fixtures (from protocol spec)
│   │
│   └── examples/
│       └── examples_test.go                  # Verify example code compiles and runs
│
└── examples/
    ├── simple_client.go                      # RUNNABLE: Create APCore client, register module via decorator, call sync, print result
    ├── global_client.go                      # RUNNABLE: Use package-level Call() convenience function
    ├── bindings/
    │   └── format_date/
    │       ├── format_date.go                # RUNNABLE: Target function for YAML binding
    │       └── binding.yaml                  # YAML binding config: module_id, target, schemas
    └── modules/
        ├── greet.go                          # RUNNABLE: Minimal struct-based module implementing Module interface
        ├── get_user.go                       # RUNNABLE: Readonly module with annotations (readonly, idempotent)
        ├── send_email.go                     # RUNNABLE: Full-featured module with annotations, examples, sensitive fields, ContextLogger
        └── decorated_add.go                  # RUNNABLE: Decorator-equivalent (ModuleFunc) module definition
```

### File Counts

| Category | Count |
|----------|-------|
| Source files (`.go`, top-level) | 16 |
| Source files (subdirectory packages) | 23 |
| **Total source files** | **39** |
| Test files (top-level) | 16 |
| Test files (middleware/) | 5 |
| Test files (registry/) | 9 |
| Test files (schema/) | 8 + fixtures |
| Test files (observability/) | 6 |
| Test files (events/) | 2 |
| Test files (sysmodules/) | 5 |
| Test files (integration/) | 10 |
| Test files (conformance/) | 1 |
| Test files (examples/) | 1 |
| **Total test files** | **63** |
| Example files | 8 (+ 1 YAML) |
| Config/meta files | 6 (go.mod, .gitignore, README, CHANGELOG, LICENSE, .code-forge.json) |
| **Grand total files** | **~117** |

### Test Stub Content Pattern

Each test file follows the Go standard testing convention with TDD red-phase stubs. Example for `tests/executor_test.go`:

```go
package apcore_test

import (
    "testing"
)

func TestExecute(t *testing.T) {
    t.Fatal("not implemented")
}

func TestExecuteStream(t *testing.T) {
    t.Fatal("not implemented")
}

func TestExecuteWithMiddleware(t *testing.T) {
    t.Fatal("not implemented")
}

func TestExecuteModuleNotFound(t *testing.T) {
    t.Fatal("not implemented")
}

func TestExecuteWithCancelToken(t *testing.T) {
    t.Fatal("not implemented")
}
```

### Example File Content Pattern

Each example is fully runnable. Example for `examples/simple_client.go`:

```go
package main

import (
    "fmt"
    "log"

    apcore "github.com/aipartnerup/apcore-go"
)

func main() {
    // 1. Initialize the APCore client
    client := apcore.NewAPCore()

    // 2. Define a module using the functional decorator equivalent
    addModule := apcore.ModuleFunc("math.add", "Add two integers",
        func(inputs map[string]any, ctx *apcore.Context) (map[string]any, error) {
            a := inputs["a"].(int)
            b := inputs["b"].(int)
            return map[string]any{"result": a + b}, nil
        },
    )

    // 3. Register the module
    client.Register("math.add", addModule)

    // 4. Call the module through the client
    result, err := client.Call("math.add", map[string]any{"a": 10, "b": 5})
    if err != nil {
        log.Fatalf("Error: %v", err)
    }
    fmt.Printf("Result: %v\n", result) // map[result:15]
}
```

### Helpers File Content Pattern

`tests/helpers_test.go` provides shared fixtures:

```go
package apcore_test

import (
    apcore "github.com/aipartnerup/apcore-go"
)

// newMockExecutor creates an Executor with an in-memory registry for testing.
func newMockExecutor() *apcore.Executor {
    // TODO: implement mock executor
    return nil
}

// sampleContext returns a Context with test identity and trace ID.
func sampleContext() *apcore.Context {
    // TODO: implement sample context
    return nil
}

// sampleModuleConfig returns a basic module descriptor for testing.
func sampleModuleConfig() map[string]any {
    // TODO: implement sample config
    return nil
}
```

---

## Step 5-6: Feature Specs and Code-Forge Config (not detailed here)

The skill would check for `apcore/docs/features/*.md`, link or generate feature specs, and write `.code-forge.json` pointing to the reference implementation and protocol spec.

---

## Step 7: Summary Output

```
apcore-skills:sdk -- SDK Bootstrap Complete

Target: /Users/tercel/WorkSpace/aipartnerup/apcore-go/
Language: go
Type: core
Modules: 39 source files scaffolded
Tests: 63 test stubs (TDD red phase)
Examples: 8 runnable examples (+ 1 YAML binding config)
Feature specs: (linked from apcore/docs/features/ or generated)
API contract: 108 public symbols to implement

Project structure:
  apcore-go/
  ├── go.mod, .gitignore, README.md, CHANGELOG.md, LICENSE
  ├── 16 top-level source files (apcore.go, executor.go, client.go, ...)
  ├── middleware/         5 files
  ├── registry/           5 files
  ├── schema/             6 files
  ├── observability/      5 files
  ├── events/             3 files
  ├── sysmodules/         5 files
  ├── utils/              4 files
  ├── tests/              63 test files mirroring source structure
  │   ├── 16 top-level unit test files
  │   ├── middleware/     5 tests
  │   ├── registry/       9 tests
  │   ├── schema/         8 tests + fixtures/
  │   ├── observability/  6 tests
  │   ├── events/         2 tests
  │   ├── sysmodules/     5 tests
  │   ├── integration/    10 tests
  │   ├── conformance/    1 test + fixtures/
  │   └── examples/       1 test
  └── examples/           8 runnable examples
      ├── simple_client.go, global_client.go
      ├── bindings/format_date/ (binding.yaml + format_date.go)
      └── modules/ (greet.go, get_user.go, send_email.go, decorated_add.go)

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-go/
  git init && git add . && git commit -m "chore: initialize apcore-go project skeleton"
  /code-forge:port @apcore --ref apcore-python --lang go        Generate implementation plans
  /code-forge:impl executor                                      Start implementing
  /apcore-skills:sync --lang go,python                           Verify API consistency
```

---

## Evaluation: Does the Skill Produce Proper Test Stubs and Examples?

### Test Stubs -- Assessment: YES, comprehensive

The skill instructions explicitly mandate:

1. **Per-module test stubs**: Step 4 lists test files for every source module (`test_executor`, `test_context`, `test_module`, `test_config`, `test_errors`, `test_acl`, `test_approval`, `test_async_task`, `test_bindings`, `test_decorator`, `test_cancel`, `test_trace_context`). This is 12 core test files matching 12+ core source files.

2. **Subdirectory test mirroring**: The skill explicitly lists `integration/`, `registry/`, `schema/`, `observability/` test subdirectories in the scaffold template. The "Reference Sync" section further instructs the sub-agent to "Read `{ref_path}/tests/` -- create corresponding test stubs for every test file" and "Preserve the subdirectory organization."

3. **TDD red-phase stubs**: The skill says each test must have "One failing stub test per public method: asserts `False` / `expect(false)` / `t.Fatal('not implemented')`". This is concrete and actionable.

4. **Helper/fixture file**: Explicitly required -- `{helpers-file}` with "shared fixtures: mock executor, sample context, sample module config."

5. **Integration test placeholder**: "Integration test directory with at least one placeholder test for end-to-end flow."

**What the reference actually has that the skill ALSO covers**: The reference has `tests/events/`, `tests/sys_modules/`, `tests/conformance/`, `tests/examples/` -- these are NOT listed in the Step 4 template but ARE covered by the "Reference Sync" instruction which says "create corresponding test stubs for every test file" in the reference. So the sub-agent should pick these up from reading the reference `tests/` directory.

**Potential gap**: The Step 4 template only explicitly lists `integration/`, `registry/`, `schema/`, `observability/` as test subdirectories. The `events/`, `sys_modules/`, `conformance/`, and `examples/` test subdirectories depend on the "Reference Sync" instruction being followed. This is a soft dependency -- a sub-agent that only follows the template and skips the reference sync step would miss these 4 subdirectories. The skill mitigates this with "If the reference has no examples/ or tests/ directory, fall back to the API contract to generate them from scratch" -- but it would be stronger to list ALL expected test subdirectories in the template.

### Examples -- Assessment: YES, comprehensive

The skill instructions explicitly require:

1. **simple_client example**: Listed in template with clear description: "basic client usage -- create executor, register module, execute."
2. **Binding examples**: Listed with YAML config + implementation.
3. **Module examples**: `greet`, `get_user`, `send_email` listed -- matching the reference exactly.
4. **Reference Sync**: "Read `{ref_path}/examples/` -- port each example file to {lang}, preserving the same usage patterns and directory structure."
5. **Runnable code, not stubs**: "Complete, runnable code (not stubs) that demonstrates one usage pattern."

**What the reference has that the skill also covers**: The reference has `simple_client.py`, `global_client.py`, `modules/{greet,get_user,send_email,decorated_add}.py`, `bindings/format_date/{binding.yaml, format_date.py}`. The skill template lists `simple_client`, binding examples, and three module examples (`greet`, `get_user`, `send_email`). The `global_client.py` and `decorated_add.py` would be picked up by the Reference Sync instruction.

**Potential gap**: `global_client` and `decorated_add` are not in the template but should be covered by reference sync. Same soft dependency as with tests.

### Summary

The skill produces comprehensive test stubs and examples. The key mechanism is two-layered:

1. **Template layer** (explicit file list): Covers the core cases -- 12 unit test files, 4 test subdirectories, 3 module examples, 1 simple client, 1 binding example.
2. **Reference Sync layer** (dynamic discovery): Covers additional files found in the reference that aren't in the template -- `events/`, `sys_modules/`, `conformance/`, `examples/` test dirs; `global_client`, `decorated_add` examples.

Both layers work together to produce a scaffold that mirrors the reference implementation's test and example coverage while using idiomatic Go patterns.
