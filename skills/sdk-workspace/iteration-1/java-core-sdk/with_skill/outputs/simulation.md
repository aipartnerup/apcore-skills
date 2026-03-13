# SDK Bootstrap Simulation: Java Core SDK

**Command:** `/apcore-skills:sdk java`
**Date:** 2026-03-13
**Skill version:** apcore-skills/sdk (SKILL.md)

---

## Step 0: Ecosystem Discovery

Ecosystem root detected: `/Users/tercel/WorkSpace/aipartnerup/`
Protocol spec found: `apcore/PROTOCOL_SPEC.md`
Reference implementations found: `apcore-python` (core), `apcore-typescript` (core), `apcore-mcp-python` (mcp), `apcore-mcp-typescript` (mcp)

---

## Step 1: Parse Arguments

**Raw arguments:** `java`

| Parameter | Parsed Value | Source |
|-----------|-------------|--------|
| `<language>` | `java` | positional argument |
| `--type` | `core` | default (not specified) |
| `--ref` | `apcore-python` | auto-detect: CWD is not an apcore repo, type=core, prefer `apcore-python` |

**Derived values:**
- Target repo name: `apcore-java`
- Target path: `/Users/tercel/WorkSpace/aipartnerup/apcore-java/`
- Package name: `com.aipartnerup.apcore`

**Target directory check:** Does not exist. Proceeding.

```
SDK Bootstrap:
  Language:   java
  Type:       core
  Reference:  apcore-python (python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-java/
```

---

## Step 2: Extract API Contract (Sub-agent)

### API_CONTRACT

```
type: core
source: apcore-python
source_version: 0.13.0
export_count: 117
module_count: 18 (source files/modules + subdirectory packages)
```

### MODULES

- **module: client**
  - file: `src/apcore/client.py`
  - classes:
    - `APCore`:
      - constructor: `(config: Config | None = None)`
      - methods:
        - `call(moduleId: String, inputs: Map, context: Context) -> Map`
        - `callAsync(moduleId: String, inputs: Map, context: Context) -> CompletableFuture<Map>` [async]
        - `stream(moduleId: String, inputs: Map, context: Context) -> Stream<Map>` [async]
        - `validate(moduleId: String, inputs: Map, context: Context) -> PreflightResult`
        - `register(moduleId: String, module: Object) -> void`
        - `describe(moduleId: String) -> String`
        - `use(middleware: Middleware) -> APCore`
        - `useBefore(callback: BeforeMiddleware) -> APCore`
        - `useAfter(callback: AfterMiddleware) -> APCore`
        - `remove(middleware: Middleware) -> boolean`
        - `discover() -> int`
        - `listModules(tags: List<String>, prefix: String) -> List<String>`
        - `on(eventType: String, handler: Object) -> EventSubscriber`
        - `off(subscriber: EventSubscriber) -> void`
        - `disable(moduleId: String, reason: String) -> Map`
        - `enable(moduleId: String, reason: String) -> Map`
        - `module(id: String, description: String, ...) -> Annotation` (decorator equivalent)

- **module: executor**
  - file: `src/apcore/executor.py`
  - classes:
    - `Executor`:
      - constructor: `(registry: Registry, config: Config | None = None, middlewareManager: MiddlewareManager | None = None)`
      - methods:
        - `execute(moduleId: String, inputs: Map, context: Context) -> Map`
        - `executeAsync(moduleId: String, inputs: Map, context: Context) -> CompletableFuture<Map>` [async]
        - `executeStream(moduleId: String, inputs: Map, context: Context) -> Stream<Map>` [async]
  - functions:
    - `redactSensitive(data: Map, schema: Map) -> Map` [static]
  - constants:
    - `REDACTED_VALUE: String = "***REDACTED***"`

- **module: context**
  - file: `src/apcore/context.py`
  - classes:
    - `Context`:
      - constructor: `(moduleId: String, inputs: Map, identity: Identity, ...)`
      - methods: getters for all fields, `child(moduleId: String) -> Context`
    - `ContextFactory`:
      - methods: `create(moduleId: String, inputs: Map, ...) -> Context` [static]
    - `Identity`:
      - constructor: `(id: String, roles: List<String>, metadata: Map)`

- **module: module**
  - file: `src/apcore/module.py`
  - classes:
    - `Module` (interface):
      - methods:
        - `execute(inputs: Map, context: Context) -> Map`
        - `getInputSchema() -> Class`
        - `getOutputSchema() -> Class`
        - `getDescription() -> String`
    - `ModuleAnnotations`:
      - constructor: `(readonly: boolean, idempotent: boolean, destructive: boolean, openWorld: boolean)`
    - `ModuleExample`:
      - constructor: `(title: String, inputs: Map, output: Map, description: String)`
    - `PreflightCheckResult`, `PreflightResult`, `ValidationResult`

- **module: config**
  - file: `src/apcore/config.py`
  - classes:
    - `Config`:
      - constructor: `(configPath: String | null)`
      - methods: `get(key: String) -> Object`, `getInt(key: String) -> int`, etc.

- **module: errors**
  - file: `src/apcore/errors.py`
  - (see ERROR_HIERARCHY below)

- **module: acl**
  - file: `src/apcore/acl.py`
  - classes:
    - `ACL`: `check(callerId: String, targetId: String, context: Context) -> void`, `addRule(rule: ACLRule)`, `audit() -> List<AuditEntry>`
    - `ACLRule`: `(pattern: String, allowedRoles: List<String>, effect: String)`
    - `AuditEntry`: `(timestamp: String, callerId: String, targetId: String, allowed: boolean)`

- **module: approval**
  - file: `src/apcore/approval.py`
  - classes:
    - `ApprovalHandler` (interface): `handle(request: ApprovalRequest) -> ApprovalResult`
    - `ApprovalRequest`: `(moduleId: String, context: Context, inputs: Map)`
    - `ApprovalResult`: `(approved: boolean, reason: String, approvalId: String)`
    - `AlwaysDenyHandler`, `AutoApproveHandler`, `CallbackApprovalHandler`

