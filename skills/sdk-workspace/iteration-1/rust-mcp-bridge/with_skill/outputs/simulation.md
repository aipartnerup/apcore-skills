# SDK Skill Simulation: `/apcore-skills:sdk rust --type mcp --ref apcore-mcp-python`

Generated: 2026-03-13

---

## Step 0: Ecosystem Discovery

Ecosystem root detected: `/Users/tercel/WorkSpace/aipartnerup/`

Repos found:
- apcore (protocol spec)
- apcore-python (core SDK, Python)
- apcore-typescript (core SDK, TypeScript)
- apcore-mcp-python (MCP bridge, Python)
- apcore-mcp-typescript (MCP bridge, TypeScript)
- apcore-rust (core SDK, Rust -- exists but is for core, not MCP)

Protocol spec: `/Users/tercel/WorkSpace/aipartnerup/apcore/PROTOCOL_SPEC.md`

---

## Step 1: Parse Arguments

**Raw arguments:** `rust --type mcp --ref apcore-mcp-python`

| Parameter | Value | Source |
|-----------|-------|--------|
| `<language>` | `rust` | Positional argument |
| `--type` | `mcp` | Explicit flag |
| `--ref` | `apcore-mcp-python` | Explicit `--ref` flag |

**Derived values:**

- Target repo name: `apcore-mcp-rust`
- Target path: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/`
- Reference path: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-python/`
- Reference language: Python

**Target directory check:** Does not exist. Proceeding.

```
SDK Bootstrap:
  Language:   rust
  Type:       mcp
  Reference:  apcore-mcp-python (python)
  Target:     /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
```

---

## Step 2: Extract API Contract (Sub-agent)

The sub-agent reads the reference implementation and produces the following structured output.

### API_CONTRACT

```
API_CONTRACT:
  type: mcp
  source: apcore-mcp-python
  source_version: 0.10.0
  export_count: 24
  module_count: 17 (source files)
```

### MODULES

