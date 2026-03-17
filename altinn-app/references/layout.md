# Altinn Form Layout Reference

## Overview

Altinn apps use declarative JSON to define form pages. Each page is a separate JSON file in `App/ui/form/layouts/`. Pages are ordered via `App/ui/form/Settings.json`.

## Page Layout Structure

Each layout file contains an array of components:

```json
{
  "$schema": "https://altinncdn.no/toolkits/altinn-app-frontend/4/schemas/json/layout/layout.schema.v1.json",
  "data": {
    "layout": [
      {
        "id": "unique-component-id",
        "type": "Input",
        "dataModelBindings": {
          "simpleBinding": "propertyName"
        },
        "textResourceBindings": {
          "title": "component-title-resource-key"
        },
        "required": true
      }
    ]
  }
}
```

## Settings.json

Located at `App/ui/form/Settings.json`. **You must update this file when adding or removing pages.**

```json
{
  "$schema": "https://altinncdn.no/toolkits/altinn-app-frontend/4/schemas/json/layout/layoutSettings.schema.v1.json",
  "pages": {
    "order": ["page1", "page2", "page3"],
    "showLanguageSelector": false
  }
}
```

- `order`: Array of page IDs (filenames without `.json`). Controls navigation order.
- `showLanguageSelector`: Set to `true` when the app supports multiple languages.

## Common Component Types

### Input (text field)
```json
{
  "id": "input-firstName",
  "type": "Input",
  "dataModelBindings": {
    "simpleBinding": "firstName"
  },
  "textResourceBindings": {
    "title": "input-firstName.title",
    "description": "input-firstName.description"
  },
  "required": true
}
```

### Datepicker
```json
{
  "id": "datepicker-birthDate",
  "type": "Datepicker",
  "dataModelBindings": {
    "simpleBinding": "birthDate"
  },
  "textResourceBindings": {
    "title": "datepicker-birthDate.title"
  },
  "minDate": "1900-01-01",
  "maxDate": "2024-12-31"
}
```

### Dropdown
```json
{
  "id": "dropdown-status",
  "type": "Dropdown",
  "dataModelBindings": {
    "simpleBinding": "status"
  },
  "textResourceBindings": {
    "title": "dropdown-status.title"
  },
  "optionsId": "statusOptions"
}
```
The `optionsId` references a file `App/options/statusOptions.json`.

### RadioButtons
```json
{
  "id": "radio-yesNo",
  "type": "RadioButtons",
  "dataModelBindings": {
    "simpleBinding": "answer"
  },
  "textResourceBindings": {
    "title": "radio-yesNo.title"
  },
  "optionsId": "JaNei"
}
```

### Checkboxes
```json
{
  "id": "checkboxes-interests",
  "type": "Checkboxes",
  "dataModelBindings": {
    "simpleBinding": "interests"
  },
  "textResourceBindings": {
    "title": "checkboxes-interests.title"
  },
  "optionsId": "interestOptions"
}
```

### TextArea
```json
{
  "id": "textarea-comments",
  "type": "TextArea",
  "dataModelBindings": {
    "simpleBinding": "comments"
  },
  "textResourceBindings": {
    "title": "textarea-comments.title"
  }
}
```

### Header
```json
{
  "id": "header-main",
  "type": "Header",
  "size": "L",
  "textResourceBindings": {
    "title": "header-main.title"
  }
}
```
Sizes: `"L"` (h2), `"M"` (h3), `"S"` (h4).

### Paragraph
```json
{
  "id": "paragraph-info",
  "type": "Paragraph",
  "textResourceBindings": {
    "title": "paragraph-info.title"
  }
}
```

### Panel
```json
{
  "id": "panel-info",
  "type": "Panel",
  "variant": "info",
  "textResourceBindings": {
    "title": "panel-info.title",
    "body": "panel-info.body"
  }
}
```
Variants: `"info"`, `"warning"`, `"success"`.

### FileUpload
```json
{
  "id": "fileupload-attachment",
  "type": "FileUpload",
  "dataModelBindings": {
    "simpleBinding": "attachmentId"
  },
  "textResourceBindings": {
    "title": "fileupload-attachment.title"
  },
  "maxFileSizeInMB": 25,
  "maxNumberOfAttachments": 3,
  "minNumberOfAttachments": 0,
  "displayMode": "simple"
}
```

### RepeatingGroup
```json
{
  "id": "group-persons",
  "type": "RepeatingGroup",
  "dataModelBindings": {
    "group": "persons"
  },
  "textResourceBindings": {
    "title": "group-persons.title"
  },
  "maxCount": 10,
  "children": ["input-personName", "input-personAge"]
}
```

## Data Bindings

Components bind to the data model via `dataModelBindings`:
- `simpleBinding`: For simple properties (string, number, boolean, date)
- `group`: For repeating groups (arrays in the data model)
- Use dot notation for nested properties: `"address.streetName"`

## Grid System

Components support a Bootstrap-style responsive grid:

```json
{
  "id": "input-example",
  "type": "Input",
  "grid": {
    "xs": 12,
    "sm": 6,
    "md": 4
  }
}
```

For label/input proportions:
```json
{
  "id": "input-example",
  "type": "Input",
  "labelGrid": { "xs": 12, "sm": 4 },
  "innerGrid": { "xs": 12, "sm": 8 }
}
```

## Text Resource Bindings

All user-visible text should reference keys in resource files:
- `title`: Main label
- `description`: Help text below the label
- `help`: Expandable help text (shown via help icon)
- `body`: Body content (for Panel, Paragraph)

See `references/resources.md` for how to define these.

## Adding a New Page - Checklist

1. Create a new JSON file in `App/ui/form/layouts/` (e.g., `newPage.json`)
2. Add the page ID to `Settings.json` `pages.order` array
3. Add all text resource keys used by components to `config/texts/resource.nb.json` (and other language files)
4. Ensure all `dataModelBindings` reference existing data model properties
5. Reference existing layout files in the app to match the established patterns
