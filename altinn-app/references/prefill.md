# Altinn Prefill Configuration Reference

## Overview

Prefill automatically populates form fields with data from external sources (registers, user profile, URL parameters) when a form instance is created.

## File Location

The prefill config file goes in `App/models/` with this naming pattern:
```
[dataModelName].prefill.json
```
For example: `model.prefill.json` alongside `model.cs`, `model.schema.json`, `model.xsd`.

**Do not create new data model files** - only create the prefill configuration file.

## Configuration Structure

```json
{
  "$schema": "https://altinncdn.no/schemas/json/prefill/prefill.schema.v1.json",
  "allowOverwrite": true,
  "ER": { },
  "DSF": { },
  "UserProfile": { },
  "QueryParameters": { }
}
```

### Properties
- `$schema`: Points to the prefill JSON schema (provides editor validation/autocomplete)
- `allowOverwrite`: Whether prefilled data can overwrite existing values
- `ER`: Mappings from the Central Coordinating Register (organization data)
- `DSF`: Mappings from the National Population Register (person data)
- `UserProfile`: Mappings from the logged-in user's profile
- `QueryParameters`: Mappings from URL query parameters

### Mapping Syntax

```json
"SourceField": "TargetModelPath"
```
- `SourceField`: Field name from the external data source
- `TargetModelPath`: Dot-notation path to the property in your data model

## Data Sources

### UserProfile (The Submitter)

Always available. Contains info about the **logged-in user** filling the form, regardless of who they represent.

| Field | Description |
|-------|-------------|
| `UserId` | User ID |
| `UserName` | Username |
| `PhoneNumber` | Phone number |
| `Email` | Email address |
| `PartyId` | Party ID |
| `Party.Name` | Full name of the party |
| `Party.SSN` | Personal identification number |
| `Party.OrgNumber` | Organization number |
| `Party.Person` | Person object (same fields as DSF) |
| `Party.Organization` | Organization object (same fields as ER) |
| `ProfileSettingPreference.Language` | Language preference |

**Universal access via Party**: `UserProfile.Party.Person` has the same fields as DSF, and `UserProfile.Party.Organization` has the same fields as ER.

### DSF (National Population Register)

Only available when submitting **on behalf of a person** (not an organization). Contains data about the **represented person**, not the submitter.

| Field | Description |
|-------|-------------|
| `SSN` | Personal identification number |
| `Name` | Full name |
| `FirstName` | First name |
| `MiddleName` | Middle name |
| `LastName` | Last name |
| `TelephoneNumber` | Telephone number |
| `MobileNumber` | Mobile phone number |
| `MailingAddress` | Mailing address |
| `MailingPostalCode` | Mailing postal code |
| `MailingPostalCity` | Mailing city/town |
| `AddressMunicipalNumber` | Municipal number |
| `AddressMunicipalName` | Municipal name |
| `AddressStreetName` | Street name |
| `AddressHouseNumber` | House number |
| `AddressHouseLetter` | House letter |
| `AddressPostalCode` | Postal code |
| `AddressCity` | City/town |

### ER (Central Coordinating Register)

Only available when submitting **on behalf of an organization**. Contains data about the **represented organization**.

| Field | Description |
|-------|-------------|
| `OrgNumber` | Organization number |
| `Name` | Organization name |
| `UnitType` | Organization type |
| `TelephoneNumber` | Telephone number |
| `MobileNumber` | Mobile phone number |
| `FaxNumber` | Fax number |
| `EMailAddress` | Email address |
| `InternetAddress` | Internet address |
| `MailingAddress` | Mailing address |
| `MailingPostalCode` | Mailing postal code |
| `MailingPostalCity` | Mailing city/town |
| `BusinessAddress` | Business address |
| `BusinessPostalCode` | Business postal code |
| `BusinessPostalCity` | Business city/town |

### QueryParameters

Maps URL query parameters to data model fields. Requires custom setup.

## Examples

### Recommended Approach (UserProfile.Party)

Works regardless of whether the user represents themselves, another person, or an organization:

```json
{
  "$schema": "https://altinncdn.no/schemas/json/prefill/prefill.schema.v1.json",
  "allowOverwrite": true,
  "UserProfile": {
    "Email": "submitter.email",
    "PhoneNumber": "submitter.phoneNumber",
    "Party.Organization.OrgNumber": "organization.orgNo",
    "Party.Organization.Name": "organization.name",
    "Party.Person.SSN": "person.personNr",
    "Party.Person.FirstName": "person.firstName",
    "Party.Person.LastName": "person.lastName"
  }
}
```

### Traditional Approach (Separate Data Sources)

```json
{
  "$schema": "https://altinncdn.no/schemas/json/prefill/prefill.schema.v1.json",
  "allowOverwrite": true,
  "ER": {
    "OrgNumber": "organization.orgNo",
    "Name": "organization.name"
  },
  "DSF": {
    "SSN": "person.personNr",
    "FirstName": "person.firstName",
    "LastName": "person.lastName"
  },
  "UserProfile": {
    "Email": "submitter.email",
    "PhoneNumber": "submitter.phoneNumber"
  }
}
```

### Nested Properties

```json
{
  "$schema": "https://altinncdn.no/schemas/json/prefill/prefill.schema.v1.json",
  "allowOverwrite": true,
  "ER": {
    "OrgNumber": "company.details.organizationNumber",
    "Name": "company.details.officialName",
    "BusinessAddress": "company.address.street"
  }
}
```

## Common Pitfalls

1. **No "User" data source exists** - use `UserProfile` instead
2. **DSF is not the logged-in user** - DSF is the *represented* person; use `UserProfile` for the submitter
3. **ER requires organization context** - instantiation fails if you prefill ER data without an organization
4. **DSF requires person context** - instantiation fails if you prefill DSF data when submitting on behalf of an organization
5. **Most common scenario**: Form submitted on behalf of an organization, with a person as submitter. Person data fields (name, phone, email) usually refer to the submitter (`UserProfile`), not DSF.

## Best Practices

1. Only include fields you actually need
2. Use exact field names from the data source tables above
3. Ensure target paths match your data model structure
4. Use `UserProfile.Party` for universal access across contexts
5. Set `allowOverwrite` to `false` if you want to preserve user-entered data