```
MODULES:
- module: apcore_mcp (top-level)
  file: src/apcore_mcp/__init__.py
  functions:
    - serve(registry_or_executor, *, transport="stdio", host="127.0.0.1",
            port=8000, name="apcore-mcp", version=None, on_startup=None,
            on_shutdown=None, tags=None, prefix=None, log_level=None,
            dynamic=False, validate_inputs=False, metrics_collector=None,
            explorer=False, explorer_prefix="/explorer", allow_execute=False,
            authenticator=None, require_auth=True, exempt_paths=None,
            approval_handler=None, output_formatter=None) -> None
    - async_serve(registry_or_executor, *, name="apcore-mcp", version=None,
                  tags=None, prefix=None, log_level=None, validate_inputs=False,
                  metrics_collector=None, explorer=False,
                  explorer_prefix="/explorer", allow_execute=False,
                  authenticator=None, require_auth=True, exempt_paths=None,
                  approval_handler=None, output_formatter=None)
                  -> AsyncIterator[Starlette] [async context manager]
    - to_openai_tools(registry_or_executor, *, embed_annotations=False,
                      strict=False, tags=None, prefix=None) -> list[dict]

- module: server.factory
  file: src/apcore_mcp/server/factory.py
  classes:
    - MCPServerFactory:
        constructor: ()
        methods:
          - create_server(name="apcore-mcp", version="0.1.0") -> Server
          - build_tool(descriptor) -> Tool
          - build_tools(registry, tags=None, prefix=None) -> list[Tool]
          - register_handlers(server, tools, router) -> None
          - register_resource_handlers(server, registry) -> None
          - build_init_options(server, name, version) -> InitializationOptions

- module: server.server
  file: src/apcore_mcp/server/server.py
  classes:
    - MCPServer:
        (wraps MCP low-level server with lifecycle management)

- module: server.router
  file: src/apcore_mcp/server/router.py
  classes:
    - ExecutionRouter:
        constructor: (executor, validate_inputs=False, output_formatter=None)
        methods:
          - handle_call(name, arguments, extra=None) -> (content, is_error, trace_id) [async]

- module: server.transport
  file: src/apcore_mcp/server/transport.py
  classes:
    - TransportManager:
        constructor: (metrics_collector=None)
        methods:
          - set_module_count(count) -> None
          - run_stdio(server, init_options) -> None [async]
          - run_streamable_http(server, init_options, host, port, extra_routes, middleware) -> None [async]
          - run_sse(server, init_options, host, port, extra_routes, middleware) -> None [async]
          - build_streamable_http_app(server, init_options, extra_routes, middleware) -> AsyncContextManager[Starlette] [async]
    - MetricsExporter:
        (interface for Prometheus metrics export)

- module: server.listener
  file: src/apcore_mcp/server/listener.py
  classes:
    - RegistryListener:
        (listens for registry events and updates tool lists dynamically)

- module: auth.protocol
  file: src/apcore_mcp/auth/protocol.py
  types:
    - Authenticator (Protocol):
        - authenticate(headers: dict[str, str]) -> Identity | None

- module: auth.jwt
  file: src/apcore_mcp/auth/jwt.py
  classes:
    - JWTAuthenticator:
        constructor: (key, algorithms=["HS256"], claim_mapping=None)
        methods:
          - authenticate(headers) -> Identity | None
    - ClaimMapping:
        (dataclass for mapping JWT claims to Identity fields)

- module: auth.middleware
  file: src/apcore_mcp/auth/middleware.py
  classes:
    - AuthMiddleware:
        constructor: (app, authenticator, require_auth=True, exempt_paths=None, exempt_prefixes=None)
        (ASGI middleware that intercepts requests and injects identity)

- module: adapters.annotations
  file: src/apcore_mcp/adapters/annotations.py
  classes:
    - AnnotationMapper:
        methods:
          - to_mcp_annotations(annotations) -> dict
          - has_requires_approval(annotations) -> bool

- module: adapters.schema
  file: src/apcore_mcp/adapters/schema.py
  classes:
    - SchemaConverter:
        methods:
          - convert_input_schema(descriptor) -> dict

- module: adapters.errors
  file: src/apcore_mcp/adapters/errors.py
  classes:
    - ErrorMapper:
        methods:
          - map_error(error) -> (content_list, is_error)

- module: adapters.id_normalizer
  file: src/apcore_mcp/adapters/id_normalizer.py
  classes:
    - ModuleIDNormalizer:
        methods:
          - normalize(module_id) -> str
          - denormalize(tool_name) -> str

- module: adapters.approval
  file: src/apcore_mcp/adapters/approval.py
  classes:
    - ElicitationApprovalHandler:
        (bridges MCP elicitation to apcore approval flow)

- module: converters.openai
  file: src/apcore_mcp/converters/openai.py
  classes:
    - OpenAIConverter:
        methods:
          - convert_registry(registry, embed_annotations=False, strict=False,
                             tags=None, prefix=None) -> list[dict]

- module: helpers
  file: src/apcore_mcp/helpers.py
  functions:
    - report_progress(context, progress, total=None, message=None) -> None [async]
    - elicit(context, message, requested_schema=None) -> ElicitResult | None [async]
  types:
    - ElicitResult: TypedDict { action: "accept"|"decline"|"cancel", content?: dict|None }
  constants:
    - MCP_PROGRESS_KEY: str = "_mcp_progress"
    - MCP_ELICIT_KEY: str = "_mcp_elicit"

- module: constants
  file: src/apcore_mcp/constants.py
  constants:
    - REGISTRY_EVENTS: dict = { "REGISTER": "register", "UNREGISTER": "unregister" }
    - ERROR_CODES: dict = { "MODULE_NOT_FOUND", "SCHEMA_VALIDATION_ERROR", "ACL_DENIED",
        "CALL_DEPTH_EXCEEDED", "CIRCULAR_CALL", "CALL_FREQUENCY_EXCEEDED",
        "INTERNAL_ERROR", "MODULE_TIMEOUT", "MODULE_LOAD_ERROR",
        "MODULE_EXECUTE_ERROR", "GENERAL_INVALID_INPUT", "APPROVAL_DENIED",
        "APPROVAL_TIMEOUT", "APPROVAL_PENDING", "VERSION_INCOMPATIBLE",
        "ERROR_CODE_COLLISION", "EXECUTION_CANCELLED" }
    - MODULE_ID_PATTERN: regex = r"^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$"

- module: explorer
  file: src/apcore_mcp/explorer/
  functions:
    - create_explorer_mount(tools, router, allow_execute, explorer_prefix, authenticator) -> Mount
  (Web UI for browsing and testing MCP tools)

- module: cli
  file: src/apcore_mcp/__main__.py
  functions:
    - main() -> None (CLI entry point: argparse-based, launches serve())
```

### ERROR_HIERARCHY

```
ERROR_HIERARCHY:
  base: Uses apcore.errors base classes (not redefined in MCP bridge)
  codes: ERROR_CODES dict with 17 values (see constants module above)
  classes:
    - ErrorMapper maps apcore errors to MCP TextContent error responses
```

### EXTENSION_POINTS

```
EXTENSION_POINTS:
  - Authenticator: authenticate(headers: dict[str, str]) -> Identity | None
  - MetricsExporter: (interface for custom metrics backends)
  - output_formatter: Callable (dict) -> str (custom result formatting)
  - approval_handler: (pluggable approval flow handler)
```

