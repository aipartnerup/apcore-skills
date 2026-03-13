# SDK Skill Simulation: Rust MCP Bridge

Command: `/apcore-skills:sdk rust --type mcp --ref apcore-mcp-python`

Working directory: `/Users/tercel/WorkSpace/aipartnerup/`

---

## Step 0: Ecosystem Discovery

Ecosystem root detected: `/Users/tercel/WorkSpace/aipartnerup/`

Discovered repos:
- apcore (protocol spec)
- apcore-python (core SDK, Python)
- apcore-typescript (core SDK, TypeScript)
- apcore-mcp-python (MCP bridge, Python)
- apcore-mcp-typescript (MCP bridge, TypeScript)
- apcore-rust (core SDK, Rust -- already exists)

Protocol spec path: `/Users/tercel/WorkSpace/aipartnerup/apcore/PROTOCOL_SPEC.md`

---

## Step 1: Parse Arguments

Parsed from `$ARGUMENTS = "rust --type mcp --ref apcore-mcp-python"`:

| Parameter | Value | Source |
|-----------|-------|--------|
| `<language>` | `rust` | Positional argument |
| `--type` | `mcp` | Explicit flag |
| `--ref` | `apcore-mcp-python` | Explicit flag |

Derived values:
- Target repo name: `apcore-mcp-rust`
- Target path: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/`
- Reference repo path: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-python/`
- Reference language: Python

