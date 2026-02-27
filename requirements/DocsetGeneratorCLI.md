# Requirements: CLI behavior and workflow

## 1. Primary CLI commands
The CLI MUST provide at minimum:

### 1.1 `generate`
Generates a docset from one or more inputs.

**Inputs**
- Path to `.sln` OR list of `.csproj` paths
- Configuration:
  - build configuration (Debug/Release)
  - target framework(s)
  - output directory
  - inclusion/exclusion filters

**Outputs**
- A docset directory with:
  - `docset.yml` manifest
  - entity YAML files
  - optional diagnostics report(s)

**Options (minimum)**
- `--input <path>` (repeatable or accepts glob)
- `--output <path>`
- `--configuration <Debug|Release>`
- `--framework <tfm>` (repeatable; default: all that can be resolved)
- `--include <pattern>` / `--exclude <pattern>` (applies to projects/namespaces/types)
- `--public-only` (default true)
- `--include-internal` / `--include-private` (default false)
- `--nullable <enable|disable|preserve>` (how to represent nullability)
- `--source-link <auto|off|path>` (repo link generation strategy)
- `--functions <auto|on|off>` (Azure Functions extraction)
- `--schema-version <semver>` (pins emitted schema)

### 1.2 `validate`
Validates generated YAML against the schema and consistency rules.

**Validations**
- YAML schema validation
- UID uniqueness
- Reference integrity (all refs point to existing UIDs unless explicitly marked external)
- Deterministic ordering constraints

### 1.3 `schema`
Emits the JSON Schema (or equivalent) for all YAML entity types and the docset manifest.

### 1.4 `diff` (recommended)
Compares two docsets and outputs a machine-readable summary:
- added/removed/changed entities
- signature changes
- doc changes

## 2. Build / analysis requirements
The CLI MUST:
- Resolve compilation for projects to obtain accurate symbol info:
  - names, signatures, generic parameters, constraints
  - attributes
  - accessibility
  - source locations (when possible)
- Extract XML documentation comments:
  - `<summary>`, `<remarks>`, `<param>`, `<returns>`, `<exception>`, `<typeparam>`, `<see>`, `<seealso>`, `<example>`, `<value>`
- Handle multi-targeting:
  - either emit per-TFM variants OR merge with per-TFM availability annotations.

The CLI SHOULD:
- Support incremental rebuilds (skip unchanged projects).
- Cache intermediate symbol graphs keyed by inputs + compiler options.

## 3. Filtering requirements
The CLI MUST support:
- Excluding by accessibility (public/internal/private).
- Excluding by namespace patterns.
- Excluding by attribute (e.g., `[Obsolete]`, custom `[ExcludeFromDocs]`).
- Excluding compiler-generated members by default (configurable).

## 4. Determinism requirements
Output MUST be deterministic:
- Stable ordering (alphabetical or declared order, but consistent and specified).
- Stable UID generation rules.
- Stable formatting for signatures and type names.

## 5. Diagnostics and transparency
The CLI MUST produce diagnostics:
- Unresolved references (e.g., `<see cref="...">` not resolvable)
- Ambiguous type matches
- Missing XML docs (optionally warnings)
- Azure Functions metadata extraction failures

Diagnostics MUST include:
- severity (info/warn/error)
- file path + line/column when available
- associated entity UID when relevant

## 6. Extensibility requirements
The CLI SHOULD support a plug-in or extension mechanism for:
- Adding new entity kinds (e.g., REST endpoints, message contracts)
- Adding new metadata collectors (e.g., additional Azure binding types)
- Custom UID strategies (careful: stability implications)
