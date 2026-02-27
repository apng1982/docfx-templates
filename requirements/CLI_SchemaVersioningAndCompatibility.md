# Requirements: Non-functional, versioning, and quality

## 1. Schema versioning
The CLI MUST embed `schemaVersion` in:
- `docset.yml`
- every entity YAML file

Versioning MUST follow Calendar Versioning: YYYY.MM.DD.HHMM

## 2. Backward compatibility policy (for your ecosystem)
Because Docusaurus plugins will consume the output, define:
- minimum supported schema versions per plugin release
- deprecation windows

## 3. Performance targets (suggested)
The CLI SHOULD:
- complete extraction for medium solutions in a reasonable time budget (to be defined by your team)
- support incremental regeneration:
  - avoid rewriting unchanged entity files
  - output stable file names and content hashing

## 4. Reliability
The CLI MUST:
- fail fast on unrecoverable build errors (configurable)
- produce partial output only if explicitly requested (e.g., `--allow-partial`), and mark manifest accordingly

## 5. Testability
The project MUST include:
- schema validation tests
- golden file tests for deterministic YAML output
- representative Azure Functions samples for each supported trigger type

## 6. Observability
The CLI SHOULD support:
- `--verbose` logging
- structured logs (JSON) for CI
- timing breakdown (build vs extraction vs write)

## 7. Security
- Do not emit secrets (see Azure Functions sanitization).
- Normalize file paths to avoid leaking absolute machine paths unless configured.
- Provide a `--path-mode relative|absolute` option, default `relative`.