Target directory check: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/` does NOT exist. Proceeding.

Display output:
```
SDK Bootstrap:
  Language:   rust
  Type:       mcp
  Reference:  apcore-mcp-python (python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
```

---

## Step 2: Extract API Contract (Sub-agent)

The sub-agent would read the following files from `apcore-mcp-python`:

- `src/apcore_mcp/__init__.py` -- all public exports
- `src/apcore_mcp/server/` -- server factory, transport interfaces
- `src/apcore_mcp/auth/` -- authentication interfaces
- `src/apcore_mcp/adapters/` -- adapter interfaces
- `src/apcore_mcp/converters/` -- converter interfaces
- `apcore/PROTOCOL_SPEC.md` -- protocol definitions
- `apcore/docs/spec/type-mapping.md` -- if exists (not found in this case)

### Extracted API Contract

```
API_CONTRACT:
  type: mcp
  source: apcore-mcp-python
  source_version: 0.10.0
  export_count: 25
  module_count: 15 (source files)

MODULES:
- module: root
  file: __init__.py
  classes:
    - APCoreMCP:
        constructor: (extensions_dir_or_backend: str | Path | object, *, name: str = "apcore-mcp", version: str | None = None, tags: list[str] | None = None, prefix: str | None = None, log_level: str | None = None, validate_inputs: bool = False)
        methods:
          - serve(*, transport: str = "stdio", host: str = "127.0.0.1", port: int = 8000, ...) -> None
          - async_serve(...) -> AsyncIterator[Starlette] [async]
          - to_openai_tools(...) -> list[dict]
  functions:
    - serve(registry_or_executor, *, transport: str = "stdio", host: str = "127.0.0.1", port: int = 8000, name: str = "apcore-mcp", version: str | None = None, on_startup: Callable | None = None, on_shutdown: Callable | None = None, tags: list[str] | None = None, prefix: str | None = None, log_level: str | None = None, dynamic: bool = False, validate_inputs: bool = False, metrics_collector: MetricsExporter | None = None, explorer: bool = False, explorer_prefix: str = "/explorer", allow_execute: bool = False, authenticator: Authenticator | None = None, require_auth: bool = True, exempt_paths: set[str] | None = None, approval_handler: object | None = None, output_formatter: Callable | None = None) -> None
    - async_serve(registry_or_executor, ...) -> AsyncIterator[Starlette] [async]
    - to_openai_tools(registry_or_executor, *, embed_annotations: bool = False, strict: bool = False, tags: list[str] | None = None, prefix: str | None = None) -> list[dict]
  types:
    - ElicitResult: TypedDict { action: "accept" | "decline" | "cancel", content?: dict | None }
  constants:
    - MCP_PROGRESS_KEY: str = "_mcp_progress"
    - MCP_ELICIT_KEY: str = "_mcp_elicit"

- module: server.factory
  file: server/factory.py
  classes:
    - MCPServerFactory:
        constructor: ()
        methods:
          - create_server(name: str = "apcore-mcp", version: str = "0.1.0") -> Server
          - build_tool(descriptor: Any) -> Tool
          - build_tools(registry: Any, tags: list[str] | None = None, prefix: str | None = None) -> list[Tool]
          - register_handlers(server: Server, tools: list[Tool], router: Any) -> None
          - register_resource_handlers(server: Server, registry: Any) -> None
          - build_init_options(server: Server, name: str, version: str) -> InitializationOptions

- module: server.router
  file: server/router.py
  classes:
    - ExecutionRouter:
        constructor: (executor: Any, validate_inputs: bool = False, output_formatter: Callable | None = None)
        methods:
          - handle_call(name: str, arguments: dict, extra: dict | None = None) -> tuple[list, bool, str | None] [async]

- module: server.transport
  file: server/transport.py
  classes:
    - MetricsExporter (Protocol):
        methods:
          - export_prometheus() -> str
    - TransportManager:
        constructor: (metrics_collector: MetricsExporter | None = None)
        methods:
          - set_module_count(count: int) -> None
          - run_stdio(server, init_options) [async]
          - run_streamable_http(server, init_options, *, host, port, extra_routes, middleware) [async]
          - run_sse(server, init_options, *, host, port, extra_routes, middleware) [async]
          - build_streamable_http_app(server, init_options, *, extra_routes, middleware) -> AsyncIterator [async]

- module: server.listener
  file: server/listener.py
  classes:
    - RegistryListener:
        constructor: (registry: Any, factory: MCPServerFactory)
        methods:
          - (listens for registry changes and updates MCP tool list)

- module: server.server
  file: server/server.py
  classes:
    - MCPServer:
        (wrapper around low-level MCP Server for convenience)

- module: auth.protocol
  file: auth/protocol.py
  classes:
    - Authenticator (Protocol):
        methods:
          - authenticate(headers: dict[str, str]) -> Identity | None

- module: auth.jwt
  file: auth/jwt.py
  classes:
    - JWTAuthenticator:
        constructor: (secret: str, algorithms: list[str], ...)
        methods:
          - authenticate(headers: dict[str, str]) -> Identity | None
    - ClaimMapping:
        (maps JWT claims to Identity fields)

- module: auth.middleware
  file: auth/middleware.py
  classes:
    - AuthMiddleware:
        constructor: (app, authenticator, require_auth: bool = True, exempt_paths: set[str] | None = None, ...)
        methods:
          - __call__(scope, receive, send) [async]
  functions:
    - extract_headers(scope) -> dict[str, str]

- module: adapters.annotations
  file: adapters/annotations.py
  classes:
    - AnnotationMapper:
        methods:
          - has_requires_approval(annotations) -> bool
          - (maps apcore annotations to MCP annotations)

- module: adapters.approval
  file: adapters/approval.py
  classes:
    - ElicitationApprovalHandler:
        (bridges MCP elicitation to apcore approval protocol)

- module: adapters.errors
  file: adapters/errors.py
  classes:
    - ErrorMapper:
        (maps apcore errors to MCP error responses)

- module: adapters.id_normalizer
  file: adapters/id_normalizer.py
  classes:
    - ModuleIDNormalizer:
        (normalizes module IDs for MCP tool naming)

- module: adapters.schema
  file: adapters/schema.py
  classes:
    - SchemaConverter:
        methods:
          - convert_input_schema(descriptor) -> dict

- module: converters.openai
  file: converters/openai.py
  classes:
    - OpenAIConverter:
        methods:
          - convert_registry(registry, *, embed_annotations: bool = False, strict: bool = False, tags: list[str] | None = None, prefix: str | None = None) -> list[dict]

- module: helpers
  file: helpers.py
  functions:
    - report_progress(context, progress: float, total: float | None = None, message: str | None = None) -> None [async]
    - elicit(context, message: str, requested_schema: dict | None = None) -> ElicitResult | None [async]

- module: constants
  file: constants.py
  constants:
    - REGISTRY_EVENTS: dict = { "REGISTER": "register", "UNREGISTER": "unregister" }
    - ERROR_CODES: dict = { "MODULE_NOT_FOUND", "SCHEMA_VALIDATION_ERROR", "ACL_DENIED", "CALL_DEPTH_EXCEEDED", "CIRCULAR_CALL", "CALL_FREQUENCY_EXCEEDED", "INTERNAL_ERROR", "MODULE_TIMEOUT", "MODULE_LOAD_ERROR", "MODULE_EXECUTE_ERROR", "GENERAL_INVALID_INPUT", "APPROVAL_DENIED", "APPROVAL_TIMEOUT", "APPROVAL_PENDING", "VERSION_INCOMPATIBLE", "ERROR_CODE_COLLISION", "EXECUTION_CANCELLED" }
    - MODULE_ID_PATTERN: regex = r"^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$"

ERROR_HIERARCHY:
  base: (MCP bridge does not define its own error hierarchy; maps apcore errors via ErrorMapper)
  codes: ERROR_CODES dict with 17 error code constants
  classes:
    - ErrorMapper maps to MCP error responses (no custom exception classes)

EXTENSION_POINTS:
  - Authenticator: authenticate(headers: dict[str, str]) -> Identity | None
  - MetricsExporter: export_prometheus() -> str
  - approval_handler: pluggable approval handling
  - output_formatter: Callable (dict) -> str for formatting results
```

---

## Step 3: Confirm Tech Stack (Questions for User)

The skill specifies these questions for Rust:

> **Rust SDK tech stack confirmation:**
>
> 1. **Rust edition:** "2021 (Recommended)" / "2024"
> 2. **Async runtime:** "tokio (Recommended)" / "async-std" / "None (sync only)"
> 3. **Serialization:** "serde (Recommended)" / "Other"
> 4. **Schema:** "schemars (Recommended)" / "Other"

For simulation, assuming recommended defaults:

```
tech_stack:
  edition: "2021"
  async_runtime: "tokio"
  serialization: "serde"
  schema: "schemars"
```

---

## Step 4: Scaffold Project (Sub-agent)

The skill instructs the sub-agent to create the MCP bridge project structure. Based on the skill's template for MCP bridges, the COMPLETE project structure would be:

```
/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
├── Cargo.toml                          # Build config: edition 2021, deps: tokio, serde, schemars, etc.
├── .gitignore                          # Rust patterns: /target, *.rs.bk, Cargo.lock (for lib)
├── README.md                           # Project name, description, installation, link to docs
├── CHANGELOG.md                        # Empty "## [Unreleased]" section
├── LICENSE                             # Apache-2.0 (detected from apcore-mcp-python)
├── src/
│   ├── lib.rs                          # Main module file — re-exports all public types
│   ├── server/
│   │   ├── mod.rs                      # Server module — re-exports factory, transport, router, listener, server
│   │   ├── factory.rs                  # MCPServerFactory stub (create_server, build_tool, build_tools, register_handlers, register_resource_handlers, build_init_options)
│   │   ├── router.rs                   # ExecutionRouter stub (handle_call)
│   │   ├── transport.rs                # TransportManager + MetricsExporter trait stub
│   │   ├── listener.rs                 # RegistryListener stub
│   │   └── server.rs                   # McpServer wrapper stub
│   ├── auth/
│   │   ├── mod.rs                      # Auth module — re-exports Authenticator, JwtAuthenticator, ClaimMapping, AuthMiddleware
│   │   ├── protocol.rs                 # Authenticator trait stub
│   │   ├── jwt.rs                      # JwtAuthenticator + ClaimMapping stubs
│   │   └── middleware.rs               # AuthMiddleware stub
│   ├── adapters/
│   │   ├── mod.rs                      # Adapters module — re-exports all adapters
│   │   ├── annotations.rs              # AnnotationMapper stub
│   │   ├── approval.rs                 # ElicitationApprovalHandler stub
│   │   ├── errors.rs                   # ErrorMapper stub
│   │   ├── id_normalizer.rs            # ModuleIdNormalizer stub
│   │   └── schema.rs                   # SchemaConverter stub
│   ├── converters/
│   │   ├── mod.rs                      # Converters module — re-exports OpenAiConverter
│   │   └── openai.rs                   # OpenAiConverter stub
│   ├── cli.rs                          # CLI entry point stub
│   └── explorer/
│       ├── mod.rs                      # Explorer module — re-exports routes, html
│       ├── routes.rs                   # Explorer routes stub
│       └── html.rs                     # Explorer HTML generation stub
└── tests/
    └── {test-config}                   # No specific test runner config for Rust (Cargo handles this)
```

### What the skill explicitly instructs for tests/ and examples/

The skill's Step 4 scaffold template for MCP bridges specifies:

```
└── tests/
    └── {test-config}                # pytest.ini / vitest.config / test runner config
```

That is the ONLY test-related content the skill instructs to create. Specifically:

- **tests/**: A `tests/` directory with a test runner configuration file. For Rust, Cargo's built-in test runner needs no separate config file. The skill does NOT instruct creating any test source files, test modules, or example test cases.
- **examples/**: The skill does NOT mention an `examples/` directory at all. The MCP bridge scaffold template has no `examples/` entry. The skill does not instruct creating any example files.

### Stub File Content

Per the skill's instructions, each stub file would contain:
1. Module/file header comment referencing the protocol spec section
2. Import of base types from the main module
3. Class/function stubs with correct signatures from the API contract
4. TODO comments indicating what needs to be implemented
5. Type annotations matching Rust convention (snake_case for functions/methods, PascalCase for types)

Example of what `src/auth/protocol.rs` would look like:

```rust
//! Authenticator trait for pluggable authentication backends.
//!
//! See: PROTOCOL_SPEC.md — Authentication

use crate::Identity;

/// Protocol for authentication backends.
///
/// Implementations extract credentials from HTTP headers and return
/// an `Identity` on success, or `None` on failure.
pub trait Authenticator {
    /// Authenticate a request from its headers.
    ///
    /// # Arguments
    /// * `headers` - Lowercase header keys mapped to their values.
    ///
    /// # Returns
    /// An `Identity` if authentication succeeds, `None` otherwise.
    ///
    /// TODO: Implement authentication logic
    fn authenticate(&self, headers: &std::collections::HashMap<String, String>) -> Option<Identity>;
}
```

### Verification Checklist (performed after sub-agent)

The skill instructs verifying:
- [x] Build config file exists (`Cargo.toml`)
- [x] Main module file exists with exports (`src/lib.rs`)
- [x] At least 5 source files exist (15+ .rs files)
- [x] Tests directory exists (`tests/`)
- [x] README.md exists

---

## Step 5: Generate Feature Specs

The skill checks for feature specs at `{protocol_path}/docs/features/*.md`.

Feature specs found at `/Users/tercel/WorkSpace/aipartnerup/apcore/docs/features/`:
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

Display: `Feature specs found: 10 specs in /Users/tercel/WorkSpace/aipartnerup/apcore/docs/features/`

These would be linked via `.code-forge.json` rather than regenerated.

---

## Step 6: Generate .code-forge.json

The skill instructs writing this file at the target path:

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
    "sources": ["../apcore-mcp-python/planning/*/plan.md"]
  },
  "port": {
    "source_docs": "../apcore",
    "reference_impl": "../apcore-mcp-python",
    "target_lang": "rust"
  },
  "execution": {
    "default_mode": "ask",
    "auto_tdd": true,
    "task_granularity": "medium"
  }
}
```

Display:
```
Project scaffolded. To initialize git:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust
  git init
  git add <files...>
  git commit -m "chore: initialize apcore-mcp-rust project skeleton"
