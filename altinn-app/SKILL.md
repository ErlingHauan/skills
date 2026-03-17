---
name: altinn-app
description: Guide for editing Altinn apps — layouts, data models, policies, texts, prefill, expressions, and process config.
---

# Altinn App Development Guide

## App Structure

An Altinn app follows this directory layout:

```
App/
  Program.cs              # Entry point, service registrations
  models/                 # One or more data models (each a set of 3 files)
    [Name].cs             # C# data model
    [Name].schema.json    # JSON Schema definition
    [Name].xsd            # XML Schema definition
    [Name].prefill.json   # Prefill configuration (optional)
  ui/
    form/
      layouts/            # Page layout JSON files (one per page)
        page1.json
        page2.json
      Settings.json       # Page ordering and form-level settings
  config/
    texts/
      resource.nb.json    # Norwegian text resources
      resource.en.json    # English text resources (optional)
    process/
      process.bpmn        # Process flow definition
    applicationmetadata.json
  logic/                  # Custom business logic (validation, instantiation)
  config/
    authorization/
      policy.xml          # XACML authorization policy
  options/                # Dropdown/select option lists (e.g., JaNei.json)
```

## Workflow: Which Reference to Read

Choose the reference file based on what you need to do:

| Task | Read This Reference |
|------|-------------------|
| Add/modify fields in the data model | `references/datamodel.md` |
| Create/edit form pages or components | `references/layout.md` |
| Add/edit text labels or translations | `references/texts.md` |
| Configure authorization and roles | `references/policy.md` |
| Set up automatic data prefilling | `references/prefill.md` |
| Add conditional logic (show/hide, validation) | `references/dynamic-expressions.md` |
| Configure process flow (tasks, signing, payment, gateways) | `references/process.md` |

**Before making changes, always explore the app directory first** to understand the existing structure, file names, and patterns. Read existing files before modifying them.

## Key Patterns

### Service Registration
Custom services are registered in `RegisterCustomAppServices()` in Program.cs:
```csharp
services.AddTransient<IInstantiationProcessor, InstantiationHandler>();
services.AddTransient<IInstanceValidator, ValidationHandler>();
```

### Namespace Conventions
Always use `Altinn.App.*` namespaces (e.g., `Altinn.App.Models.ModelName`).

### Data Bindings
Layout components bind to data model properties via `dataModelBindings.simpleBinding` using the property name from the model.

### Text Resources
UI text is stored in `config/texts/resource.nb.json` and referenced by keys in layouts via `textResourceBindings.title`.

### Options
Dropdown/select options go in separate JSON files under `options/` (e.g., `JaNei.json` for yes/no).

## Critical Rules

1. **Always update all 3 data model files** when making model changes: `.cs`, `.schema.json`, `.xsd` — all sharing the same base name. See `references/datamodel.md`.

2. **Always update Settings.json** when adding new pages. Add the page ID to `pages.order`. See `references/layout.md`.

3. **Always keep all resource files in sync** across languages. Every text resource ID must exist in all language files. See `references/resources.md`.

4. **Follow the dependency order**: data model -> layouts -> text resources. Create the data model fields first, then layouts that bind to them, then text resources referenced by layouts.

5. **Don't create new schema URLs or metadata properties** - match existing patterns in the app's files.

6. **Don't ask for React components or frontend code** - Altinn apps use declarative JSON layouts, not React.

7. **Remember to update service registration in Program.cs** when adding new custom services.

## Common Mistakes to Avoid

- Creating files without reading existing ones first
- Assuming file locations without exploring the directory
- Creating new data model files instead of editing existing ones
- Forgetting to update Settings.json when adding a new page
- Using general web knowledge instead of Altinn-specific patterns
- Missing the `[Required]` attribute / `minOccurs` / `required` array sync across data model files