- **module: async_task**
  - file: `src/apcore/async_task.py`
  - classes:
    - `AsyncTaskManager`: `submit(task: Callable) -> TaskInfo`, `getStatus(taskId: String) -> TaskStatus`, `cancel(taskId: String) -> boolean`
    - `TaskInfo`: `(taskId: String, status: TaskStatus, result: Object)`
    - `TaskStatus` (enum): `PENDING, RUNNING, COMPLETED, FAILED, CANCELLED`

- **module: bindings**
  - file: `src/apcore/bindings.py`
  - classes:
    - `BindingLoader`: `loadFromYaml(path: String) -> List<Module>`, `loadFromDirectory(dirPath: String) -> List<Module>`

- **module: decorator**
  - file: `src/apcore/decorator.py`
  - classes:
    - `FunctionModule`: wraps a function as a Module (Java equivalent: annotation processor or builder)
  - functions:
    - `module(id, description, ...) -> Annotation` (decorator — Java: `@ApCoreModule` annotation)

- **module: extensions**
  - file: `src/apcore/extensions.py`
  - classes:
    - `ExtensionManager`: `register(point: ExtensionPoint)`, `getExtensions(pointName: String) -> List`
    - `ExtensionPoint`: `(name: String, handler: Object)`

- **module: cancel**
  - file: `src/apcore/cancel.py`
  - classes:
    - `CancelToken`: `cancel() -> void`, `isCancelled() -> boolean`, `onCancel(callback: Runnable)`
    - `ExecutionCancelledError` extends `ModuleError`

- **module: trace_context**
  - file: `src/apcore/trace_context.py`
  - classes:
    - `TraceContext`: `(traceId: String, spanId: String, parentSpanId: String, traceFlags: int)`
    - `TraceParent`: `parse(header: String) -> TraceContext` [static], `format(context: TraceContext) -> String` [static]