```

---

## Step 7: Display Summary and Next Steps

```
apcore-skills:sdk — SDK Bootstrap Complete

Target: /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
Language: rust
Type: mcp
Modules: 15 source files scaffolded
Feature specs: 10 specs available
API contract: 25 public symbols to implement

Project structure:
  apcore-mcp-rust/
  ├── Cargo.toml
  ├── .gitignore
  ├── README.md
  ├── CHANGELOG.md
  ├── LICENSE
  ├── .code-forge.json
  ├── src/
  │   ├── lib.rs
  │   ├── cli.rs
  │   ├── server/
  │   │   ├── mod.rs
  │   │   ├── factory.rs
  │   │   ├── router.rs
  │   │   ├── transport.rs
  │   │   ├── listener.rs
  │   │   └── server.rs
  │   ├── auth/
  │   │   ├── mod.rs
  │   │   ├── protocol.rs
  │   │   ├── jwt.rs
  │   │   └── middleware.rs
  │   ├── adapters/
  │   │   ├── mod.rs
  │   │   ├── annotations.rs
  │   │   ├── approval.rs
  │   │   ├── errors.rs
  │   │   ├── id_normalizer.rs
  │   │   └── schema.rs
  │   ├── converters/
  │   │   ├── mod.rs
  │   │   └── openai.rs
  │   └── explorer/
  │       ├── mod.rs
  │       ├── routes.rs
  │       └── html.rs
  └── tests/

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
  /code-forge:port @../apcore --ref apcore-mcp-python --lang rust    Generate implementation plans
  /code-forge:impl {first-feature}                                    Start implementing
  /apcore-skills:sync --lang rust,python                              Verify API consistency
