# Simulation: /apcore-skills:sdk java (OLD Skill)

Simulated execution of SKILL.md at `skills/sdk-workspace/skill-snapshot/SKILL.md`

---

## Step 0: Ecosystem Discovery

The skill references `@../shared/ecosystem.md` for ecosystem discovery. Working directory is `/Users/tercel/WorkSpace/aipartnerup/`, which contains the apcore ecosystem repos. The ecosystem root is identified as `/Users/tercel/WorkSpace/aipartnerup/`.

Protocol spec found at: `/Users/tercel/WorkSpace/aipartnerup/apcore/PROTOCOL_SPEC.md`

---

## Step 1: Parse Arguments

**Input:** `java`

**Parsing result:**

| Parameter | Value | Source |
|-----------|-------|--------|
| `<language>` | `java` | Positional argument |
| `--type` | `core` | Default (not specified) |
| `--ref` | `apcore-python` | Auto-detected: CWD is not a same-type apcore repo, so for `core` type, prefer `apcore-python` |

**Derived values:**
- Target repo name: `apcore-java`
- Target path: `/Users/tercel/WorkSpace/aipartnerup/apcore-java/`

**Directory check:** `/Users/tercel/WorkSpace/aipartnerup/apcore-java/` does not exist. Proceed.

**Display output:**
```
SDK Bootstrap:
  Language:   java
  Type:       core
  Reference:  apcore-python (python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-java/
```

---

## Step 2: Extract API Contract (Sub-agent)

A sub-agent would be spawned to read the reference implementation. Based on reading `/Users/tercel/WorkSpace/aipartnerup/apcore-python/src/apcore/__init__.py`, the sub-agent would produce the following API contract summary:

### API_CONTRACT:
```
type: core
source: apcore-python
source_version: 0.13.0
export_count: 170 (from __all__)
module_count: 64 (.py files in src/apcore/)
```

### MODULES (key modules extracted):

- **module: client** (`client.py`)
  - Classes:
    - `APCore`: Main client class
      - Methods: `call()`, `call_async()`, `stream()`, `validate()`, `register()`, `describe()`, `use()`, `use_before()`, `use_after()`, `remove()`, `discover()`, `list_modules()`, `on()`, `off()`, `disable()`, `enable()`, `module()` (decorator)

- **module: executor** (`executor.py`)
  - Classes:
    - `Executor`
  - Functions:
    - `redact_sensitive()`
  - Constants:
    - `REDACTED_VALUE`

- **module: context** (`context.py`)
  - Classes:
    - `Context`, `ContextFactory`, `Identity`

- **module: module** (`module.py`)
  - Classes:
    - `Module`, `ModuleAnnotations`, `ModuleExample`, `PreflightCheckResult`, `PreflightResult`, `ValidationResult`

- **module: config** (`config.py`)
  - Classes:
    - `Config`

- **module: acl** (`acl.py`)
  - Classes:
    - `ACL`, `ACLRule`, `AuditEntry`

- **module: approval** (`approval.py`)
  - Classes:
    - `ApprovalHandler`, `ApprovalRequest`, `ApprovalResult`, `AlwaysDenyHandler`, `AutoApproveHandler`, `CallbackApprovalHandler`

- **module: async_task** (`async_task.py`)
  - Classes:
    - `AsyncTaskManager`, `TaskInfo`, `TaskStatus`

- **module: bindings** (`bindings.py`)
  - Classes:
    - `BindingLoader`

- **module: cancel** (`cancel.py`)
  - Classes:
    - `CancelToken`, `ExecutionCancelledError`

- **module: decorator** (`decorator.py`)
  - Classes:
    - `FunctionModule`
  - Functions:
    - `module()` (decorator factory)

- **module: extensions** (`extensions.py`)
  - Classes:
    - `ExtensionManager`, `ExtensionPoint`

- **module: trace_context** (`trace_context.py`)
  - Classes:
    - `TraceContext`, `TraceParent`

- **module: version** (`version.py`)
  - Classes:
    - `VersionIncompatibleError`
  - Functions:
    - `negotiate_version()`

