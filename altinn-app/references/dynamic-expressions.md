# Altinn Dynamic Expressions Reference

## Overview

Dynamic expressions add conditional logic to form components in existing layout files. They control visibility, required status, read-only state, and validation.

## Purpose

- Add conditional show/hide logic to components
- Make fields conditionally required
- Set fields as read-only based on conditions
- Implement dynamic validation rules

**Do not create new layout files** for expressions - only modify existing ones.

## Expression Syntax

Expressions are arrays where the first element is the function name and subsequent elements are arguments:

```json
["equals", ["dataModel", "fieldName"], "expectedValue"]
```

### Supported Functions

| Function | Arguments | Returns | Description |
|----------|-----------|---------|-------------|
| `equals` | value1, value2 | boolean | True if values are equal |
| `notEquals` | value1, value2 | boolean | True if values are not equal |
| `greaterThan` | value1, value2 | boolean | True if value1 > value2 |
| `greaterThanEq` | value1, value2 | boolean | True if value1 >= value2 |
| `lessThan` | value1, value2 | boolean | True if value1 < value2 |
| `lessThanEq` | value1, value2 | boolean | True if value1 <= value2 |
| `not` | boolean | boolean | Negates the value |
| `and` | bool1, bool2, ... | boolean | True if all are true |
| `or` | bool1, bool2, ... | boolean | True if any is true |
| `if` | condition, then, else | any | Conditional value |
| `concat` | str1, str2, ... | string | Concatenates strings |
| `contains` | string, substring | boolean | True if string contains substring |
| `endsWith` | string, suffix | boolean | True if string ends with suffix |
| `startsWith` | string, prefix | boolean | True if string starts with prefix |

### Data Sources in Expressions

```json
["dataModel", "path.to.field"]      // Reference a data model field
["component", "componentId"]        // Reference another component's value
["instanceContext", "instanceId"]    // Instance context values
["frontendSettings", "settingKey"]  // Frontend settings values
null                                 // Null/empty value
```

## Common Usage Patterns

### Conditional Visibility (hidden)

Show a component only when a condition is met:

```json
{
  "id": "input-email",
  "type": "Input",
  "hidden": ["equals", ["dataModel", "contactMethod"], "email"],
  "dataModelBindings": {
    "simpleBinding": "emailAddress"
  }
}
```

**Note**: `hidden` means the component is hidden when the expression is `true`. To show a component conditionally, negate the logic or use `not`.

### Conditionally Required

```json
{
  "id": "input-orgNumber",
  "type": "Input",
  "required": ["equals", ["dataModel", "entityType"], "organization"],
  "dataModelBindings": {
    "simpleBinding": "orgNumber"
  }
}
```

### Read-Only Based on Condition

```json
{
  "id": "input-name",
  "type": "Input",
  "readOnly": ["notEquals", ["dataModel", "status"], "draft"],
  "dataModelBindings": {
    "simpleBinding": "name"
  }
}
```

### Nested Expressions

```json
{
  "hidden": ["and",
    ["equals", ["dataModel", "type"], "person"],
    ["greaterThanEq", ["dataModel", "age"], 18]
  ]
}
```

### If/Else Expression

```json
{
  "required": ["if",
    ["equals", ["dataModel", "country"], "NO"],
    true,
    false
  ]
}
```

## Supported Component Properties

These properties accept dynamic expressions:
- `hidden` - Show/hide the component
- `required` - Make the field required
- `readOnly` - Make the field read-only
- `editIndex` - Control editing in repeating groups

**Do not use expressions in properties that don't support them** (e.g., `dataModelBindings`, `onClickAction`).

## Common Pitfalls

1. **Type mismatch**: Ensure the operation is valid for the data type (don't compare strings with `greaterThan`)
2. **Missing quotes**: String literals must be in quotes
3. **Invalid data paths**: Verify the data model binding path exists in the model
4. **Case sensitivity**: Function names and data paths are case-sensitive
5. **Nested return types**: Each function's return type must match the expected input of the parent function
6. **hidden vs visible**: `hidden: true` means the component is NOT shown. Invert logic accordingly.