### EXAMPLES

```
EXAMPLES:
  - examples/run.py: Main entry point — launches MCP server with class-based
    extensions + binding.yaml modules, optional JWT auth, and Explorer UI
    on streamable-http transport.
  - examples/extensions/greeting.py: Simple greeting module with style
    parameter (friendly/formal/pirate). Demonstrates class-based module
    with Pydantic input/output schemas and ModuleAnnotations.
  - examples/extensions/math_calc.py: Basic arithmetic calculator module
    (add/sub/mul/div). Demonstrates readonly+idempotent annotations and
    input validation with error handling.
  - examples/extensions/text_echo.py: Minimal text echo module with
    optional uppercase transform. Demonstrates simplest possible
    read-only module pattern.
  - examples/binding_demo/run.py: Launches MCP server using ONLY
    binding.yaml modules (zero code intrusion). Demonstrates
    BindingLoader + Registry integration.
  - examples/binding_demo/extensions/convert_temperature.binding.yaml:
    Zero-code binding mapping myapp.convert_temperature to an apcore
    module with auto_schema inference from type hints.
  - examples/binding_demo/extensions/word_count.binding.yaml:
    Zero-code binding mapping myapp.word_count to an apcore module.
  - examples/README.md: Setup and usage instructions for running examples.
```

### TESTS

```
TESTS:
  structure:
    - tests/ (root-level test files)
    - tests/adapters/ (adapter unit tests)
    - tests/auth/ (authentication tests)
    - tests/converters/ (converter tests)
    - tests/e2e/ (end-to-end tests)
    - tests/explorer/ (explorer UI tests)
    - tests/integration/ (cross-component integration tests)
    - tests/performance/ (benchmark tests)
    - tests/security/ (security tests)
    - tests/server/ (server implementation tests)

  files:
    - tests/conftest.py: Shared fixtures — ModuleAnnotations, ModuleExample,
      ModuleDescriptor stubs; reusable descriptor fixtures (simple, empty_schema,
      nested_schema, destructive, no_annotations, all_types)
    - tests/test_api.py: Public API tests for serve() and to_openai_tools()
    - tests/test_apcore_mcp.py: APCoreMCP convenience class tests
    - tests/test_async_serve.py: async_serve() context manager tests
    - tests/test_cli.py: CLI entry point (__main__.py) argument parsing and dispatch
    - tests/test_helpers.py: report_progress() and elicit() helper function tests
    - tests/test_logging.py: Logging configuration and output tests
    - tests/test_serve_params.py: serve() parameter validation edge cases
    - tests/adapters/test_annotations.py: AnnotationMapper unit tests
    - tests/adapters/test_approval.py: ElicitationApprovalHandler tests
    - tests/adapters/test_errors.py: ErrorMapper unit tests
    - tests/adapters/test_errors_integration.py: Error mapping integration tests
    - tests/adapters/test_id_normalizer.py: ModuleIDNormalizer tests
    - tests/adapters/test_schema.py: SchemaConverter tests
    - tests/auth/test_jwt.py: JWTAuthenticator unit tests
    - tests/auth/test_middleware.py: AuthMiddleware ASGI tests
    - tests/auth/test_integration.py: Auth end-to-end integration tests
    - tests/converters/test_openai.py: OpenAIConverter tests
    - tests/e2e/test_e2e.py: Full end-to-end server lifecycle tests
    - tests/explorer/test_explorer.py: Explorer UI route and HTML tests
    - tests/integration/test_integration.py: Cross-component integration tests
    - tests/performance/test_benchmarks.py: Performance benchmark tests
    - tests/security/test_security.py: Security-focused tests (auth bypass, injection)
    - tests/server/test_factory.py: MCPServerFactory unit tests
    - tests/server/test_listener.py: RegistryListener tests
    - tests/server/test_metrics_endpoint.py: Prometheus /metrics endpoint tests
    - tests/server/test_router.py: ExecutionRouter unit tests
    - tests/server/test_router_stream.py: ExecutionRouter streaming tests
    - tests/server/test_server.py: MCPServer lifecycle tests
    - tests/server/test_transport.py: TransportManager tests

  total_count: 28
```

---

## Step 3: Confirm Tech Stack (Questions for Rust)

The skill instructs to present these choices via `AskUserQuestion`:

```
Rust MCP Bridge — Tech Stack Confirmation:

1. Rust edition: "2021 (Recommended)" / "2024"
2. Async runtime: "tokio (Recommended)" / "async-std" / "None (sync only)"
3. Serialization: "serde (Recommended)" / "Other"
4. Schema: "schemars (Recommended)" / "Other"
```

