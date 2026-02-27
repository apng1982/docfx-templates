# Requirements: Conceptual data model (entities and relationships)

## 1. Guiding principles
- Every documented "thing" is an **Entity** with:
  - `kind` (type discriminator)
  - `uid` (stable identifier)
  - `name` and `displayName`
  - `summary/remarks` docs (when applicable)
  - `source` provenance
  - `refs` to other entities by UID
- Entity graph is navigable:
  - Project -> Assembly -> Namespace -> Type -> Member
  - Project -> Azure Function App -> Function -> Binding(s)

## 2. Entity kinds (required)
### 2.1 `Docset` (manifest)
Top-level entry referencing all generated content.

### 2.2 `Project`
Represents a .NET project (csproj) and its build contexts.
Relationships:
- has many `Assembly` (usually 1 per TFM)
- has many `AzureFunctionApp` (0 or 1 typically, but allow many)

### 2.3 `Assembly`
Represents a compiled assembly artifact for a given TFM.
Relationships:
- contains many `Namespace`
- contains many `Type` (directly or via namespaces)

### 2.4 `Namespace`
Namespace container.
Relationships:
- belongs to `Assembly`
- contains many `Type`

### 2.5 `Type`
Represents a named type:
- class, struct, interface, enum, delegate, record

Relationships:
- belongs to `Namespace`
- has base type, implemented interfaces
- has many members
- nested types are supported

### 2.6 `Member`
Represents a type member:
- method, constructor, property, field, event, operator, indexer

Relationships:
- belongs to `Type`
- has parameters (for callables and indexers)
- uses `TypeRef` for parameter/return/property types

### 2.7 `TypeRef`
A normalized representation of a type usage in signatures:
- supports generics, arrays, pointers, nullable, tuples
- supports external types (not in docset)

### 2.8 `DocComment`
Normalized form of extracted documentation content:
- structured fields + rich inline references
- supports code blocks and examples

### 2.9 `AttributeData`
Captured attribute (name + constructor args + named args) where needed.

### 2.10 `AzureFunctionApp`
Represents an Azure Functions application (a deployable function app boundary).
Relationships:
- belongs to `Project`
- contains many `AzureFunction`

### 2.11 `AzureFunction`
Represents one function entry point (HTTP trigger, timer trigger, queue trigger, etc.).
Relationships:
- belongs to `AzureFunctionApp`
- points to underlying `Member` (method) symbol via `entryPointUid`
- contains many `Binding` (trigger + inputs + outputs)

### 2.12 `Binding`
Represents a binding on a function:
- trigger binding (exactly 1 required)
- input bindings (0..n)
- output bindings (0..n)

Binding includes:
- `bindingType` (e.g., `httpTrigger`, `timerTrigger`, `serviceBusTrigger`, ...)
- `direction` (in/out/inout)
- `name` (binding name)
- strongly-typed `properties` map
- optionally `rawAttribute` and provenance

## 3. Cross-cutting concepts (required)
### 3.1 UID
A UID MUST:
- be unique within a docset
- be stable across runs given same symbol identity
- be URL-safe (or easily encoded)
- support overload disambiguation

Recommended conceptual shape:
- Types: `T:<namespace>.<typeName>{<arity>}` plus nesting markers
- Members: `M:<typeUid>.<memberName>(<paramTypeRefs>)-><returnTypeRef>` (return optional)
- Namespaces: `N:<namespace>`
- Projects: `P:<projectName>`
- Assemblies: `A:<assemblyName>@<tfm>`
- Azure functions: `F:<functionAppUid>/<functionName>`

(Exact formatting is an implementation detail, but the rules MUST be specified and versioned.)

### 3.2 Display names and signatures
For every type/member, store:
- `name` (simple identifier)
- `fullName` (namespace-qualified)
- `displayName` (suitable for page titles)
- `signature` forms:
  - `csharp` signature string
  - optional `il` or `metadata` signature
  - optional "friendly" simplified signature

### 3.3 Availability and conditionality
Entities MUST support:
- `tfm` availability list OR per-TFM variants
- `platform` or `os` restrictions if available (optional)
- `obsolete` metadata (message, isError)

### 3.4 Source and provenance
Entities MUST include:
- source file path (relative, normalized)
- source span (startLine/startColumn/endLine/endColumn) when available
- repository URL mapping (SourceLink) when available
- provenance for extracted Azure Functions metadata:
  - attribute-based (e.g., `[FunctionName]`, binding attributes)
  - host.json / function.json (if used)
  - other conventions

### 3.5 External references
Type/member references to external libraries MUST be representable:
- `external: true`
- `externalId` (optional)
- `package` info if known (NuGet package id/version)

## 4. Azure Functions domain model requirements
### 4.1 Function identity
A function MUST capture:
- `functionName` (as deployed; often from `[FunctionName]`)
- `entryPointUid` referencing the method member
- `trigger` binding (exactly one)

### 4.2 Trigger-specific metadata
The model MUST support trigger-specific fields to drive rendering decisions.
Examples:
- HTTP: routes, methods, auth level
- Timer: schedule, runOnStartup, useMonitor
- Service Bus: queue/topic, subscription, connection, session settings
- Storage Queue/Blob: queueName/container, connection, poison handling hints if discoverable
- Event Grid: topic endpoint type if available
- Cosmos DB: database/container, lease container, connection
- Durable Functions: orchestration/activity/entity roles (if applicable)

This implies bindings have a `bindingType` discriminator and a flexible `properties` bag, with a defined schema per known binding type.

### 4.3 Parameter-to-binding mapping
For each binding, capture:
- which method parameter (if any) it binds to
- the parameter type (TypeRef)
- direction and binding name

### 4.4 Return/output mapping
If the function uses return value as output:
- capture `usesReturnValue: true`
- map return type to output binding(s), if determinable

### 4.5 Security and auth (HTTP)
For HTTP triggers, capture:
- auth level (anonymous/function/admin)
- declared methods
- route template
- output response type hints (if documented via attributes or conventions)

(Do not attempt to infer authorization policies beyond what metadata declares.)

## 5. Relationships required by Docusaurus plugins
The emitted model MUST allow plugins to:
- Build sidebars:
  - by namespace
  - by assembly
  - by "area" tags (custom grouping)
  - by trigger type (Functions)
- Render pages:
  - type pages with member tables
  - member pages with parameter/return/exception sections
  - function pages with trigger-specific sections and binding tables
- Provide cross-links (`xref`) between entities using UIDs
