# Requirements: Doc Model Generator CLI (Replacement for DocFX data extraction)

## 1. Purpose
Create a new CLI that:
- Analyzes a .NET solution (or set of projects) and extracts an API + metadata model.
- Emits a **YAML-based documentation data set** (plus optional auxiliary artifacts) to disk.
- Is designed to be consumed by **custom Docusaurus plugins**, which will render a modern static site.
- Supports **standard .NET libraries/apps** and additionally supports **Azure Function Apps** by extracting trigger/binding metadata.

This is **not** a migration. No backward compatibility with DocFX output formats is required.

## 2. Core outputs (high level)
The CLI produces a "docset" directory containing:
- `docset.yml` (index/manifest for the whole output)
- One YAML file per documentation entity (or per logical unit) such as:
  - assembly / project
  - namespace
  - type
  - member (method/property/field/event)
  - Azure Function endpoint (function) and bindings
- Optional: search index inputs, tag maps, and diagnostics.

The Docusaurus side is responsible for:
- Routing / page generation
- Theming
- Navigation trees
- Rendering decisions based on metadata (e.g., trigger type)

## 3. Goals
- Be **deterministic**: same inputs produce identical outputs.
- Be **stable**: explicit schema versioning and deprecation policy.
- Be **composable**: incremental builds, partial regeneration, and modular extraction.
- Be **inspectable**: YAML is human-readable; include diagnostics and provenance (where data came from).
- Be **source-link friendly**: link types/members back to repository paths and line ranges when available.
- Be **Azure Functions aware**: include triggers/bindings and enough metadata for rendering function-specific docs.

## 4. Non-goals
- Rendering HTML (handled by Docusaurus plugins).
- Supporting DocFX plugins/templating.
- Supporting legacy project types that cannot be built/analyzed by modern .NET tooling unless explicitly added later.
- Implementing a full C# compiler—use official build/artifact pipelines to obtain symbols and docs.

## 5. Key concepts (vocabulary)
- **Docset**: The full generated output for a run.
- **Entity**: A documented item (type, member, namespace, function, etc.).
- **UID**: Stable identifier used for cross-references in YAML and by Docusaurus.
- **Ref**: A lightweight pointer to another entity by UID, plus display names.
- **Symbol**: Compiler-level representation (types/members) used during extraction.
- **Function**: Azure Function entry point with triggers/bindings.
- **Binding**: Azure Functions binding (trigger/input/output).
- **Provenance**: Source of a field (XML doc, attribute, reflection metadata, config file, etc.).