**Additional questions for MCP bridge specifically** (inferred from the reference):
- MCP SDK: Use `rmcp` crate or build from scratch?
- HTTP framework: `axum (Recommended)` / `actix-web` / `warp`?
- JWT library: `jsonwebtoken (Recommended)` / `jwt-simple` / Other?

**Simulated user response (assumed defaults):**
```
tech_stack:
  edition: "2021"
  async_runtime: "tokio"
  serialization: "serde"
  schema: "schemars"
  http_framework: "axum"
  mcp_sdk: "rmcp"
  jwt_library: "jsonwebtoken"
```

---

## Step 4: Scaffold Project (Complete Project Structure)

Target: `/Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/`

```
apcore-mcp-rust/
├── Cargo.toml                              # Workspace/package config with all dependencies
├── .gitignore                              # Rust patterns: /target, Cargo.lock (lib), etc.
├── README.md                               # Project name, description, installation, link to docs
├── CHANGELOG.md                            # Empty "## [Unreleased]" section
├── LICENSE                                 # MIT (matching ecosystem)
├── src/
│   ├── lib.rs                              # Crate root — pub mod declarations, re-exports for all 24 symbols
│   ├── server/
│   │   ├── mod.rs                          # pub mod factory, server, router, transport, listener
│   │   ├── factory.rs                      # McpServerFactory stub: create_server, build_tool, build_tools,
│   │   │                                   #   register_handlers, register_resource_handlers, build_init_options
│   │   ├── server.rs                       # McpServer stub: lifecycle wrapper around low-level MCP server
│   │   ├── router.rs                       # ExecutionRouter stub: handle_call() async
│   │   ├── transport.rs                    # TransportManager stub: run_stdio, run_streamable_http, run_sse,
│   │   │                                   #   build_streamable_http_app; MetricsExporter trait
│   │   └── listener.rs                     # RegistryListener stub: dynamic tool registration
│   ├── auth/
│   │   ├── mod.rs                          # pub mod protocol, jwt, middleware
│   │   ├── protocol.rs                     # Authenticator trait: authenticate(&self, headers) -> Option<Identity>
│   │   ├── jwt.rs                          # JwtAuthenticator stub: new(key, algorithms, claim_mapping),
│   │   │                                   #   authenticate(); ClaimMapping struct
│   │   └── middleware.rs                   # AuthMiddleware stub: ASGI-equivalent tower middleware
│   ├── adapters/
│   │   ├── mod.rs                          # pub mod annotations, approval, errors, id_normalizer, schema
│   │   ├── annotations.rs                  # AnnotationMapper stub: to_mcp_annotations, has_requires_approval
│   │   ├── approval.rs                     # ElicitationApprovalHandler stub
│   │   ├── errors.rs                       # ErrorMapper stub: map_error
│   │   ├── id_normalizer.rs                # ModuleIdNormalizer stub: normalize, denormalize
│   │   └── schema.rs                       # SchemaConverter stub: convert_input_schema
│   ├── converters/
│   │   ├── mod.rs                          # pub mod openai
│   │   └── openai.rs                       # OpenAiConverter stub: convert_registry
│   ├── cli.rs                              # CLI entry point stub — clap-based argument parsing, calls serve()
│   ├── helpers.rs                          # report_progress, elicit async fns; ElicitResult enum;
│   │                                       #   MCP_PROGRESS_KEY, MCP_ELICIT_KEY constants
│   ├── constants.rs                        # REGISTRY_EVENTS, ERROR_CODES enum, MODULE_ID_PATTERN (regex)
│   ├── utils.rs                            # resolve_executor, resolve_registry helper functions
│   ├── version.rs                          # VERSION constant = "0.1.0"
│   └── explorer/
│       ├── mod.rs                          # pub mod html, routes
│       ├── html.rs                         # Explorer HTML template rendering stubs
│       └── routes.rs                       # create_explorer_mount, route handlers stubs
├── tests/
│   ├── common/
│   │   └── mod.rs                          # Shared test fixtures: ModuleAnnotations, ModuleExample,
│   │                                       #   ModuleDescriptor structs; factory fns: simple_descriptor(),
│   │                                       #   empty_schema_descriptor(), nested_schema_descriptor(),
│   │                                       #   destructive_descriptor(), no_annotations_descriptor(),
│   │                                       #   all_types_descriptor()
│   ├── test_api.rs                         # Tests for serve() and to_openai_tools() public API
│   │                                       #   - test_serve_creates_server_with_defaults [FAIL stub]
│   │                                       #   - test_serve_rejects_empty_name [FAIL stub]
│   │                                       #   - test_to_openai_tools_converts_registry [FAIL stub]
│   │                                       #   - test_to_openai_tools_filters_by_tags [FAIL stub]
│   ├── test_apcore_mcp.rs                  # APCoreMCP convenience wrapper tests
│   │                                       #   - test_apcore_mcp_builder_pattern [FAIL stub]
│   ├── test_async_serve.rs                 # async_serve() context manager equivalent tests
│   │                                       #   - test_async_serve_yields_app [FAIL stub]
│   │                                       #   - test_async_serve_cleanup_on_drop [FAIL stub]
│   ├── test_cli.rs                         # CLI argument parsing and dispatch tests
│   │                                       #   - test_cli_default_args [FAIL stub]
│   │                                       #   - test_cli_custom_transport [FAIL stub]
│   │                                       #   - test_cli_extensions_dir [FAIL stub]
│   ├── test_helpers.rs                     # report_progress() and elicit() tests
│   │                                       #   - test_report_progress_with_callback [FAIL stub]
│   │                                       #   - test_report_progress_noop_without_callback [FAIL stub]
│   │                                       #   - test_elicit_returns_result [FAIL stub]
│   │                                       #   - test_elicit_returns_none_without_callback [FAIL stub]
│   ├── test_logging.rs                     # Logging configuration tests
│   │                                       #   - test_log_level_setting [FAIL stub]
│   ├── test_serve_params.rs                # serve() parameter validation edge cases
│   │                                       #   - test_serve_rejects_invalid_log_level [FAIL stub]
│   │                                       #   - test_serve_rejects_empty_tag [FAIL stub]
│   │                                       #   - test_serve_rejects_empty_prefix [FAIL stub]
│   │                                       #   - test_explorer_prefix_must_start_with_slash [FAIL stub]
│   ├── adapters/
│   │   ├── mod.rs                          # Adapter test module declarations
│   │   ├── test_annotations.rs             # AnnotationMapper tests
│   │   │                                   #   - test_readonly_maps_to_read_only_hint [FAIL stub]
│   │   │                                   #   - test_destructive_maps_correctly [FAIL stub]
│   │   │                                   #   - test_none_annotations_default [FAIL stub]
│   │   ├── test_approval.rs                # ElicitationApprovalHandler tests
│   │   │                                   #   - test_approval_bridges_elicitation [FAIL stub]
│   │   ├── test_errors.rs                  # ErrorMapper tests
│   │   │                                   #   - test_map_module_not_found [FAIL stub]
│   │   │                                   #   - test_map_schema_validation_error [FAIL stub]
│   │   ├── test_errors_integration.rs      # Error mapping integration tests
│   │   │                                   #   - test_error_roundtrip [FAIL stub]
│   │   ├── test_id_normalizer.rs           # ModuleIdNormalizer tests
│   │   │                                   #   - test_normalize_dotted_id [FAIL stub]
│   │   │                                   #   - test_denormalize_roundtrip [FAIL stub]
│   │   └── test_schema.rs                  # SchemaConverter tests
│   │                                       #   - test_convert_simple_schema [FAIL stub]
│   │                                       #   - test_convert_nested_schema_with_defs [FAIL stub]
│   │                                       #   - test_convert_empty_schema [FAIL stub]
│   ├── auth/
│   │   ├── mod.rs                          # Auth test module declarations
│   │   ├── test_jwt.rs                     # JwtAuthenticator tests
│   │   │                                   #   - test_valid_token_returns_identity [FAIL stub]
│   │   │                                   #   - test_expired_token_returns_none [FAIL stub]
│   │   │                                   #   - test_invalid_signature_returns_none [FAIL stub]
│   │   ├── test_middleware.rs              # AuthMiddleware tests
│   │   │                                   #   - test_authenticated_request_passes [FAIL stub]
│   │   │                                   #   - test_unauthenticated_returns_401 [FAIL stub]
│   │   │                                   #   - test_exempt_path_bypasses_auth [FAIL stub]
│   │   └── test_integration.rs             # Auth integration tests
│   │                                       #   - test_full_auth_flow [FAIL stub]
│   ├── converters/
│   │   ├── mod.rs                          # Converter test module declarations
│   │   └── test_openai.rs                  # OpenAiConverter tests
│   │                                       #   - test_convert_simple_module [FAIL stub]
│   │                                       #   - test_strict_mode_adds_flag [FAIL stub]
│   │                                       #   - test_embed_annotations [FAIL stub]
│   ├── e2e/
│   │   ├── mod.rs                          # E2E test module declarations
│   │   └── test_e2e.rs                     # End-to-end server lifecycle tests
│   │                                       #   - test_server_start_and_stop [FAIL stub]
│   │                                       #   - test_tool_call_roundtrip [FAIL stub]
│   ├── explorer/
│   │   ├── mod.rs                          # Explorer test module declarations
│   │   └── test_explorer.rs                # Explorer UI route tests
│   │                                       #   - test_explorer_mount_returns_html [FAIL stub]
│   │                                       #   - test_explorer_lists_tools [FAIL stub]
│   ├── integration/
│   │   ├── mod.rs                          # Integration test module declarations
│   │   └── test_integration.rs             # Cross-component integration tests
│   │                                       #   - test_registry_to_tool_to_execution [FAIL stub]
│   │                                       #   - test_multiple_transports [FAIL stub]
│   ├── performance/
│   │   ├── mod.rs                          # Performance test module declarations
│   │   └── test_benchmarks.rs              # Performance benchmark tests
│   │                                       #   - test_build_tools_performance [FAIL stub]
│   │                                       #   - test_schema_conversion_performance [FAIL stub]
│   ├── security/
│   │   ├── mod.rs                          # Security test module declarations
│   │   └── test_security.rs                # Security-focused tests
│   │                                       #   - test_auth_bypass_attempt [FAIL stub]
│   │                                       #   - test_injection_in_module_id [FAIL stub]
│   └── server/
│       ├── mod.rs                          # Server test module declarations
│       ├── test_factory.rs                 # McpServerFactory tests
│       │                                   #   - test_create_server [FAIL stub]
│       │                                   #   - test_build_tool_from_descriptor [FAIL stub]
│       │                                   #   - test_build_tools_skips_missing_definition [FAIL stub]
│       │                                   #   - test_register_handlers [FAIL stub]
│       │                                   #   - test_register_resource_handlers [FAIL stub]
│       ├── test_listener.rs                # RegistryListener tests
│       │                                   #   - test_listener_receives_register_event [FAIL stub]
│       ├── test_metrics_endpoint.rs        # Metrics endpoint tests
│       │                                   #   - test_metrics_endpoint_returns_prometheus_format [FAIL stub]
│       ├── test_router.rs                  # ExecutionRouter tests
│       │                                   #   - test_route_to_correct_module [FAIL stub]
│       │                                   #   - test_route_unknown_module_returns_error [FAIL stub]
│       │                                   #   - test_validate_inputs_flag [FAIL stub]
│       ├── test_router_stream.rs           # ExecutionRouter streaming tests
│       │                                   #   - test_streaming_progress_notifications [FAIL stub]
│       ├── test_server.rs                  # McpServer lifecycle tests
│       │                                   #   - test_server_lifecycle [FAIL stub]
│       └── test_transport.rs               # TransportManager tests
│                                           #   - test_stdio_transport [FAIL stub]
│                                           #   - test_streamable_http_transport [FAIL stub]
│                                           #   - test_sse_transport [FAIL stub]
│                                           #   - test_unknown_transport_error [FAIL stub]
├── examples/
│   ├── run.rs                              # Main entry point — starts MCP server with class-based
│   │                                       #   extensions + binding YAML, optional JWT auth, Explorer UI.
│   │                                       #   Equivalent of Python's examples/run.py.
│   │                                       #   COMPLETE RUNNABLE CODE (not a stub).
│   ├── README.md                           # Setup and usage instructions
│   ├── extensions/
│   │   ├── mod.rs                          # Module declarations for example extensions
│   │   ├── greeting.rs                     # Greeting extension: struct with execute(),
│   │   │                                   #   GreetingInput/GreetingOutput, style parameter
│   │   │                                   #   (friendly/formal/pirate). COMPLETE CODE.
│   │   ├── math_calc.rs                    # MathCalc extension: add/sub/mul/div with validation.
│   │   │                                   #   COMPLETE CODE.
│   │   └── text_echo.rs                    # TextEcho extension: echo with optional uppercase.
│   │                                       #   COMPLETE CODE.
│   └── binding_demo/
│       ├── run.rs                          # Binding-only demo — loads .binding.yaml files,
│       │                                   #   starts MCP server with Explorer. COMPLETE CODE.
│       └── extensions/
│           ├── convert_temperature.binding.yaml  # Copied from reference — zero-code binding
│           └── word_count.binding.yaml           # Copied from reference — zero-code binding
└── .code-forge.json                        # Code-forge configuration for planning
```

