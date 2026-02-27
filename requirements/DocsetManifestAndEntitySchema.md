# Requirements: YAML schema and file layout

## 1. Output directory layout
A generated docset directory MUST contain:

- `docset.yml` (manifest)
- `entities/` folder containing entity YAML files (or another stable folder name)
- Optional:
  - `assets/` for copied images or static artifacts (if supported later)
  - `diagnostics/diagnostics.yml` (or JSON) for warnings/errors
  - `schema/` containing schemas used (pinned version)

Example layout:
- docset.yml
- entities/
  - N.MyCompany.Core.yml
  - T.MyCompany.Core.Widget.yml
  - M.MyCompany.Core.Widget.DoWork(System.String).yml
  - F.P.MyFuncApp/HealthCheck.yml

## 2. YAML document conventions
- UTF-8
- Deterministic ordering of keys (documented and consistent)
- All entities MUST include:
  - `schemaVersion`
  - `kind`
  - `uid`
  - `name`
  - `displayName`
  - `refs` (can be empty)
  - `source` (can be null if unknown)
- Avoid anchors/aliases to keep processing simple across YAML libraries.

## 3. Required entity schemas (conceptual)
Below are conceptual fields. Exact schema encoding can be JSON Schema, OpenAPI, or custom validation rules, but it MUST be versioned.

### 3.1 `docset.yml` (manifest)
Required fields:
- `schemaVersion`
- `generator` (name + version)
- `generatedAt` (ISO 8601)
- `inputs` (list of project/solution paths, normalized)
- `projects` (list of project refs: uid + path)
- `entitiesIndex`:
  - `count`
  - `byKind` counts
  - optional: paths to per-kind indexes for fast loading
- `externalReferences`:
  - optional list of known external libraries/packages used for linking

### 3.2 Common fields for all entities
- `schemaVersion`
- `kind`
- `uid`
- `name`
- `fullName` (when applicable)
- `displayName`
- `description` (optional plain text; separate from structured doc comments)
- `doc`:
  - `summary`
  - `remarks`
  - `examples[]`
  - `params[]`
  - `returns`
  - `exceptions[]`
  - `seeAlso[]`
- `tags[]` (for grouping; optional)
- `refs[]` (list of `Ref`)
- `source` (file + repo mapping)
- `metadata` (free-form map, reserved for extension)

### 3.3 `Ref`
- `uid`
- `kind` (optional)
- `name`
- `fullName` (optional)
- `external` (boolean)
- `href` (optional; used if external linking is pre-resolved)

### 3.4 `TypeRef`
A TypeRef MUST represent:
- `name` (display)
- `uid` (if resolvable to an entity)
- `namespace` (optional)
- `genericArgs[]` (TypeRef)
- `arrayRank` (optional)
- `nullable` (optional)
- `byRef` (optional)
- `pointer` (optional)
- `tupleElements[]` (optional)
- `constraints` (optional, mostly for generic params)

## 4. Entity specifics (required fields)
### 4.1 Namespace entity
- `kind: namespace`
- `assemblyUids[]`
- `typeUids[]` (or a separate index for scalability)

### 4.2 Type entity
- `kind: type`
- `typeKind` (class/struct/interface/enum/delegate/record)
- `namespaceUid`
- `assemblyUid`
- `accessibility`
- `modifiers` (static/abstract/sealed/partial)
- `genericParams[]`
- `baseType` (TypeRef)
- `interfaces[]` (TypeRef)
- `members[]` (list of member refs or ids)
- `nestedTypes[]` (optional)

### 4.3 Member entity
- `kind: member`
- `memberKind` (method/ctor/property/field/event/operator/indexer)
- `declaringTypeUid`
- `accessibility`
- `modifiers` (static/virtual/override/abstract/async/etc.)
- `signature`:
  - `csharp`
  - optional `metadata`
- `parameters[]`:
  - `name`
  - `type` (TypeRef)
  - `defaultValue` (optional; redacted/normalized)
  - `isOptional`
  - `isParams`
- `returnType` (TypeRef; for methods and operators)
- `type` (TypeRef; for properties/fields/events)
- `overloadGroupUid` (optional)
- `attributes[]` (optional, but required for Azure Functions extraction provenance)

## 5. Azure Functions entities
### 5.1 Function App entity
- `kind: azureFunctionApp`
- `projectUid`
- `name`
- `functions[]` (refs)

### 5.2 Azure Function entity
- `kind: azureFunction`
- `functionName` (deployed name)
- `functionAppUid`
- `entryPointUid` (member uid)
- `trigger` (Binding)
- `bindings[]` (Binding list including trigger + others)
- `http` (optional normalized section for httpTrigger convenience)
- `durable` (optional normalized section if durable role detected)

### 5.3 Binding schema
- `bindingType` (string discriminator, e.g. `httpTrigger`)
- `direction` (`in`|`out`|`inout`)
- `name` (binding name)
- `parameterName` (optional; method parameter it binds to)
- `parameterType` (optional TypeRef)
- `properties` (map; validated per bindingType)
- `provenance`:
  - `source` (attribute|config|convention)
  - `attributeType` (optional)
  - `raw` (optional raw kv, sanitized)

## 6. Example YAML snippets (illustrative)
These examples are informative only; final shapes are governed by the schema version.

### 6.1 Type entity (illustrative)

```yaml
schemaVersion: 1.0.0 
kind: type 
uid: T:MyCompany.Core.Widget 
name: Widget 
fullName: MyCompany.Core.Widget 
displayName: Widget 
typeKind: class 
namespaceUid: N:MyCompany.Core 
assemblyUid: A:MyCompany.Core@net8.0 
accessibility: public 
modifiers: [sealed] 
doc: 
summary: "Represents a widget." 
refs: [] 
source: 
filePath: src/MyCompany.Core/Widget.cs 
startLine: 10 
endLine: 120 
repo: 
url: [https://example.invalid/repo](https://example.invalid/repo) 
revision: "" 
path: src/MyCompany.Core/Widget.cs
``` 

### 6.2 Azure Function entity (illustrative)

```yaml
schemaVersion: 1.0.0 
kind: azureFunction 
uid: F:P:MyFuncApp/HealthCheck 
functionName: HealthCheck 
functionAppUid: FA:P:MyFuncApp 
entryPointUid: M:T:MyFuncApp.Functions.Health.HealthCheck.Run(Microsoft.AspNetCore.Http.HttpRequest) 
trigger: 
bindingType: httpTrigger 
direction: in 
name: req 
parameterName: req 
properties: 
  methods: [GET] 
  route: health 
  authLevel: anonymous 
  bindings:
    - bindingType: httpTrigger 
      direction: in 
      name: req 
      parameterName: req 
      properties: 
        methods: [GET] 
        route: health 
        authLevel: anonymous
    - bindingType: http 
      direction: out 
      name: res 
      properties: {} 
  doc: 
  summary: "Health endpoint." 
  refs: []
```