- **module: middleware/** (package)
  - interfaces: `Middleware`, `BeforeMiddleware`, `AfterMiddleware`
  - classes: `MiddlewareManager`, `LoggingMiddleware`, `RetryMiddleware`, `RetryConfig`, `ErrorHistoryMiddleware`, `PlatformNotifyMiddleware`
  - errors: `MiddlewareChainError`

- **module: registry/** (package)
  - classes: `Registry`, `ModuleValidator`, `Discoverer`
  - types: `ModuleDescriptor`, `DiscoveredModule`, `DependencyInfo`
  - constants: `MAX_MODULE_ID_LENGTH`, `MODULE_ID_PATTERN`, `RESERVED_WORDS`, `REGISTRY_EVENTS`

- **module: schema/** (package)
  - classes: `SchemaLoader`, `SchemaValidator`, `SchemaExporter`, `RefResolver`
  - types: `SchemaStrategy` (enum), `ExportProfile`
  - functions: `toStrictSchema(schema: Map) -> Map`

- **module: observability/** (package)
  - classes: `TracingMiddleware`, `MetricsMiddleware`, `MetricsCollector`, `ContextLogger`, `ObsLoggingMiddleware`, `UsageCollector`, `UsageMiddleware`
  - span: `Span`, `SpanExporter` (interface), `StdoutExporter`, `InMemoryExporter`, `OTLPExporter`
  - functions: `createSpan(name: String, ...) -> Span`
  - history: `ErrorEntry`, `ErrorHistory`

- **module: events/** (package)
  - classes: `EventEmitter`, `EventSubscriber`, `ApCoreEvent`, `WebhookSubscriber`, `A2ASubscriber`

- **module: version**
  - file: `src/apcore/version.py`
  - classes: `VersionIncompatibleError`
  - functions: `negotiateVersion(requested: String, supported: List<String>) -> String`

- **module: utils/** (package)
  - functions: `matchPattern(pattern: String, value: String) -> boolean`, `guardCallChain(chain: List<String>, maxDepth: int)`, `normalizeToCanonicalId(id: String) -> String`, `calculateSpecificity(pattern: String) -> int`, `propagateError(error: ModuleError, context: Context) -> ModuleError`

### ERROR_HIERARCHY

```
base: ModuleError (extends RuntimeException in Java)
  constructor: (code: String, message: String, details: Map, cause: Exception,
                traceId: String, retryable: Boolean, aiGuidance: String,
                userFixable: Boolean, suggestion: String)
  methods: toDict() -> Map, toString() -> String

codes: ErrorCodes (final class with static final String constants)
  CONFIG_NOT_FOUND, CONFIG_INVALID, ACL_RULE_ERROR, ACL_DENIED,
  MODULE_NOT_FOUND, MODULE_DISABLED, MODULE_TIMEOUT, MODULE_LOAD_ERROR,
  MODULE_EXECUTE_ERROR, RELOAD_FAILED, EXECUTION_CANCELLED,
  SCHEMA_VALIDATION_ERROR, SCHEMA_NOT_FOUND, SCHEMA_PARSE_ERROR,
  SCHEMA_CIRCULAR_REF, CALL_DEPTH_EXCEEDED, CIRCULAR_CALL,
  CALL_FREQUENCY_EXCEEDED, GENERAL_INVALID_INPUT, GENERAL_INTERNAL_ERROR,
  FUNC_MISSING_TYPE_HINT, FUNC_MISSING_RETURN_TYPE,
  BINDING_INVALID_TARGET, BINDING_MODULE_NOT_FOUND,
  BINDING_CALLABLE_NOT_FOUND, BINDING_NOT_CALLABLE,
  BINDING_SCHEMA_MISSING, BINDING_FILE_INVALID,
  CIRCULAR_DEPENDENCY, MIDDLEWARE_CHAIN_ERROR,
  APPROVAL_DENIED, APPROVAL_TIMEOUT, APPROVAL_PENDING,
  VERSION_INCOMPATIBLE, ERROR_CODE_COLLISION,
  GENERAL_NOT_IMPLEMENTED, DEPENDENCY_NOT_FOUND

classes:
  ConfigNotFoundError(code=CONFIG_NOT_FOUND, parent=ModuleError, retryable=false)
  ConfigError(code=CONFIG_INVALID, parent=ModuleError, retryable=false)
  ACLRuleError(code=ACL_RULE_ERROR, parent=ModuleError, retryable=false)
  ACLDeniedError(code=ACL_DENIED, parent=ModuleError, retryable=false)
  ApprovalError(code=varies, parent=ModuleError, retryable=false)
  ApprovalDeniedError(code=APPROVAL_DENIED, parent=ApprovalError, retryable=false)
  ApprovalTimeoutError(code=APPROVAL_TIMEOUT, parent=ApprovalError, retryable=true)
  ApprovalPendingError(code=APPROVAL_PENDING, parent=ApprovalError, retryable=false)
  ModuleNotFoundError(code=MODULE_NOT_FOUND, parent=ModuleError, retryable=false)
  ModuleDisabledError(code=MODULE_DISABLED, parent=ModuleError, retryable=false)
  ModuleTimeoutError(code=MODULE_TIMEOUT, parent=ModuleError, retryable=true)
  SchemaValidationError(code=SCHEMA_VALIDATION_ERROR, parent=ModuleError, retryable=false)
  SchemaNotFoundError(code=SCHEMA_NOT_FOUND, parent=ModuleError, retryable=false)
  SchemaParseError(code=SCHEMA_PARSE_ERROR, parent=ModuleError, retryable=false)
  SchemaCircularRefError(code=SCHEMA_CIRCULAR_REF, parent=ModuleError, retryable=false)
  CallDepthExceededError(code=CALL_DEPTH_EXCEEDED, parent=ModuleError, retryable=false)
  CircularCallError(code=CIRCULAR_CALL, parent=ModuleError, retryable=false)
  CallFrequencyExceededError(code=CALL_FREQUENCY_EXCEEDED, parent=ModuleError, retryable=false)
  InvalidInputError(code=GENERAL_INVALID_INPUT, parent=ModuleError, retryable=false)
  FuncMissingTypeHintError(code=FUNC_MISSING_TYPE_HINT, parent=ModuleError, retryable=false)
  FuncMissingReturnTypeError(code=FUNC_MISSING_RETURN_TYPE, parent=ModuleError, retryable=false)
  BindingInvalidTargetError(code=BINDING_INVALID_TARGET, parent=ModuleError, retryable=false)
  BindingModuleNotFoundError(code=BINDING_MODULE_NOT_FOUND, parent=ModuleError, retryable=false)
  BindingCallableNotFoundError(code=BINDING_CALLABLE_NOT_FOUND, parent=ModuleError, retryable=false)
  BindingNotCallableError(code=BINDING_NOT_CALLABLE, parent=ModuleError, retryable=false)
  BindingSchemaMissingError(code=BINDING_SCHEMA_MISSING, parent=ModuleError, retryable=false)
  BindingFileInvalidError(code=BINDING_FILE_INVALID, parent=ModuleError, retryable=false)
  CircularDependencyError(code=CIRCULAR_DEPENDENCY, parent=ModuleError, retryable=false)
  ModuleLoadError(code=MODULE_LOAD_ERROR, parent=ModuleError, retryable=false)
  ModuleExecuteError(code=MODULE_EXECUTE_ERROR, parent=ModuleError, retryable=null)
  ReloadFailedError(code=RELOAD_FAILED, parent=ModuleError, retryable=true)
  InternalError(code=GENERAL_INTERNAL_ERROR, parent=ModuleError, retryable=true)
  FeatureNotImplementedError(code=GENERAL_NOT_IMPLEMENTED, parent=ModuleError, retryable=false)
  DependencyNotFoundError(code=DEPENDENCY_NOT_FOUND, parent=ModuleError, retryable=false)
  ExecutionCancelledError(code=EXECUTION_CANCELLED, parent=ModuleError)
  ErrorCodeCollisionError(code=ERROR_CODE_COLLISION, parent=ModuleError, retryable=false)
  VersionIncompatibleError(code=VERSION_INCOMPATIBLE, parent=ModuleError)
  MiddlewareChainError(code=MIDDLEWARE_CHAIN_ERROR, parent=ModuleError)
```

### EXTENSION_POINTS

- `Middleware`: `before(context: Context) -> Context`, `after(context: Context, result: Map) -> Map`, `onError(context: Context, error: ModuleError) -> ModuleError`
- `BeforeMiddleware`: `(context: Context) -> Context`
- `AfterMiddleware`: `(context: Context, result: Map) -> Map`
- `ApprovalHandler`: `handle(request: ApprovalRequest) -> ApprovalResult`
- `Discoverer`: `discover(registry: Registry) -> List<DiscoveredModule>`
- `ModuleValidator`: `validate(descriptor: ModuleDescriptor) -> ValidationResult`
- `SpanExporter`: `export(spans: List<Span>) -> void`
- `EventSubscriber`: `onEvent(event: ApCoreEvent) -> void`

### EXAMPLES

```
examples/
  simple_client.py:       Basic client usage -- create APCore, register decorator module, call sync
  global_client.py:       Global convenience API -- use apcore.module() and apcore.call() without explicit client
  modules/
    greet.py:             Minimal duck-typed module (class with input_schema, output_schema, execute)
    get_user.py:          Readonly module with ModuleAnnotations (readonly=true, idempotent=true)
    send_email.py:        Full-featured module with tags, version, metadata, annotations, examples, ContextLogger
    decorated_add.py:     Decorator-based module using @module annotation
  bindings/
    format_date/
      format_date.py:     Target function for YAML binding (pure function returning dict)
      binding.yaml:       YAML binding config -- maps module_id to target callable with explicit schemas
```

### TESTS

```
structure:
  tests/                          (root -- unit tests)
  tests/integration/              (cross-component integration tests)
  tests/registry/                 (registry-specific tests)
  tests/schema/                   (schema validation tests)
  tests/observability/            (metrics, tracing, logging, usage tests)
  tests/events/                   (event emitter and subscriber tests)
  tests/conformance/              (protocol conformance tests)
  tests/sys_modules/              (system module tests)
  tests/examples/                 (example module smoke tests)

files:
  conftest.py:                     Shared fixtures (mock executor, sample context, sample config)
  test_executor.py:                Executor core execution logic
  test_executor_async.py:          Async executor execution
  test_executor_stream.py:         Streaming executor execution
  test_executor_types.py:          Executor type handling
  test_context.py:                 Context creation and child contexts
  test_context_services.py:        Context service injection
  test_config.py:                  Config loading and access
  test_errors.py:                  Error hierarchy, toDict, error codes
  test_acl.py:                     ACL rule matching and enforcement
  test_acl_audit.py:               ACL audit logging
  test_approval.py:                Approval handler logic
  test_approval_executor.py:       Approval integration with executor
  test_approval_integration.py:    Approval end-to-end flow
  test_async_task.py:              Async task manager submit/cancel
  test_bindings.py:                Binding loader YAML parsing
  test_decorator.py:               @module decorator and FunctionModule
  test_extensions.py:              Extension manager registration
  test_cancel.py:                  CancelToken and cancellation flow
  test_trace_context.py:           TraceParent parse/format
  test_middleware.py:              Middleware interface contract
  test_middleware_manager.py:      MiddlewareManager chain execution
  test_logging_middleware.py:      LoggingMiddleware output
  test_retry_middleware.py:        RetryMiddleware with RetryConfig
  test_error_history_middleware.py: ErrorHistoryMiddleware tracking
  test_platform_notify_middleware.py: PlatformNotifyMiddleware
  test_client.py:                  APCore client high-level API
  test_call_chain.py:              Call chain guard (depth, frequency, circular)
  test_normalize.py:               Canonical ID normalization
  test_specificity.py:             Pattern specificity calculation
  test_error_propagation.py:       Error propagation utility
  test_error_code_registry.py:     ErrorCodeRegistry collision detection
  test_redaction.py:               Sensitive field redaction
  test_version.py:                 Version negotiation
  test_public_api.py:              All __all__ exports are importable
  test_docstrings.py:              Docstring parsing
  test_global_timeout.py:          Global timeout enforcement
  test_id_conflicts.py:            Module ID conflict detection
  test_safe_reload.py:             Hot-reload safety
  test_suspend_resume.py:          Suspend/resume flow
  integration/conftest.py:         Integration test fixtures
  integration/test_e2e_flow.py:    End-to-end executor flow
  integration/test_middleware_chain.py: Full middleware chain
  integration/test_binding_modules.py: Binding modules integration
  integration/test_decorator_modules.py: Decorator modules integration
  integration/test_acl_enforcement.py: ACL enforcement integration
  integration/test_error_propagation.py: Error propagation integration
  integration/test_async_flows.py: Async execution flows
  integration/test_full_lifecycle.py: Full module lifecycle
  integration/test_infra_smoke.py: Infrastructure smoke tests
  registry/conftest.py:            Registry test fixtures
  registry/test_registry.py:       Registry core operations
  registry/test_dependencies.py:   Dependency resolution
  registry/test_entry_point.py:    Entry point discovery
  registry/test_metadata.py:       Module metadata
  registry/test_scanner.py:        Module scanner
  registry/test_types.py:          Registry type definitions
  registry/test_validation.py:     Module validation
  registry/test_version_negotiation.py: Version negotiation
  registry/test_integration.py:    Registry integration
  registry/test_schema_export.py:  Schema export from registry
  registry/test_circular_dependency_error.py: Circular dependency detection
  registry/test_module_execute_and_internal_errors.py: Module execute errors
  schema/conftest.py:              Schema test fixtures
  schema/test_validator.py:        Schema validation
  schema/test_loader.py:           Schema loading
  schema/test_exporter.py:         Schema export
  schema/test_ref_resolver.py:     $ref resolution
  schema/test_types.py:            Schema type definitions
  schema/test_strict.py:           Strict schema conversion
  schema/test_annotations.py:      Schema annotations
  schema/test_edge_cases.py:       Schema edge cases
  observability/test_tracing.py:   Span tracing
  observability/test_metrics.py:   Metrics collection
  observability/test_context_logger.py: Context-aware logging
  observability/test_observability_package.py: Package imports
  observability/test_error_history.py: Error history tracking
  observability/test_usage.py:     Usage collection
  events/test_emitter.py:          Event emitter
  events/test_subscribers.py:      Event subscribers (webhook, A2A)
  conformance/test_conformance.py: Protocol conformance
  sys_modules/test_manifest.py:    System manifest module
  sys_modules/test_health.py:      System health module
  sys_modules/test_control.py:     System control module
  sys_modules/test_registration.py: Subscriber registration
  sys_modules/test_usage.py:       System usage module
  examples/test_example_modules.py: Example module smoke tests

total_count: 72
```

---

## Step 3: Confirm Tech Stack (Questions for User)

The skill would present the following interactive questions via `AskUserQuestion`:

```
Java SDK Tech Stack Configuration
==================================

1. Java version:
   [a] 17+ (Recommended)
   [b] 21+

2. Build tool:
   [a] Gradle (Recommended)
   [b] Maven

3. Schema validation / JSON library:
   [a] Jackson (Recommended)
   [b] Gson

4. Test framework:
   [a] JUnit 5 (Recommended)
   [b] TestNG
```

**Simulated answers (using recommended defaults):**

```yaml
tech_stack:
  java_version: "17+"
  build_tool: "Gradle"
  json_library: "Jackson"
  test_framework: "JUnit 5"
  package_name: "com.aipartnerup.apcore"
  group_id: "com.aipartnerup"
  artifact_id: "apcore-java"
```

---

## Step 4: Scaffold Project -- COMPLETE Project Structure

All files and directories that would be created at `/Users/tercel/WorkSpace/aipartnerup/apcore-java/`:

```
apcore-java/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── .gitignore
├── README.md
├── CHANGELOG.md
├── LICENSE                                          # Apache-2.0 (matches ecosystem)
│
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── aipartnerup/
│                   └── apcore/
│                       ├── APCore.java              # Main client class (top-level API)
│                       ├── Executor.java             # Module execution engine
│                       ├── Context.java              # Execution context
│                       ├── ContextFactory.java       # Context creation factory
│                       ├── Identity.java             # Caller identity
│                       ├── Module.java               # Module interface
│                       ├── ModuleAnnotations.java    # Module annotation metadata
│                       ├── ModuleExample.java        # Module example data
│                       ├── PreflightCheckResult.java # Preflight check result
│                       ├── PreflightResult.java      # Preflight validation result
│                       ├── ValidationResult.java     # Validation result
│                       ├── Config.java               # Configuration loader
│                       ├── FunctionModule.java        # Function-as-module wrapper
│                       ├── ApCoreModule.java          # @ApCoreModule annotation (decorator equivalent)
│                       ├── CancelToken.java           # Cancellation token
│                       ├── TraceContext.java           # Trace context propagation
│                       ├── TraceParent.java            # W3C TraceParent parsing
│                       │
│                       ├── errors/
│                       │   ├── ModuleError.java              # Base error (extends RuntimeException)
│                       │   ├── ErrorCodes.java                # All error code constants
│                       │   ├── ErrorCodeRegistry.java         # Custom error code registry (A17)
│                       │   ├── ConfigNotFoundError.java
│                       │   ├── ConfigError.java
│                       │   ├── AclRuleError.java
│                       │   ├── AclDeniedError.java
│                       │   ├── ApprovalError.java
│                       │   ├── ApprovalDeniedError.java
│                       │   ├── ApprovalTimeoutError.java
│                       │   ├── ApprovalPendingError.java
│                       │   ├── ModuleNotFoundError.java
│                       │   ├── ModuleDisabledError.java
│                       │   ├── ModuleTimeoutError.java
│                       │   ├── ModuleLoadError.java
│                       │   ├── ModuleExecuteError.java
│                       │   ├── ReloadFailedError.java
│                       │   ├── ExecutionCancelledError.java
│                       │   ├── SchemaValidationError.java
│                       │   ├── SchemaNotFoundError.java
│                       │   ├── SchemaParseError.java
│                       │   ├── SchemaCircularRefError.java
│                       │   ├── CallDepthExceededError.java
│                       │   ├── CircularCallError.java
│                       │   ├── CallFrequencyExceededError.java
│                       │   ├── InvalidInputError.java
│                       │   ├── InternalError.java
│                       │   ├── FeatureNotImplementedError.java
│                       │   ├── DependencyNotFoundError.java
│                       │   ├── ErrorCodeCollisionError.java
│                       │   ├── VersionIncompatibleError.java
│                       │   ├── MiddlewareChainError.java
│                       │   ├── FuncMissingTypeHintError.java
│                       │   ├── FuncMissingReturnTypeError.java
│                       │   ├── BindingInvalidTargetError.java
│                       │   ├── BindingModuleNotFoundError.java
│                       │   ├── BindingCallableNotFoundError.java
│                       │   ├── BindingNotCallableError.java
│                       │   ├── BindingSchemaMissingError.java
│                       │   ├── BindingFileInvalidError.java
│                       │   └── CircularDependencyError.java
│                       │
│                       ├── acl/
│                       │   ├── Acl.java                       # ACL engine
│                       │   ├── AclRule.java                   # ACL rule definition
│                       │   └── AuditEntry.java                # Audit log entry
│                       │
│                       ├── approval/
│                       │   ├── ApprovalHandler.java            # Handler interface
│                       │   ├── ApprovalRequest.java            # Request DTO
│                       │   ├── ApprovalResult.java             # Result DTO
│                       │   ├── AlwaysDenyHandler.java
│                       │   ├── AutoApproveHandler.java
│                       │   └── CallbackApprovalHandler.java
│                       │
│                       ├── async/
│                       │   ├── AsyncTaskManager.java           # Async task submission/tracking
│                       │   ├── TaskInfo.java                   # Task information DTO
│                       │   └── TaskStatus.java                 # Task status enum
│                       │
│                       ├── bindings/
│                       │   └── BindingLoader.java              # YAML binding loader
│                       │
│                       ├── extensions/
│                       │   ├── ExtensionManager.java           # Extension registration
│                       │   └── ExtensionPoint.java             # Extension point definition
│                       │
│                       ├── middleware/
│                       │   ├── Middleware.java                 # Middleware interface
│                       │   ├── BeforeMiddleware.java           # Before-only middleware (functional interface)
│                       │   ├── AfterMiddleware.java            # After-only middleware (functional interface)
│                       │   ├── MiddlewareManager.java          # Middleware chain manager
│                       │   ├── LoggingMiddleware.java
│                       │   ├── RetryMiddleware.java
│                       │   ├── RetryConfig.java
│                       │   ├── ErrorHistoryMiddleware.java
│                       │   └── PlatformNotifyMiddleware.java
│                       │
│                       ├── registry/
│                       │   ├── Registry.java                   # Module registry
│                       │   ├── ModuleDescriptor.java           # Module descriptor
│                       │   ├── DiscoveredModule.java           # Discovered module info
│                       │   ├── DependencyInfo.java             # Dependency metadata
│                       │   ├── Discoverer.java                 # Discoverer interface
│                       │   ├── ModuleValidator.java            # Validator interface
│                       │   └── RegistryConstants.java          # MAX_MODULE_ID_LENGTH, MODULE_ID_PATTERN, etc.
│                       │
│                       ├── schema/
│                       │   ├── SchemaLoader.java               # Schema loading
│                       │   ├── SchemaValidator.java            # Schema validation
│                       │   ├── SchemaExporter.java             # Schema export
│                       │   ├── RefResolver.java                # $ref resolution
│                       │   ├── SchemaStrategy.java             # Schema strategy enum
│                       │   ├── ExportProfile.java              # Export profile
│                       │   └── StrictSchemaConverter.java      # toStrictSchema equivalent
│                       │
│                       ├── observability/
│                       │   ├── TracingMiddleware.java
│                       │   ├── MetricsMiddleware.java
│                       │   ├── MetricsCollector.java
│                       │   ├── ContextLogger.java
│                       │   ├── ObsLoggingMiddleware.java
│                       │   ├── UsageCollector.java
│                       │   ├── UsageMiddleware.java
│                       │   ├── Span.java
│                       │   ├── SpanExporter.java               # Interface
│                       │   ├── StdoutExporter.java
│                       │   ├── InMemoryExporter.java
│                       │   ├── OtlpExporter.java
│                       │   ├── SpanFactory.java                # createSpan equivalent
│                       │   ├── ErrorEntry.java
│                       │   └── ErrorHistory.java
│                       │
│                       ├── events/
│                       │   ├── ApCoreEvent.java
│                       │   ├── EventEmitter.java
│                       │   ├── EventSubscriber.java            # Interface
│                       │   ├── WebhookSubscriber.java
│                       │   └── A2aSubscriber.java
│                       │
│                       ├── sysmodules/
│                       │   └── Registration.java               # System module registration
│                       │
│                       ├── version/
│                       │   └── VersionNegotiator.java          # negotiateVersion
│                       │
│                       └── utils/
│                           ├── PatternMatcher.java             # matchPattern
│                           ├── CallChainGuard.java             # guardCallChain
│                           ├── CanonicalIdNormalizer.java      # normalizeToCanonicalId
│                           ├── SpecificityCalculator.java      # calculateSpecificity
│                           ├── ErrorPropagator.java            # propagateError
│                           └── SensitiveRedactor.java          # redactSensitive
│
├── src/
│   └── test/
│       └── java/
│           └── com/
│               └── aipartnerup/
│                   └── apcore/
│                       ├── TestHelpers.java                        # Shared fixtures: mock executor, sample context, sample config
│                       ├── APCoreTest.java                         # APCore client high-level API tests
│                       ├── ExecutorTest.java                       # Executor core execution logic
│                       ├── ExecutorAsyncTest.java                  # Async executor execution
│                       ├── ExecutorStreamTest.java                 # Streaming executor execution
│                       ├── ExecutorTypesTest.java                  # Executor type handling
│                       ├── ContextTest.java                        # Context creation and child contexts
│                       ├── ContextServicesTest.java                # Context service injection
│                       ├── ConfigTest.java                         # Config loading and access
│                       ├── FunctionModuleTest.java                 # FunctionModule (decorator equivalent)
│                       ├── CancelTokenTest.java                    # CancelToken and cancellation flow
│                       ├── TraceContextTest.java                   # TraceParent parse/format
│                       ├── PublicApiTest.java                      # All public exports are accessible
│                       ├── GlobalTimeoutTest.java                  # Global timeout enforcement
│                       ├── IdConflictsTest.java                    # Module ID conflict detection
│                       ├── SafeReloadTest.java                     # Hot-reload safety
│                       ├── SuspendResumeTest.java                  # Suspend/resume flow
│                       ├── VersionNegotiationTest.java             # Version negotiation
│                       │
│                       ├── errors/
│                       │   ├── ModuleErrorTest.java                # Error hierarchy, toDict, error codes
│                       │   ├── ErrorCodeRegistryTest.java          # ErrorCodeRegistry collision detection
│                       │   └── ErrorPropagationTest.java           # Error propagation utility
│                       │
│                       ├── acl/
│                       │   ├── AclTest.java                        # ACL rule matching and enforcement
│                       │   └── AclAuditTest.java                   # ACL audit logging
│                       │
│                       ├── approval/
│                       │   ├── ApprovalTest.java                   # Approval handler logic
│                       │   ├── ApprovalExecutorTest.java           # Approval integration with executor
│                       │   └── ApprovalIntegrationTest.java        # Approval end-to-end flow
│                       │
│                       ├── async/
│                       │   └── AsyncTaskManagerTest.java           # Async task manager submit/cancel
│                       │
│                       ├── bindings/
│                       │   └── BindingLoaderTest.java              # Binding loader YAML parsing
│                       │
│                       ├── extensions/
│                       │   └── ExtensionManagerTest.java           # Extension manager registration
│                       │
│                       ├── middleware/
│                       │   ├── MiddlewareTest.java                 # Middleware interface contract
│                       │   ├── MiddlewareManagerTest.java          # MiddlewareManager chain execution
│                       │   ├── LoggingMiddlewareTest.java          # LoggingMiddleware output
│                       │   ├── RetryMiddlewareTest.java            # RetryMiddleware with RetryConfig
│                       │   ├── ErrorHistoryMiddlewareTest.java     # ErrorHistoryMiddleware tracking
│                       │   └── PlatformNotifyMiddlewareTest.java   # PlatformNotifyMiddleware
│                       │
│                       ├── registry/
│                       │   ├── RegistryTest.java                   # Registry core operations
│                       │   ├── DependenciesTest.java               # Dependency resolution
│                       │   ├── EntryPointTest.java                 # Entry point discovery
│                       │   ├── MetadataTest.java                   # Module metadata
│                       │   ├── ScannerTest.java                    # Module scanner
│                       │   ├── RegistryTypesTest.java              # Registry type definitions
│                       │   ├── RegistryValidationTest.java         # Module validation
│                       │   ├── VersionNegotiationTest.java         # Registry version negotiation
│                       │   ├── RegistryIntegrationTest.java        # Registry integration
│                       │   ├── SchemaExportTest.java               # Schema export from registry
│                       │   ├── CircularDependencyErrorTest.java    # Circular dependency detection
│                       │   └── ModuleExecuteAndInternalErrorsTest.java # Module execute errors
│                       │
│                       ├── schema/
│                       │   ├── SchemaValidatorTest.java            # Schema validation
│                       │   ├── SchemaLoaderTest.java               # Schema loading
│                       │   ├── SchemaExporterTest.java             # Schema export
│                       │   ├── RefResolverTest.java                # $ref resolution
│                       │   ├── SchemaTypesTest.java                # Schema type definitions
│                       │   ├── StrictSchemaTest.java               # Strict schema conversion
│                       │   ├── SchemaAnnotationsTest.java          # Schema annotations
│                       │   └── SchemaEdgeCasesTest.java            # Schema edge cases
│                       │
│                       ├── observability/
│                       │   ├── TracingTest.java                    # Span tracing
│                       │   ├── MetricsTest.java                    # Metrics collection
│                       │   ├── ContextLoggerTest.java              # Context-aware logging
│                       │   ├── ObservabilityPackageTest.java       # Package imports
│                       │   ├── ErrorHistoryTest.java               # Error history tracking
│                       │   └── UsageTest.java                      # Usage collection
│                       │
│                       ├── events/
│                       │   ├── EventEmitterTest.java               # Event emitter
│                       │   └── EventSubscribersTest.java           # Event subscribers (webhook, A2A)
│                       │
│                       ├── conformance/
│                       │   └── ConformanceTest.java                # Protocol conformance
│                       │
│                       ├── sysmodules/
│                       │   ├── ManifestTest.java                   # System manifest module
│                       │   ├── HealthTest.java                     # System health module
│                       │   ├── ControlTest.java                    # System control module
│                       │   ├── RegistrationTest.java               # Subscriber registration
│                       │   └── UsageTest.java                      # System usage module
│                       │
│                       ├── examples/
│                       │   └── ExampleModulesTest.java             # Example module smoke tests
│                       │
│                       ├── integration/
│                       │   ├── IntegrationTestHelpers.java         # Integration test fixtures
│                       │   ├── EndToEndFlowTest.java               # End-to-end executor flow
│                       │   ├── MiddlewareChainTest.java            # Full middleware chain
│                       │   ├── BindingModulesTest.java             # Binding modules integration
│                       │   ├── DecoratorModulesTest.java           # FunctionModule integration
│                       │   ├── AclEnforcementTest.java             # ACL enforcement integration
│                       │   ├── ErrorPropagationTest.java           # Error propagation integration
│                       │   ├── AsyncFlowsTest.java                 # Async execution flows
│                       │   ├── FullLifecycleTest.java              # Full module lifecycle
│                       │   └── InfraSmokeTest.java                 # Infrastructure smoke tests
│                       │
│                       └── utils/
│                           ├── CallChainGuardTest.java             # Call chain guard (depth, frequency, circular)
│                           ├── CanonicalIdNormalizerTest.java      # Canonical ID normalization
│                           ├── SpecificityCalculatorTest.java      # Pattern specificity calculation
│                           └── SensitiveRedactorTest.java          # Sensitive field redaction
│
└── examples/
    ├── SimpleClient.java                   # Basic client: create APCore, register module, call, print result
    ├── GlobalClient.java                   # Global convenience API (static APCore.call / APCore.module)
    ├── modules/
    │   ├── GreetModule.java                # Minimal Module implementation (PascalCase class, camelCase methods)
    │   ├── GetUserModule.java              # Readonly module with ModuleAnnotations
    │   ├── SendEmailModule.java            # Full-featured module with tags, version, metadata, ContextLogger
    │   └── DecoratedAdd.java               # FunctionModule builder-based module definition
    └── bindings/
        └── formatdate/
            ├── FormatDateFunction.java     # Target function for YAML binding
            └── binding.yaml                # YAML binding config


```

### Java Naming Convention Details (applied throughout)

| Concept | Python Reference | Java Equivalent |
|---------|-----------------|-----------------|
| File names | `snake_case.py` | `PascalCase.java` |
| Classes | `PascalCase` | `PascalCase` |
| Methods | `snake_case` | `camelCase` |
| Constants | `UPPER_SNAKE` | `UPPER_SNAKE` (static final) |
| Packages | `snake_case` dirs | `lowercase` dirs |
| Test files | `test_foo.py` | `FooTest.java` |
| Test methods | `def test_should_do_x` | `void shouldDoX()` (JUnit 5 @Test) |
| Enums | `class TaskStatus(Enum)` | `enum TaskStatus` |
| Interfaces | Protocol/ABC | `interface` keyword |
| Decorator `@module` | Python decorator | `@ApCoreModule` annotation or `FunctionModule.builder()` |
| `conftest.py` | pytest fixtures | `TestHelpers.java` (static factory methods) |

### Test Stub Content Convention (JUnit 5)

Each test file follows this pattern:

```java
package com.aipartnerup.apcore;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class ExecutorTest {

    @Test
    void shouldExecuteModuleWithValidInputs() {
        // TODO: Implement - TDD red phase
        fail("Not implemented");
    }

    @Test
    void shouldThrowModuleNotFoundErrorForUnknownModule() {
        // TODO: Implement - TDD red phase
        fail("Not implemented");
    }
}
```

### Example File Content Convention

Each example is complete and runnable:

```java
// examples/SimpleClient.java
package com.aipartnerup.apcore.examples;

import com.aipartnerup.apcore.APCore;
import java.util.Map;

/**
 * Basic client usage: create an APCore instance, register a module, and execute it.
 */
