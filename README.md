# intent-compiler

**Deterministic Markdown → structured artifacts.**

`intent-compiler` transforms Markdown protocols into typed, validated artifacts — mocks for tests, forms for users, and CI-enforced contracts. One source of truth.

## Why intent-compiler

Prompt workflows drift silently. Your AI breaks without warning. This toolkit solves that:

- **Schema as source of truth** — Define once, generate everything
- **Type-safe slots** — Wrong type injection fails at parse time
- **Schema validation** — LLM output validated against JSON Schema Draft 7
- **Semantic versioning** — Breaking changes are explicit
- **CI-friendly** — Gate every PR, know exactly what your model will do

## Protocol Format

A protocol file (`.md`) contains:

```markdown
---
Version: 1.0.0
Model: anthropic/claude-sonnet-4
Author: platform-team
Schema:
  type: object
  properties:
    username: { type: string, minLength: 3 }
    email: { type: string, format: email }
  required: [username, email]
---

## Context
You are a user registration handler.

## Slots
{{username}} string — unique username
{{email}} string — valid email address

## Constraints
1. Username must be alphanumeric.
2. Email must be valid format.
```

## Quick Start

```bash
pip install pyyaml jsonschema
```

### CLI Commands

```bash
# Lint protocols
intent lint protocols/

# Resolve to artifact
intent resolve protocol.md --output json

# Generate mocks from schema
intent generate protocol.md --mock --count 3

# Generate UI form from schema
intent generate protocol.md --ui --out form.html
```

### Programmatic Usage

```python
from intent_compiler.parser import ProtocolParser
from intent_compiler.validator import SchemaValidator
from intent_compiler.generators.mock_generator import generate_mock
from intent_compiler.generators.ui_generator import generate_ui

# Parse protocol
protocol = ProtocolParser().parse_file("protocol.md")
schema = protocol["frontmatter"]["schema"]

# Validate LLM output
validator = SchemaValidator()
result = validator.validate_output(llm_output, schema)

# Generate mock data
mock = generate_mock(schema)

# Generate UI form
html = generate_ui(schema, title="User Form")
```

## Features

### Type-Safe Slots
```python
# Wrong type? Fails at parse time, not runtime.
{{user_count}} int  # must be integer
{{name}} string     # must be string
```

### Schema Validation
```bash
$ intent lint output.json
✓ valid — schema matches, types correct
```

### Semantic Versioning
```bash
# Breaking change? Bump major version.
# protocol.md@1.0.0 → protocol.md@2.0.0
```

### CI Integration
```bash
# .github/workflows/quality.yml
- name: Lint Protocols
  run: intent lint protocols/ --strict
# Exit 0 = merge allowed, Exit 1 = PR blocked
```

## Repository Layout

```
intent-compiler/
├── cli.py              # CLI: lint, resolve, generate
├── parser.py           # Protocol Markdown parser
├── validator.py        # JSON Schema validation
├── mock_generator.py    # Generate mock data from schema
├── ui_generator.py     # Generate HTML forms from schema
├── lint_config.py      # Configurable lint rules
├── protocols/           # Sample protocols
│   ├── valid_protocol.md
│   ├── invalid_protocol.md
│   └── user_profile.md
├── test_mvp.py         # Test suite
├── index.html          # Landing page
└── .github/workflows/  # CI/CD pipelines
```

## Roadmap

### Completed (Phase 1: Engine & Enterprise Architecture)
- [x] **Core Parsing:** AST-based Markdown parsing (`markdown-it-py`) with Type-Safe Slots.
- [x] **Validation:** JSON Schema Draft 7 integration natively without manual overhead.
- [x] **CLI Evolution:** Intuitive `Click` module commands (`intent lint`, `intent resolve`, `intent generate`).
- [x] **Outputs:** Emits structured artifacts (JSON, YAML, TypeScript, semantic prompts).
- [x] **Generators:** Scalable Mock API payloads & Jinja2-based HTML form templating.
- [x] **Developer Experience:** Multi-format CLI reporting (table, compact, JSON) with strict CI exit codes.
- [x] **Configuration:** `.intent.yaml` standard config with warnings/errors thresholds and semver spec validations.
- [x] **Testing:** Robust test suite coverage via `Pytest` and dynamic fixtures.

### Planned (Phase 2: Ecosystem & Developer Tooling)
- [ ] **DX & Hardening:** Line-specific context in error messages, >90% test coverage, and complete SAST/typecheck CI pipelines.
- [ ] **Scaffolding:** `intent init` command to quickly bootstrap protocol templates.
- [ ] **Ecosystem:** Finalize PyPI publishing for the Python SDK and build a native TypeScript SDK (`npm`).
- [ ] **Editor Tooling:** Official VS Code Extension for on-the-fly markdown highlighting, autocomplete, and slot hovering.
- [ ] **Advanced Features:** Protocol composition (`import`/`extend` specs) and integration tests with live LLM Providers.
- [ ] **Strategic Positioning:** Re-evaluate messaging to heavily emphasize "Intent Compilation" strictly over traditional "Linting".

## Documentation

- `SPEC_FRONTMATTER.md` — Frontmatter specification
- `SPEC_PROTO_LINT.md` — Linting rules layout and behavior
- `index.html` — Live interactive visualizer and Demo landing page

## License

MIT