**File counts:**
- Source files: 23 (in `src/`)
- Test files: 28 (in `tests/`, mirroring reference 1:1)
- Example files: 9 (in `examples/`)
- Config/meta: 6 (Cargo.toml, .gitignore, README.md, CHANGELOG.md, LICENSE, .code-forge.json)
- **Total: 66 files**

### Test Stub Example (what test_helpers.rs would contain)

```rust
//! Tests for report_progress() and elicit() helper functions.
//!
//! Reference: apcore-mcp-python/tests/test_helpers.py

mod common;

use apcore_mcp_rust::helpers::{elicit, report_progress, ElicitResult, MCP_ELICIT_KEY, MCP_PROGRESS_KEY};

#[tokio::test]
async fn test_report_progress_with_callback() {
    // TODO: Create mock context with progress callback, call report_progress,
    //       verify callback received (progress, total, message).
    panic!("not implemented — TDD red phase");
}

#[tokio::test]
async fn test_report_progress_noop_without_callback() {
    // TODO: Create context without MCP_PROGRESS_KEY, call report_progress,
    //       verify no panic and silent no-op.
    panic!("not implemented — TDD red phase");
}

#[tokio::test]
async fn test_elicit_returns_result() {
    // TODO: Create mock context with elicit callback that returns Accept,
    //       verify ElicitResult { action: Accept, content: Some(...) }.
    panic!("not implemented — TDD red phase");
}

#[tokio::test]
async fn test_elicit_returns_none_without_callback() {
    // TODO: Create context without MCP_ELICIT_KEY, call elicit,
    //       verify returns None.
    panic!("not implemented — TDD red phase");
}
```