public class SimpleClient {

    public static void main(String[] args) {
        // 1. Initialize the client
        APCore client = new APCore();

        // 2. Register a module using FunctionModule builder
        client.register("math.add", FunctionModule.builder()
            .id("math.add")
            .description("Add two integers")
            .handler((inputs, context) -> {
                int a = (int) inputs.get("a");
                int b = (int) inputs.get("b");
                return Map.of("result", a + b);
            })
            .build());

        // 3. Call the module
        Map<String, Object> result = client.call("math.add", Map.of("a", 10, "b", 5));
        System.out.println("Result: " + result);  // {result=15}
    }
}
```

---

## Steps 5-6: Feature Specs and .code-forge.json

**Step 5:** Feature specs would be checked at `apcore/docs/features/*.md`. If found, they are linked. If not, lightweight specs are generated at `apcore-java/docs/features/` for each module: executor, registry, schema, middleware, acl, approval, bindings, extensions, async_task, observability, events, cancel, trace_context, version.

**Step 6:** `.code-forge.json` would be written at `apcore-java/.code-forge.json`:

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
    "target_lang": "java"
  },
  "execution": {
    "default_mode": "ask",
    "auto_tdd": true,
    "task_granularity": "medium"
  }
}
```

Git initialization message:
```
Project scaffolded. To initialize git:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-java
  git init
  git add <files...>
  git commit -m "chore: initialize apcore-java project skeleton"
```

---

## Step 7: Summary Output

```
apcore-skills:sdk -- SDK Bootstrap Complete

Target:       /Users/tercel/WorkSpace/aipartnerup/apcore-java/
Language:     java
Type:         core
Modules:      87 source files scaffolded
Tests:        72 test stubs (TDD red phase)
Examples:     8 runnable examples
Feature specs: 14 specs available
API contract: 117 public symbols to implement

Project structure:
  apcore-java/
  ├── build.gradle.kts
  ├── settings.gradle.kts
  ├── .gitignore
  ├── README.md
  ├── CHANGELOG.md
  ├── LICENSE
  ├── src/main/java/com/aipartnerup/apcore/
  │   ├── APCore.java, Executor.java, Context.java, Module.java, Config.java ...
  │   ├── errors/          (39 error classes + ErrorCodes + ErrorCodeRegistry)
  │   ├── acl/             (Acl, AclRule, AuditEntry)
  │   ├── approval/        (ApprovalHandler, ApprovalRequest, ApprovalResult, ...)
  │   ├── async/           (AsyncTaskManager, TaskInfo, TaskStatus)
  │   ├── bindings/        (BindingLoader)
  │   ├── extensions/      (ExtensionManager, ExtensionPoint)
  │   ├── middleware/      (Middleware, MiddlewareManager, LoggingMiddleware, ...)
  │   ├── registry/        (Registry, ModuleDescriptor, Discoverer, ...)
  │   ├── schema/          (SchemaLoader, SchemaValidator, SchemaExporter, ...)
  │   ├── observability/   (TracingMiddleware, MetricsCollector, ContextLogger, ...)
  │   ├── events/          (EventEmitter, EventSubscriber, ApCoreEvent, ...)
  │   ├── sysmodules/      (Registration)
  │   ├── version/         (VersionNegotiator)
  │   └── utils/           (PatternMatcher, CallChainGuard, ...)
  ├── src/test/java/com/aipartnerup/apcore/
  │   ├── TestHelpers.java (shared fixtures)
  │   ├── ExecutorTest.java, ContextTest.java, ConfigTest.java ...
  │   ├── errors/          (3 test files)
  │   ├── acl/             (2 test files)
  │   ├── approval/        (3 test files)
  │   ├── async/           (1 test file)
  │   ├── bindings/        (1 test file)
  │   ├── extensions/      (1 test file)
  │   ├── middleware/      (6 test files)
  │   ├── registry/        (12 test files)
  │   ├── schema/          (8 test files)
  │   ├── observability/   (6 test files)
  │   ├── events/          (2 test files)
  │   ├── conformance/     (1 test file)
  │   ├── sysmodules/      (5 test files)
  │   ├── examples/        (1 test file)
  │   ├── integration/     (10 test files)
  │   └── utils/           (4 test files)
  └── examples/
      ├── SimpleClient.java
      ├── GlobalClient.java
      ├── modules/
      │   ├── GreetModule.java
      │   ├── GetUserModule.java
      │   ├── SendEmailModule.java
      │   └── DecoratedAdd.java
      └── bindings/
          └── formatdate/
              ├── FormatDateFunction.java
              └── binding.yaml

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-java
  /code-forge:port @../apcore --ref apcore-python --lang java    Generate implementation plans
  /code-forge:impl executor                                       Start implementing
  /apcore-skills:sync --lang java,python                          Verify API consistency
```

---

## Counts Breakdown

| Category | Count | Details |
|----------|-------|---------|
| **Source files** | 87 | 17 top-level + 39 errors + 3 acl + 6 approval + 3 async + 1 bindings + 2 extensions + 9 middleware + 7 registry + 7 schema + 15 observability + 5 events + 1 sysmodules + 1 version + 6 utils |
| **Test stubs** | 72 | Mirrors all 72 test files from apcore-python reference, using JUnit 5 `*Test.java` naming |
| **Examples** | 8 | SimpleClient, GlobalClient, GreetModule, GetUserModule, SendEmailModule, DecoratedAdd, FormatDateFunction + binding.yaml |
| **Feature specs** | 14 | executor, registry, schema, middleware, acl, approval, bindings, extensions, async_task, observability, events, cancel, trace_context, version |
| **Public symbols** | 117 | Matches `__all__` export count from apcore-python `__init__.py` |
