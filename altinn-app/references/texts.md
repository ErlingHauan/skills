# Altinn Text Resources Reference

## Critical Requirement

When working with text resources, **all resource files must be updated**. Every text resource ID must exist in all language files.

## File Location

Text resources are stored in `App/config/texts/`:
- `resource.nb.json` - Norwegian Bokmal (primary)
- `resource.nn.json` - Norwegian Nynorsk (optional)
- `resource.en.json` - English (optional)

## File Structure

```json
{
  "language": "nb",
  "resources": [
    {
      "id": "input-firstName.title",
      "value": "Fornavn"
    },
    {
      "id": "input-firstName.description",
      "value": "Skriv inn ditt fornavn"
    }
  ]
}
```

- `language`: ISO 639-1 two-letter code (`nb`, `nn`, `en`, etc.)
- `resources`: Array of `{id, value}` objects
- `id`: Must match the key referenced in layout component `textResourceBindings`

## Variable Interpolation

You can embed data model values in text resources using `{0}`, `{1}`, etc.:

```json
{
  "id": "summary-paragraph.title",
  "value": "Ditt telefonnummer er: {0}\n\nDin epostadresse er: {1}",
  "variables": [
    {
      "key": "kontaktinformasjon.Telefonnummer",
      "dataSource": "dataModel.model",
      "defaultValue": " "
    },
    {
      "key": "kontaktinformasjon.Epost",
      "dataSource": "dataModel.model",
      "defaultValue": " "
    }
  ]
}
```

- `key`: Property path in the data model
- `dataSource`: Always `"dataModel.model"` for data model references
- `defaultValue`: Fallback if the value is empty

## Editing Guidelines

### Adding a Resource
1. Add to **all** resource files with the same `id`
2. The `id` must match the key used in layout component `textResourceBindings`

### Removing a Resource
1. Find the `id` from the layout component
2. Remove from **all** resource files

### Modifying a Resource
1. Find the `id` from the layout component
2. Update in **all** resource files

### Adding a Language
1. Create a new file: `resource.[lang].json` (use ISO 639-1 code)
2. Copy all resource IDs from existing files and translate values
3. Add `"showLanguageSelector": true` to `ui/form/Settings.json` inside the `pages` property
4. In `App/config/applicationmetadata.json`, add the app title in the new language

## Naming Conventions for Resource IDs

Follow the pattern used in existing layout files. Common patterns:
- `componentId.title` - Main label
- `componentId.description` - Description/help text
- `componentId.body` - Body content (panels, paragraphs)
- `header-pageName.title` - Page headers

## Validation Checklist

- [ ] ID identical across all resource files
- [ ] ID matches component reference in layout files
- [ ] All resources exist in all resource files
- [ ] Variable indices (`{0}`, `{1}`) match the `variables` array order
- [ ] `dataSource` is `"dataModel.model"` for data model references