### Example File (what examples/run.rs would contain)

```rust
//! Launch MCP server with example extensions and optional JWT auth.
//!
//! Usage:
//!     cargo run --example run
//!
//! Then open http://127.0.0.1:8000/explorer/ in your browser.
//!
//! Enable JWT authentication:
//!     JWT_SECRET=my-secret cargo run --example run

use std::env;

use apcore::Registry;
use apcore_mcp_rust::{serve, JwtAuthenticator, ServeConfig};

mod extensions;

fn main() {
    // 1. Create registry and discover class-based modules
    let mut registry = Registry::with_extensions_dir("./examples/extensions");
    let n_class = registry.discover();

    // 2. Load binding.yaml modules
    let binding_modules = apcore::BindingLoader::new()
        .load_binding_dir("./examples/binding_demo/extensions", &mut registry);

    println!("Class-based modules: {}", n_class);
    println!("Binding modules:     {}", binding_modules.len());
    println!("Total:               {}", registry.module_ids().len());

    // 3. Build JWT authenticator if JWT_SECRET is set
    let authenticator = env::var("JWT_SECRET").ok().map(|secret| {
        println!("JWT authentication:  enabled (HS256)");
        JwtAuthenticator::new(&secret, vec!["HS256"], None)
    });

    // 4. Launch MCP server with Explorer UI
    serve(
        &registry,
        ServeConfig::builder()
            .transport("streamable-http")
            .host("127.0.0.1")
            .port(8000)
            .explorer(true)
            .allow_execute(true)
            .authenticator(authenticator)
            .build(),
    );
}
```

