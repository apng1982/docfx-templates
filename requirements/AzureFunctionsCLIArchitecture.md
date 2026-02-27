# Requirements: Azure Functions extraction (detailed)

## 1. Detection
The CLI MUST support detecting Azure Functions projects/apps via one or more strategies:
- SDK/package references (typical Azure Functions packages)
- project properties (where applicable)
- presence of conventional files (`host.json`, local settings) if provided (optional)
- attributes on methods/classes (the most reliable for in-code extraction)

Detection MUST be configurable:
- `--functions on|off|auto`

## 2. Function entry point discovery
The CLI MUST discover functions by scanning compiled symbols for:
- a function naming attribute (e.g., `[FunctionName("X")]`) OR other supported conventions depending on the Azure Functions model targeted.
- corresponding trigger attributes on parameters (HTTP trigger, timer trigger, etc.)

Each discovered function MUST map to:
- `AzureFunction` entity with `entryPointUid` referencing the member entity.

## 3. Binding extraction
For each function:
- Exactly one trigger binding MUST be identified; otherwise emit a diagnostic:
  - no trigger => warning or error (configurable)
  - multiple triggers => error
- All other bindings MUST be extracted where possible:
  - input bindings
  - output bindings
  - return-value-based output (if detectable)

The binding model MUST:
- preserve raw binding properties
- provide normalized known fields for common binding types

## 4. Known binding types (initial minimum set)
The CLI MUST define schemas for at least:
- `httpTrigger`
- `timerTrigger`
- `queueTrigger`
- `blobTrigger`
- `serviceBusTrigger`
- `cosmosDBTrigger`

The CLI SHOULD add:
- `eventGridTrigger`
- `eventHubTrigger`
- `signalR`
- `kafkaTrigger` (if applicable in your ecosystem)
- durable orchestration/activity/entity roles (if used)

## 5. Normalized HTTP trigger section (rendering-friendly)
For `httpTrigger`, the CLI MUST provide normalized fields (even if derived from the properties bag):
- `methods[]`
- `route`
- `authLevel`
- `requestType` (optional TypeRef, if discoverable)
- `responseType` (optional TypeRef, if discoverable)

## 6. Provenance rules
For any extracted binding property, the CLI MUST record provenance at least at the binding level:
- attribute-based (most common)
- config-based (if host.json / function.json is parsed)
- convention-based inference (should be minimized and clearly flagged)

## 7. Safety and sanitization
The CLI MUST NOT emit secrets.
If any configuration sources contain sensitive values (e.g., connection strings), the CLI MUST:
- omit them, or
- replace with placeholders like `<CONNECTION_SETTING_NAME>` rather than values

Diagnostics SHOULD warn when redactions occur.

## 8. Rendering intent (what the plugin will need)
The emitted function model MUST allow the Docusaurus plugin to:
- group functions by trigger type
- show a "Bindings" table with binding type, direction, name, parameter, and key properties
- show an "HTTP" section with route/methods/auth
- deep-link from a function page to the entry point method/type docs
