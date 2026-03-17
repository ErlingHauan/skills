# Altinn Data Model Reference

## Critical Requirement

When working with Altinn data models, you **MUST** maintain **ALL THREE files** for each data model as a synchronized set:
- `[ModelName].cs` - C# data model
- `[ModelName].schema.json` - JSON Schema
- `[ModelName].xsd` - XML Schema

An Altinn app can have **multiple data models**. Each data model consists of its own set of three files, all sharing the same base name. Explore `App/models/` to discover which data models exist before making changes.

## File Structures

### 1. C# Model File (model.cs)

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.Text.Json.Serialization;
using System.Xml.Serialization;
using Newtonsoft.Json;

namespace Altinn.App.Models.ModelName
{
  [XmlRoot(ElementName="ModelName")]
  public class ModelName
  {
    [XmlElement("propertyName", Order = 1)]
    [Required]
    [JsonProperty("propertyName")]
    [JsonPropertyName("propertyName")]
    public string propertyName { get; set; }

    [XmlElement("optionalProp", Order = 2)]
    [JsonProperty("optionalProp")]
    [JsonPropertyName("optionalProp")]
    public string optionalProp { get; set; }
  }
}
```

Key points:
- Namespace: `Altinn.App.Models.[ModelName]`
- `[XmlRoot]` on the class
- Every property needs `[XmlElement]` (with Order), `[JsonProperty]`, and `[JsonPropertyName]`
- Required properties get `[Required]`

### 2. JSON Schema File (model.schema.json)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "http://altinn-repositories:3000/org/repo/App/models/ModelName.schema.json",
  "info": {
    "rootNode": ""
  },
  "@xsdNamespaces": {
    "xsd": "http://www.w3.org/2001/XMLSchema",
    "xsi": "http://www.w3.org/2001/XMLSchema-instance",
    "seres": "http://seres.no/xsd/forvaltningsdata"
  },
  "@xsdSchemaAttributes": {
    "AttributeFormDefault": "Unqualified",
    "ElementFormDefault": "Qualified",
    "BlockDefault": "None",
    "FinalDefault": "None"
  },
  "@xsdRootElement": "ModelName",
  "type": "object",
  "required": ["propertyName"],
  "properties": {
    "propertyName": {
      "type": "string"
    },
    "optionalProp": {
      "type": "string",
      "@xsdMinOccurs": 0
    },
    "complexProperty": {
      "$ref": "#/$defs/ComplexType"
    }
  },
  "$defs": {
    "ComplexType": {
      "properties": {
        "subProperty": {
          "@xsdType": "string",
          "type": "string",
          "@xsdMinOccurs": 0
        }
      }
    }
  }
}
```

Key points:
- `@xsdRootElement` must match the model name
- Required properties go in the `"required"` array
- Optional properties get `"@xsdMinOccurs": 0`
- Complex types go in `$defs` and are referenced via `$ref`

### 3. XSD File (model.xsd)

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsd:schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:seres="http://seres.no/xsd/forvaltningsdata"
            xmlns:xs="http://www.w3.org/2001/XMLSchema"
            attributeFormDefault="unqualified"
            elementFormDefault="qualified"
            xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <xs:annotation>
    <xs:documentation>
      <xsd:attribute name="rootNode" fixed="" />
    </xs:documentation>
  </xs:annotation>
  <xs:element name="ModelName">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="propertyName" type="xs:string" />
        <xs:element minOccurs="0" name="optionalProp" type="xs:string" />
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xsd:schema>
```

Key points:
- Required properties: omit `minOccurs`
- Optional properties: `minOccurs="0"`
- Property order in `xs:sequence` must match C# `Order` attributes

## Type Mappings

| Base Type | C# Type  | JSON Schema Type | XSD Type    |
|-----------|----------|------------------|-------------|
| string    | string   | string           | xs:string   |
| integer   | int      | integer          | xs:integer  |
| number    | decimal  | number           | xs:decimal  |
| boolean   | bool     | boolean          | xs:boolean  |
| date      | DateTime | string           | xs:date     |
| datetime  | DateTime | string           | xs:dateTime |
| array     | List<T>  | array            | complex     |
| object    | Class    | object/$ref      | complex     |

## Naming Conventions

- **Model names**: PascalCase (e.g., `TaxReport`)
- **Property names**: camelCase (e.g., `firstName`)
- **Type names in $defs**: PascalCase (e.g., `PersonDetails`)
- Names must be **consistent across all three files**

## Common Patterns

### Enumeration Types
```json
"$defs": {
  "StatusType": {
    "@xsdType": "string",
    "@xsdStructure": "XmlSchemaSimpleTypeRestriction",
    "type": "string",
    "enum": ["Active", "Inactive", "Pending"]
  }
}
```

### Restricted String Types
```json
"$defs": {
  "PersonName": {
    "@xsdType": "string",
    "@xsdStructure": "XmlSchemaSimpleTypeRestriction",
    "type": "string",
    "maxLength": 175,
    "minLength": 1
  }
}
```

### Array Properties
- C#: `List<T>` type
- JSON Schema: `"type": "array"` with `"items"` containing `$ref`, plus `"@xsdMaxOccurs": "unbounded"`
- XSD: `maxOccurs="unbounded"`

### Root Model with oneOf
For complex models, use `oneOf` at root level:
```json
"@xsdRootElement": "model",
"type": "object",
"oneOf": [{ "$ref": "#/$defs/MainDataModel" }],
"$defs": {
  "MainDataModel": {
    "properties": { ... }
  }
}
```

## Required vs Optional Properties

| File | Required | Optional |
|------|----------|----------|
| C# | Add `[Required]` attribute | No `[Required]` attribute |
| JSON Schema | Include in `"required"` array | Add `"@xsdMinOccurs": 0` |
| XSD | Omit `minOccurs` | Add `minOccurs="0"` |

## Editing Guidelines

### Adding a Property
1. Add to ALL THREE files with consistent naming and types
2. Use `$ref` in JSON Schema if referencing a defined type
3. Assign a unique Order number in C#
4. Set required/optional status consistently across files

### Removing a Property
1. Remove from ALL THREE files
2. Update `"required"` array in JSON Schema
3. Remove unused type definitions from `$defs`

### Adding a Complex Type
1. Add `$defs` entry in JSON Schema
2. Create corresponding C# class
3. Add corresponding XSD complex type
4. Use consistent naming across files

## Validation Checklist

Before finalizing any data model change, verify:

- [ ] Property names identical across all three files
- [ ] Required properties match in all files
- [ ] Data types correspond according to type mapping table
- [ ] Property order in C# matches XSD sequence
- [ ] Optional properties have `@xsdMinOccurs: 0` in JSON Schema
- [ ] Arrays have `@xsdMaxOccurs: "unbounded"` in JSON Schema
- [ ] All `$ref` references point to valid `$defs` entries
- [ ] `@xsdRootElement` matches the model name
- [ ] Enums have `@xsdStructure: "XmlSchemaSimpleTypeRestriction"`