- **module: middleware/** (subdirectory with `base.py`, `manager.py`, `logging.py`, `retry.py`, `error_history.py`, `platform_notify.py`, `adapters.py`)
  - Classes:
    - `Middleware`, `MiddlewareManager`, `BeforeMiddleware`, `AfterMiddleware`, `LoggingMiddleware`, `MiddlewareChainError`, `RetryConfig`, `RetryMiddleware`, `ErrorHistoryMiddleware`, `PlatformNotifyMiddleware`

- **module: registry/** (subdirectory with `registry.py`, `types.py`, `dependencies.py`, `scanner.py`, `metadata.py`, `validation.py`, `conflicts.py`, `entry_point.py`, `version.py`, `schema_export.py`)
  - Classes:
    - `Registry`, `Discoverer`, `ModuleValidator`, `ModuleDescriptor`, `DiscoveredModule`, `DependencyInfo`
  - Constants:
    - `MAX_MODULE_ID_LENGTH`, `MODULE_ID_PATTERN`, `REGISTRY_EVENTS`, `RESERVED_WORDS`

- **module: schema/** (subdirectory with `loader.py`, `validator.py`, `exporter.py`, `ref_resolver.py`, `types.py`, `strict.py`, `annotations.py`)
  - Classes:
    - `SchemaLoader`, `SchemaValidator`, `SchemaExporter`, `RefResolver`, `SchemaStrategy`, `ExportProfile`
  - Functions:
    - `to_strict_schema()`

- **module: observability/** (subdirectory with `tracing.py`, `metrics.py`, `context_logger.py`, `error_history.py`, `usage.py`)
  - Classes:
    - `TracingMiddleware`, `ContextLogger`, `ObsLoggingMiddleware`, `MetricsMiddleware`, `MetricsCollector`, `Span`, `StdoutExporter`, `InMemoryExporter`, `OTLPExporter`, `SpanExporter`, `ErrorEntry`, `ErrorHistory`, `UsageCollector`, `UsageMiddleware`
  - Functions:
    - `create_span()`

- **module: events/** (subdirectory with `emitter.py`, `subscribers.py`)
  - Classes:
    - `ApCoreEvent`, `EventEmitter`, `EventSubscriber`, `WebhookSubscriber`, `A2ASubscriber`

- **module: sys_modules/** (subdirectory with `registration.py`, `health.py`, `manifest.py`, `usage.py`, `control.py`)
  - Functions:
    - `register_sys_modules()`, `register_subscriber_type()`, `unregister_subscriber_type()`, `reset_subscriber_registry()`

- **module: utils/** (subdirectory with `call_chain.py`, `normalize.py`, `error_propagation.py`, `pattern.py`)
  - Functions:
    - `match_pattern()`, `guard_call_chain()`, `normalize_to_canonical_id()`, `calculate_specificity()`, `propagate_error()`

### ERROR_HIERARCHY:
```
base: ModuleError
  constructor: (code, message, details=None, cause=None, trace_id=None, retryable=_UNSET, ai_guidance=None, user_fixable=None, suggestion=None)
codes: ErrorCodes (enum with all error code string values)
classes:
  - ConfigNotFoundError, ConfigError
  - ACLRuleError, ACLDeniedError
  - ApprovalError, ApprovalDeniedError, ApprovalTimeoutError, ApprovalPendingError
  - ModuleNotFoundError, ModuleDisabledError, ModuleTimeoutError, ModuleLoadError, ModuleExecuteError
  - SchemaValidationError, SchemaNotFoundError, SchemaParseError, SchemaCircularRefError
  - CallDepthExceededError, CircularCallError, CallFrequencyExceededError
  - InvalidInputError, FuncMissingTypeHintError, FuncMissingReturnTypeError
  - BindingInvalidTargetError, BindingModuleNotFoundError, BindingCallableNotFoundError, BindingNotCallableError, BindingSchemaMissingError, BindingFileInvalidError
  - CircularDependencyError, DependencyNotFoundError
  - ReloadFailedError, InternalError, FeatureNotImplementedError
  - ErrorCodeCollisionError
  - VersionIncompatibleError (in version.py)
  - ExecutionCancelledError (in cancel.py)
  - MiddlewareChainError (in middleware/)
support: ErrorCodeRegistry (for collision detection)
```

### EXTENSION_POINTS:
```
- Middleware: before(context, module_id, inputs) -> (context, module_id, inputs), after(context, module_id, result) -> result
- Discoverer: discover(registry) -> list[DiscoveredModule]
- ModuleValidator: validate(module) -> bool
- ApprovalHandler: request_approval(request) -> ApprovalResult
- SpanExporter: export(spans) -> None
- EventSubscriber: on_event(event) -> None
- ExtensionPoint: (dynamic extension registration)
```

---

## Step 3: Confirm Tech Stack (AskUserQuestion)

The skill specifies these questions for Java:

```
For Java, confirm the following tech stack choices:

1. Java version: "17+ (Recommended)" / "21+"
2. Build tool: "Gradle (Recommended)" / "Maven"
3. Schema validation: "Jackson (Recommended)" / "Gson"
4. Test framework: "JUnit 5 (Recommended)" / "TestNG"
```

**Simulated user response (all defaults):**
```
tech_stack:
  java_version: "17+"
  build_tool: "Gradle"
  schema_validation: "Jackson"
  test_framework: "JUnit 5"
```

---

## Step 4: Scaffold Project (Sub-agent)

The skill instructs a sub-agent to create the project skeleton. Based on the SKILL.md template for core SDK, the COMPLETE list of files and directories that would be scaffolded:

### Complete File Listing

```
/Users/tercel/WorkSpace/aipartnerup/apcore-java/
├── build.gradle.kts                          # Gradle build config (Java 17+, JUnit 5, Jackson)
├── settings.gradle.kts                       # Gradle settings
├── .gitignore                                # Java/Gradle-appropriate patterns
├── README.md                                 # Project name, description, installation, link to docs
├── CHANGELOG.md                              # Empty "## [Unreleased]" section
├── LICENSE                                   # Apache-2.0 (detected from apcore-python)
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/aipartnerup/apcore/
│   │           ├── ApCore.java               # Main module file / exports (empty stubs with TODO)
│   │           ├── Executor.java             # stub
│   │           ├── Context.java              # stub
│   │           ├── Module.java               # stub
│   │           ├── Config.java               # stub
│   │           ├── Errors.java               # stub with ErrorCode enum and base ModuleError
│   │           ├── Acl.java                  # stub
│   │           ├── Approval.java             # stub
│   │           ├── AsyncTask.java            # stub
│   │           ├── Bindings.java             # stub
│   │           ├── Decorator.java            # stub
│   │           ├── Extensions.java           # stub
│   │           ├── Cancel.java               # stub -- cancellation support
│   │           ├── TraceContext.java          # stub -- trace context propagation
│   │           ├── middleware/               # stub directory
│   │           ├── registry/                 # stub directory
│   │           ├── schema/                   # stub directory
│   │           ├── observability/            # stub directory
│   │           └── utils/                    # stub directory
│   └── test/
│       └── java/
│           └── com/aipartnerup/apcore/
│               └── (test runner config implied by build.gradle.kts JUnit 5 setup)
└── tests/                                    # NOTE: see analysis below
```

### Regarding tests/ and examples/ Content

**tests/:** The SKILL.md specifies:
```
└── tests/
    └── {test-config}                # pytest.ini / vitest.config / test runner config
```

The skill ONLY instructs creation of a `tests/` directory with a `{test-config}` file inside. For Java with Gradle + JUnit 5, this would be handled by `build.gradle.kts` (the JUnit 5 dependency and test task configuration). There is no instruction to create individual test files, test stubs, or test classes. The skill does NOT tell you to create any test source files -- only test infrastructure/configuration.

In idiomatic Java with Gradle, the test configuration lives in `build.gradle.kts` and test sources go under `src/test/java/`. The skill's `tests/` directory with `{test-config}` might translate to either:
- A top-level `tests/` dir (as written literally), or
- The standard `src/test/` Gradle convention

The skill says to "use idiomatic target-language patterns" so the sub-agent would likely use `src/test/java/` instead of a top-level `tests/` directory, with the test configuration embedded in `build.gradle.kts`.

**examples/:** The skill does NOT mention creating an `examples/` directory anywhere in Step 4. There is no instruction to scaffold example files. The skill's project structure template has no `examples/` entry. This is a notable omission -- the reference implementation (`apcore-python`) has `examples/` with `simple_client.py`, `global_client.py`, and `bindings/` and `modules/` subdirectories, but the skill does not instruct the sub-agent to create any of this.

### Stub File Content

Per the skill, each stub file should contain:
1. Module/file header comment referencing the protocol spec section
2. Import of base types from the main module
3. Class/function stubs with correct signatures from the API contract
4. TODO comments indicating what needs to be implemented
5. Type annotations matching Java convention (camelCase for methods, PascalCase for classes)

### Post-scaffold Verification

The skill instructs verification of:
- [x] Build config file exists (`build.gradle.kts`)
- [x] Main module file exists with exports (`ApCore.java`)
- [x] At least 5 source files exist (14 listed above)
- [x] Tests directory exists (`src/test/java/` or `tests/`)
- [x] README.md exists

---

## Step 5: Generate Feature Specs

Feature specs do NOT exist at `/Users/tercel/WorkSpace/aipartnerup/apcore/docs/features/*.md` (directory not found).

Per the skill, since they don't exist, it would:
- Extract module list from the API contract
- Generate lightweight feature specs at `/Users/tercel/WorkSpace/aipartnerup/apcore-java/docs/features/`
- One per module: executor, registry, schema, middleware, observability, context, module, config, acl, approval, async_task, bindings, cancel, trace_context, decorator, extensions, events, utils, sys_modules, errors, client, version
- Each spec contains: module purpose, public API surface, acceptance criteria
- Display: `Feature specs generated: 22 specs in docs/features/`

---

## Step 6: Generate .code-forge.json

The skill would write `/Users/tercel/WorkSpace/aipartnerup/apcore-java/.code-forge.json`:

```json
{
  "_tool": {
    "name": "code-forge",
    "description": "Transform documentation into actionable development plans",
    "url": "https://github.com/tercel/code-forge"
  },
  "directories": {
    "base": "./",
    "input": "docs/features",
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

Display output:
```
Project scaffolded. To initialize git:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-java/
  git init
  git add <files...>
  git commit -m "chore: initialize apcore-java project skeleton"
```

---

## Step 7: Display Summary and Next Steps

```
apcore-skills:sdk -- SDK Bootstrap Complete

Target: /Users/tercel/WorkSpace/aipartnerup/apcore-java/
Language: java
Type: core
Modules: 14 source files scaffolded
Feature specs: 22 specs available
API contract: 170 public symbols to implement

Project structure:
  apcore-java/
  ├── build.gradle.kts
  ├── settings.gradle.kts
  ├── .gitignore
  ├── README.md
  ├── CHANGELOG.md
  ├── LICENSE
  ├── src/main/java/com/aipartnerup/apcore/
  │   ├── ApCore.java
  │   ├── Executor.java
  │   ├── Context.java
  │   ├── Module.java
  │   ├── Config.java
  │   ├── Errors.java
  │   ├── Acl.java
  │   ├── Approval.java
  │   ├── AsyncTask.java
  │   ├── Bindings.java
  │   ├── Decorator.java
  │   ├── Extensions.java
  │   ├── Cancel.java
  │   ├── TraceContext.java
  │   ├── middleware/
  │   ├── registry/
  │   ├── schema/
  │   ├── observability/
  │   └── utils/
  ├── src/test/java/com/aipartnerup/apcore/
  └── docs/features/ (22 specs)

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-java/
  /code-forge:port @../apcore --ref apcore-python --lang java    Generate implementation plans
  /code-forge:impl {first-feature}                                Start implementing
  /apcore-skills:sync --lang java,python                          Verify API consistency
```

---

## Observations and Gaps in the OLD Skill

### What the skill DOES instruct:
1. 14 top-level source stubs (matching the template list exactly)
2. 5 stub subdirectories (middleware/, registry/, schema/, observability/, utils/)
3. Build configuration, README, CHANGELOG, LICENSE, .gitignore
4. A `tests/` directory with test runner config only
5. Feature specs (one per module) since protocol-level specs don't exist
6. `.code-forge.json` for plan generation

### What the skill does NOT instruct:
1. **No examples/ directory** -- despite the reference having examples
2. **No individual test files** -- only test infrastructure/config, no test stubs per module
3. **No files inside stub subdirectories** -- middleware/, registry/, schema/, observability/, utils/ are listed as "stub directory" with no file listing. The sub-agent receives the API contract and would need to decide what files to create inside these directories.
4. **No events/ subdirectory** -- despite events being a major module in the reference (EventEmitter, EventSubscriber, etc.), the template does not list `events/` as a directory
5. **No sys_modules/ subdirectory** -- despite sys_modules being present in the reference
6. **No client file** -- `client.py` (APCore class) is the main entry point in Python, but the template calls it `{main-module-file}` without being explicit about splitting client logic
7. **No _docstrings equivalent** -- the `parse_docstring` utility from `_docstrings.py` is not represented
8. **No version file** -- `version.py` with `negotiate_version()` is not in the template list
9. **No events integration** -- the template has no events directory despite 5 event-related exports

### Structural ambiguity:
- The template says `src/` as the source directory, but Java convention is `src/main/java/`. The skill says "use idiomatic target-language patterns" which creates tension with the literal template.
- The `tests/` directory with `{test-config}` is vague for Gradle projects where test config lives in `build.gradle.kts` and test sources live under `src/test/java/`.
- Sub-directories are listed as "stub directory" with no guidance on what files to create inside them. The sub-agent must infer this from the API contract.