```

---

## Observations on Skill Gaps

### What the skill DOES instruct:
- Full project skeleton with all source stub files
- tests/ directory with a test config placeholder
- README.md, CHANGELOG.md, LICENSE, .gitignore
- .code-forge.json for plan generation
- Feature specs (link existing or generate)
- Git initialization instructions (left to user)

### What the skill does NOT instruct:
- **No test source files**: Only `tests/{test-config}` is specified. No actual test files (e.g., `tests/test_factory.rs`, `tests/integration/`) are created. For Rust, there is no test runner config file needed, so the tests/ directory would effectively be empty.
- **No examples/ directory**: The scaffold template has no `examples/` entry for either core or MCP type.
- **No inline #[cfg(test)] modules**: The skill says stubs contain "TODO comments" but does not mention inline test modules.
- **No constants/helpers file**: The Python reference has `constants.py` and `helpers.py` but these are not listed in the MCP bridge scaffold template (they would need to be added as extra files outside the template).
- **No _utils equivalent**: The `_utils.py` (resolve_executor, resolve_registry) is not in the template.
- **No __main__.py / binary target**: The skill lists `cli.{ext}` but does not mention a `[[bin]]` section in Cargo.toml or a `main.rs`.

### TDD Gap:
The Anti-Rationalization Table states "TDD is mandatory. Test infrastructure is set up in scaffolding." However, the actual scaffold template only creates `tests/{test-config}` -- no test files, no test helpers, no test fixtures. The test infrastructure claim is aspirational rather than implemented in the template.
