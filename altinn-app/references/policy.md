# Altinn Authorization Policy Reference

## Overview

Authorization in Altinn apps is configured via `App/config/authorization/policy.xml` using XACML 3.0. Each rule defines **Who** (roles) can do **What** (actions) on **Which** resources.

## Critical Rules

- Only add access controls that are explicitly specified - never add extra permissions
- Add one role per rule for clarity
- The policy.xml file should not contain XML comments

## Policy Structure

```xml
<xacml:Policy xmlns:xacml="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17"
              PolicyId="urn:altinn:example:policyid:1"
              Version="1.0"
              RuleCombiningAlgId="urn:oasis:names:tc:xacml:3.0:rule-combining-algorithm:permit-overrides">

  <!-- All rules come first -->
  <xacml:Rule RuleId="urn:altinn:example:ruleid:1" Effect="Permit">
    ...
  </xacml:Rule>

  <xacml:Rule RuleId="urn:altinn:example:ruleid:2" Effect="Permit">
    ...
  </xacml:Rule>

  <!-- ObligationExpressions come after all rules -->
  <xacml:ObligationExpressions>
    ...
  </xacml:ObligationExpressions>
</xacml:Policy>
```

## Rule Structure

Each rule has three sections: Who, What, How.

```xml
<xacml:Rule RuleId="urn:altinn:example:ruleid:1" Effect="Permit">
  <xacml:Description>Human-readable description</xacml:Description>
  <xacml:Target>

    <!-- Section 1: WHO (Roles) -->
    <xacml:AnyOf>
      <xacml:AllOf>
        <xacml:Match MatchId="urn:oasis:names:tc:xacml:3.0:function:string-equal-ignore-case">
          <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">DAGL</xacml:AttributeValue>
          <xacml:AttributeDesignator AttributeId="urn:altinn:rolecode" Category="urn:oasis:names:tc:xacml:1.0:subject-category:access-subject" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false" />
        </xacml:Match>
      </xacml:AllOf>
    </xacml:AnyOf>

    <!-- Section 2: WHAT (Resources) -->
    <xacml:AnyOf>
      <xacml:AllOf>
        <xacml:Match MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
          <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">[ORG]</xacml:AttributeValue>
          <xacml:AttributeDesignator AttributeId="urn:altinn:org" Category="urn:oasis:names:tc:xacml:3.0:attribute-category:resource" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false" />
        </xacml:Match>
        <xacml:Match MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
          <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">[APP]</xacml:AttributeValue>
          <xacml:AttributeDesignator AttributeId="urn:altinn:app" Category="urn:oasis:names:tc:xacml:3.0:attribute-category:resource" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false" />
        </xacml:Match>
      </xacml:AllOf>
    </xacml:AnyOf>

    <!-- Section 3: HOW (Actions) -->
    <xacml:AnyOf>
      <xacml:AllOf>
        <xacml:Match MatchId="urn:oasis:names:tc:xacml:3.0:function:string-equal-ignore-case">
          <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">read</xacml:AttributeValue>
          <xacml:AttributeDesignator AttributeId="urn:oasis:names:tc:xacml:1.0:action:action-id" Category="urn:oasis:names:tc:xacml:3.0:attribute-category:action" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false" />
        </xacml:Match>
      </xacml:AllOf>
      <xacml:AllOf>
        <xacml:Match MatchId="urn:oasis:names:tc:xacml:3.0:function:string-equal-ignore-case">
          <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">write</xacml:AttributeValue>
          <xacml:AttributeDesignator AttributeId="urn:oasis:names:tc:xacml:1.0:action:action-id" Category="urn:oasis:names:tc:xacml:3.0:attribute-category:action" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false" />
        </xacml:Match>
      </xacml:AllOf>
    </xacml:AnyOf>

  </xacml:Target>
</xacml:Rule>
```

## Available Actions

`read`, `write`, `instantiate`, `complete`, `confirm`, `delete`, `pay`, `sign`, `reject`, `publish`, `subscribe`, `access`, `scopeaccess`

## Available Roles

| Role Name | Code | Description |
|-----------|------|-------------|
| Daglig leder | `DAGL` | Managing director (external role from ER) |
| Utfyller/Innsender | `UTINN` | Filler/submitter - broad access to forms |
| Regnskapsmedarbeider | `REGNA` | Accounting employee |
| Begrenset signeringsrettighet | `SISKD` | Limited signing rights |
| Revisorrettighet | `REVAI` | Auditor rights |
| Ansvarlig revisor | `A0237` | Responsible auditor with signing rights |
| Revisormedarbeider | `A0238` | Auditor employee (no signing) |
| Regnskapsforer med signeringsrettighet | `A0239` | Accountant with signing rights |
| Regnskapsforer uten signeringsrettighet | `A0240` | Accountant without signing rights |
| Regnskapsforer lonn | `A0241` | Accountant for salary services |
| Kontaktperson for NUF | `KNUF` | Contact person for foreign enterprises |
| Selvregistrert bruker | `SELN` | Self-registered user |

## Resource Attributes

| Attribute | Description | Typical Value |
|-----------|-------------|---------------|
| `urn:altinn:org` | Owning organization | `[ORG]` |
| `urn:altinn:app` | App identifier | `[APP]` |
| `urn:altinn:task` | Process task | Task name (e.g., `Task_1`) |
| `urn:altinn:event` | Event type | Event name |

## ObligationExpressions

Defines authentication level requirements. Placed after all rules:

```xml
<xacml:ObligationExpressions>
  <xacml:ObligationExpression ObligationId="urn:altinn:obligation:authenticationLevel1" FulfillOn="Permit">
    <xacml:AttributeAssignmentExpression AttributeId="urn:altinn:obligation1-assignment1" Category="urn:altinn:minimum-authenticationlevel">
      <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#integer">4</xacml:AttributeValue>
    </xacml:AttributeAssignmentExpression>
  </xacml:ObligationExpression>
  <xacml:ObligationExpression ObligationId="urn:altinn:obligation:authenticationLevel2" FulfillOn="Permit">
    <xacml:AttributeAssignmentExpression AttributeId="urn:altinn:obligation2-assignment2" Category="urn:altinn:minimum-authenticationlevel-org">
      <xacml:AttributeValue DataType="http://www.w3.org/2001/XMLSchema#integer">3</xacml:AttributeValue>
    </xacml:AttributeAssignmentExpression>
  </xacml:ObligationExpression>
</xacml:ObligationExpressions>
```

## Editing Guidelines

### Adding a New Rule
1. Assign a unique numeric RuleId (next number in sequence)
2. Place before `ObligationExpressions`
3. Include all three sections: Who, What, How
4. Only add the actions specified in the request

### Modifying a Rule with Multiple Roles
If you need different permissions for one role in a multi-role rule:
1. Remove that role from the original rule
2. Create a new rule for that role with the desired permissions

### Removing Access from a Role
1. Find the rule containing the role
2. Remove the entire `<xacml:AllOf>` block for the action to remove
3. If all actions are removed, remove the entire rule

## Validation Checklist

- [ ] Only specified access controls are added
- [ ] RuleId is unique and numeric
- [ ] New rules are placed before ObligationExpressions
- [ ] All XML tags are properly opened and closed
- [ ] Role code matches one from the roles table
- [ ] Actions are from the available actions list