---

## Step 5-6: Feature Specs and Code-Forge Config

(Skipped in simulation — would check for `{protocol_path}/docs/features/*.md` and generate `.code-forge.json`.)

---

## Step 7: Summary Output

```
apcore-skills:sdk — SDK Bootstrap Complete

Target: /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
Language: rust
Type: mcp
Modules: 23 source files scaffolded
Tests: 28 test stubs (TDD red phase)
Examples: 9 runnable examples
Feature specs: (pending — check protocol docs)
API contract: 24 public symbols to implement

Project structure:
  apcore-mcp-rust/
  ├── Cargo.toml
  ├── src/
  │   ├── lib.rs               (24 re-exports)
  │   ├── server/              (factory, server, router, transport, listener)
  │   ├── auth/                (protocol trait, jwt, middleware)
  │   ├── adapters/            (annotations, approval, errors, id_normalizer, schema)
  │   ├── converters/          (openai)
  │   ├── explorer/            (html, routes)
  │   ├── cli.rs
  │   ├── helpers.rs
  │   ├── constants.rs
  │   ├── utils.rs
  │   └── version.rs
  ├── tests/
  │   ├── common/mod.rs        (shared fixtures)
  │   ├── test_api.rs          (public API)
  │   ├── test_cli.rs          (CLI)
  │   ├── test_helpers.rs      (helpers)
  │   ├── adapters/            (6 test files)
  │   ├── auth/                (3 test files)
  │   ├── converters/          (1 test file)
  │   ├── e2e/                 (1 test file)
  │   ├── explorer/            (1 test file)
  │   ├── integration/         (1 test file)
  │   ├── performance/         (1 test file)
  │   ├── security/            (1 test file)
  │   └── server/              (7 test files)
  └── examples/
      ├── run.rs               (main server with extensions)
      ├── extensions/          (greeting, math_calc, text_echo)
      └── binding_demo/        (run.rs + .binding.yaml files)

Next steps:
  cd /Users/tercel/WorkSpace/aipartnerup/apcore-mcp-rust/
  git init && git add . && git commit -m "chore: initialize apcore-mcp-rust project skeleton"
  /code-forge:port @apcore/PROTOCOL_SPEC.md --ref apcore-mcp-python --lang rust    Generate implementation plans
  /code-forge:impl server.factory                                                    Start implementing
  /apcore-skills:sync --lang rust,python                                             Verify API consistency
```

---

## Analysis: Skill Effectiveness for Test Stubs and Examples

### What the skill instructions got right

1. **Test mirroring works well.** The Step 4 MCP bridge template says to create
   `tests/server/`, `tests/auth/`, `tests/adapters/`, `tests/converters/`, and
   `tests/explorer/` directories, and the "Reference Sync" section instructs the
   sub-agent to read the reference `tests/` and mirror its structure. The reference
   has exactly these directories plus `e2e/`, `integration/`, `performance/`, and
   `security/`. Following the skill, all 28 test files from the reference get a
   corresponding Rust stub.

2. **TDD red phase is explicit.** The instruction "One failing stub test per public
   method: asserts `False` / `expect(false)` / `t.Fatal("not implemented")`" translates
   cleanly to Rust's `panic!("not implemented")` idiom. Each test file would contain
   commented-out TODOs describing what to test, with a hard failure to enforce TDD.

3. **Examples are ported, not stubbed.** The skill distinguishes between "test stubs"
   (TDD red phase, intentionally failing) and "example files" (complete, runnable code).
   This is correct -- examples/run.rs should be fully functional from day one.

4. **Shared fixtures are handled.** The skill explicitly calls for a `{helpers-file}`
   with "shared fixtures and utilities," which maps to `tests/common/mod.rs` in Rust.

### Gaps and areas for improvement

1. **Missing test directories in the MCP template.** The skill's MCP bridge test
   template (Step 4) only lists: `server/`, `auth/`, `adapters/`, `converters/`,
   `explorer/`. But the reference also has `e2e/`, `integration/`, `performance/`,
   and `security/` test directories. The "Reference Sync" instruction catches these
   ("Preserve the subdirectory organization"), but only because it says to read the
   actual reference tests. If the reference were unavailable, these directories would
   be missed. **Recommendation:** Add `e2e/`, `integration/`, `performance/`, and
   `security/` to the MCP template explicitly.

2. **Root-level test files not enumerated.** The template lists `test_api.{ext}`,
   `test_cli.{ext}`, and `test_helpers.{ext}` but the reference also has
   `test_apcore_mcp.py`, `test_async_serve.py`, `test_logging.py`, and
   `test_serve_params.py`. Again, "Reference Sync" catches these, but the template
   itself is incomplete. **Recommendation:** Add the missing root-level test files
   to the MCP template or add a catch-all instruction like "one test file per
   root-level source module."

3. **Binding demo example structure.** The template correctly shows
   `examples/binding_demo/run.{ext}` and `examples/binding_demo/extensions/`, but
   does not mention the `myapp.py` equivalent (the target of the bindings). In the
   Python reference, `myapp.py` contains the plain functions that `.binding.yaml`
   files wrap. For Rust, this would need a `myapp.rs` or equivalent crate.
   **Recommendation:** Add `{binding_target_module}` to the binding_demo template.

4. **Rust-specific test organization.** The skill does not account for Rust's
   integration test convention where `tests/*.rs` files are separate crates. The
   `tests/common/mod.rs` pattern is correct, but `tests/adapters/mod.rs` (as a
   nested module within an integration test) requires specific Rust knowledge.
   An alternative would be to use `tests/adapters_test_annotations.rs` flat
   naming. The skill leaves this to the sub-agent's language knowledge, which
   is acceptable but could be made more explicit.

5. **No mention of `#[cfg(test)]` unit tests.** Rust idiom often puts unit tests
   in the same file as the source code (`#[cfg(test)] mod tests { ... }`). The
   skill only creates external test files in `tests/`. This is fine for integration
   tests but misses the Rust convention of co-located unit tests.
   **Recommendation:** Mention that source stubs should include a `#[cfg(test)]`
   module with at least one placeholder test.

### Verdict

The skill produces a comprehensive and accurate scaffold. The test and example
coverage is strong because the "Reference Sync" sub-section in Step 4 instructs
the sub-agent to read and mirror the reference implementation's actual test and
example structure, rather than relying solely on the template. The template
provides a solid baseline, and the reference sync fills in the gaps. The main
risk is that without a reference, the template alone would miss several test
directories and root-level test files for the MCP bridge type.
