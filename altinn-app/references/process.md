# Altinn Process Reference

## Overview

Every Altinn app has a process defined in `App/config/process/process.bpmn`. The process controls the workflow: which steps the user goes through, in what order, and under what conditions. It uses BPMN (Business Process Model and Notation) format.

A newly created app comes with a single `data` task by default. You can add more tasks (confirmation, signing, payment, feedback) and connect them with gateways for conditional flow.

## BPMN File Structure

The process file has this basic structure:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL"
                  xmlns:altinn="http://altinn.no/process"
                  id="Definitions_1">
  <bpmn:process id="Process_1" isExecutable="false">
    <bpmn:startEvent id="StartEvent_1">
      <bpmn:outgoing>Flow_1</bpmn:outgoing>
    </bpmn:startEvent>

    <bpmn:task id="Task_1" name="Utfylling">
      <bpmn:incoming>Flow_1</bpmn:incoming>
      <bpmn:outgoing>Flow_2</bpmn:outgoing>
      <bpmn:extensionElements>
        <altinn:taskExtension>
          <altinn:taskType>data</altinn:taskType>
        </altinn:taskExtension>
      </bpmn:extensionElements>
    </bpmn:task>

    <bpmn:endEvent id="EndEvent_1">
      <bpmn:incoming>Flow_2</bpmn:incoming>
    </bpmn:endEvent>

    <bpmn:sequenceFlow id="Flow_1" sourceRef="StartEvent_1" targetRef="Task_1" />
    <bpmn:sequenceFlow id="Flow_2" sourceRef="Task_1" targetRef="EndEvent_1" />
  </bpmn:process>
</bpmn:definitions>
```

Key elements:
- `bpmn:startEvent` — process entry point
- `bpmn:task` — a step in the process (with Altinn-specific `taskType`)
- `bpmn:serviceTask` — an automated server-side step
- `bpmn:exclusiveGateway` — conditional branching
- `bpmn:endEvent` — process completion
- `bpmn:sequenceFlow` — connects elements together

## Task Types

### Data Task

The default task type. Enables users/systems to read, write, and modify form data. All data must be valid before the process can advance.

```xml
<bpmn:task id="Task_1" name="Utfylling">
  <bpmn:incoming>Flow_1</bpmn:incoming>
  <bpmn:outgoing>Flow_2</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>data</altinn:taskType>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

### Confirmation Task

Allows the user to confirm the submitted data. Creates an audit log recording who confirmed and when.

```xml
<bpmn:task id="Task_2" name="Bekreftelse">
  <bpmn:incoming>Flow_2</bpmn:incoming>
  <bpmn:outgoing>Flow_3</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>confirmation</altinn:taskType>
      <altinn:actions>
        <altinn:action>confirm</altinn:action>
      </altinn:actions>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

### Signing Task

Enables digital signing of data. Generates a signature object with a hash of the specified data elements and logs who signed and when.

```xml
<bpmn:task id="Task_3" name="Signering">
  <bpmn:incoming>Flow_3</bpmn:incoming>
  <bpmn:outgoing>Flow_4</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>signing</altinn:taskType>
      <altinn:actions>
        <altinn:action>sign</altinn:action>
      </altinn:actions>
      <altinn:signatureConfig>
        <altinn:dataTypesToSign>
          <altinn:dataType>model</altinn:dataType>
        </altinn:dataTypesToSign>
        <altinn:signatureDataType>signatures</altinn:signatureDataType>
        <altinn:uniqueFromSignaturesInDataTypes>
          <altinn:dataType>previousSignatures</altinn:dataType>
        </altinn:uniqueFromSignaturesInDataTypes>
      </altinn:signatureConfig>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

Signing config properties:
- `dataTypesToSign` — which data elements are signed
- `signatureDataType` — where the signature object is stored (must be registered in `applicationmetadata.json`)
- `uniqueFromSignaturesInDataTypes` — ensures the signer hasn't already signed in a previous step (references other signature data types)
- `runDefaultValidator` — set to `true` to run data validation before signing (used in sign-and-submit)

The signature data type must be registered in `applicationmetadata.json`:
```json
{
  "id": "signatures",
  "allowedContentTypes": ["application/json"],
  "allowedContributors": ["app:owned"],
  "maxCount": 1,
  "minCount": 1
}
```

#### Sign-and-Submit Pattern

Combines data entry and signing into a single task. Change the data task's `taskType` to `signing` and add the `sign` action:

```xml
<bpmn:task id="Task_1" name="Fyll ut og signer">
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>signing</altinn:taskType>
      <altinn:actions>
        <altinn:action>sign</altinn:action>
      </altinn:actions>
      <altinn:signatureConfig>
        <altinn:dataTypesToSign>
          <altinn:dataType>model</altinn:dataType>
        </altinn:dataTypesToSign>
        <altinn:signatureDataType>signatures</altinn:signatureDataType>
        <altinn:runDefaultValidator>true</altinn:runDefaultValidator>
      </altinn:signatureConfig>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

Remember to update `policy.xml` to grant the `sign` action to the relevant roles.

### Payment Task

Handles payment processing. Supports three user actions: `pay`, `confirm`, and `reject`.

```xml
<bpmn:task id="Task_Payment" name="Betaling">
  <bpmn:incoming>Flow_4</bpmn:incoming>
  <bpmn:outgoing>Flow_5</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>payment</altinn:taskType>
      <altinn:actions>
        <altinn:action>pay</altinn:action>
        <altinn:action>confirm</altinn:action>
        <altinn:action>reject</altinn:action>
      </altinn:actions>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

Payment requires:
1. **Data types** in `applicationmetadata.json` for payment information and receipt PDF
2. **`IOrderDetailsCalculator`** implementation registered in `Program.cs`:
   ```csharp
   services.AddTransient<IOrderDetailsCalculator, OrderDetailsCalculator>();
   ```
3. **Payment processor config** in `appsettings.json` (e.g., Nets Easy settings)
4. **An exclusive gateway** after the payment task to handle confirm vs reject
5. **Authorization** for `read`, `write`, `pay`, `confirm`, and `reject` actions on the payment task
6. **Layout** with `PaymentDetails` component and a payment page at `App/ui/payment/payment.json`

### Feedback Task

Enables the service owner to provide feedback to the user while allowing continued interaction.

```xml
<bpmn:task id="Task_Feedback" name="Tilbakemelding">
  <bpmn:incoming>Flow_5</bpmn:incoming>
  <bpmn:outgoing>Flow_6</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>feedback</altinn:taskType>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:task>
```

## Service Tasks (Automated)

Service tasks run automatically on the server without user interaction. The process advances to the next step when the task finishes. Requires NuGet package version 8.9.0+.

### Built-in Service Tasks
- **PDF generation** — auto-generates PDF documents
- **eFormidling** — document distribution
- **Fiks Arkiv** — archival operations

### Custom Service Tasks

Use `bpmn:serviceTask` (not `bpmn:task`) in the BPMN:

```xml
<bpmn:serviceTask id="ServiceTask_1" name="My custom task">
  <bpmn:incoming>Flow_3</bpmn:incoming>
  <bpmn:outgoing>Flow_4</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:taskExtension>
      <altinn:taskType>myCustomTask</altinn:taskType>
    </altinn:taskExtension>
  </bpmn:extensionElements>
</bpmn:serviceTask>
```

Implement the `IServiceTask` interface:

```csharp
public class MyCustomTask : IServiceTask
{
    public string Type => "myCustomTask";

    public async Task<ServiceTaskResult> Execute(ServiceTaskContext context)
    {
        // Access instance data via context.InstanceDataMutator
        // Return ServiceTaskResult.Success() or ServiceTaskResult.FailedAbortProcessNext()
    }
}
```

Register in `Program.cs`:
```csharp
services.AddTransient<IServiceTask, MyCustomTask>();
```

The `Type` property must match the `taskType` in the BPMN file.

Custom service tasks require authorization as write operations. Add the task type as an action in `policy.xml` using the standard XACML action match against `urn:oasis:names:tc:xacml:1.0:action:action-id`.

## Flow Control with Exclusive Gateways

Exclusive gateways route the process based on conditions, user actions, or custom logic. Only one outgoing flow may be selected.

### Gateway Structure

```xml
<bpmn:exclusiveGateway id="Gateway_1">
  <bpmn:incoming>Flow_from_task</bpmn:incoming>
  <bpmn:outgoing>Flow_option_a</bpmn:outgoing>
  <bpmn:outgoing>Flow_option_b</bpmn:outgoing>
</bpmn:exclusiveGateway>
```

### Expression-Based Conditions

Use the same expression syntax as dynamic expressions in layouts. The gateway must reference a data type:

```xml
<bpmn:exclusiveGateway id="Gateway_1">
  <bpmn:incoming>Flow_2</bpmn:incoming>
  <bpmn:outgoing>Flow_high</bpmn:outgoing>
  <bpmn:outgoing>Flow_low</bpmn:outgoing>
  <bpmn:extensionElements>
    <altinn:gatewayExtension>
      <altinn:connectedDataTypeId>model</altinn:connectedDataTypeId>
    </altinn:gatewayExtension>
  </bpmn:extensionElements>
</bpmn:exclusiveGateway>

<bpmn:sequenceFlow id="Flow_high" sourceRef="Gateway_1" targetRef="Task_Review">
  <bpmn:conditionExpression>["greaterThanEq", ["dataModel", "amount"], 1000]</bpmn:conditionExpression>
</bpmn:sequenceFlow>

<bpmn:sequenceFlow id="Flow_low" sourceRef="Gateway_1" targetRef="Task_AutoApprove">
  <bpmn:conditionExpression>["lessThan", ["dataModel", "amount"], 1000]</bpmn:conditionExpression>
</bpmn:sequenceFlow>
```

### Action-Based Routing

Route based on which action the user performed in the preceding task (common after payment tasks):

```xml
<bpmn:sequenceFlow id="Flow_confirmed" sourceRef="Gateway_1" targetRef="Task_Next">
  <bpmn:conditionExpression>["equals", ["gatewayAction"], "confirm"]</bpmn:conditionExpression>
</bpmn:sequenceFlow>

<bpmn:sequenceFlow id="Flow_rejected" sourceRef="Gateway_1" targetRef="Task_Previous">
  <bpmn:conditionExpression>["equals", ["gatewayAction"], "reject"]</bpmn:conditionExpression>
</bpmn:sequenceFlow>
```

### Custom Gateway Logic

For complex routing, implement `IProcessExclusiveGateway`:

```csharp
public class MyGateway : IProcessExclusiveGateway
{
    public string GatewayId => "Gateway_1";

    public async Task<List<SequenceFlow>> FilterAsync(
        List<SequenceFlow> outgoingFlows,
        Instance instance)
    {
        // Return only the flows that should be available
    }
}
```

Register in `Program.cs`:
```csharp
services.AddTransient<IProcessExclusiveGateway, MyGateway>();
```

The `GatewayId` must match the gateway's `id` in the BPMN file.

## Common Process Patterns

### Simple: Data -> End
```
StartEvent -> Data Task -> EndEvent
```

### Data -> Confirmation -> End
```
StartEvent -> Data Task -> Confirmation Task -> EndEvent
```

### Data -> Signing -> End
```
StartEvent -> Data Task -> Signing Task -> EndEvent
```

### Data -> Payment with reject loop
```
StartEvent -> Data Task -> Payment Task -> Gateway
  -> (confirm) -> EndEvent
  -> (reject)  -> Data Task
```

### Sign-and-Submit (single step)
```
StartEvent -> Signing Task (with data entry) -> EndEvent
```

## Checklist When Modifying the Process

- [ ] All tasks have unique `id` attributes
- [ ] Every task has matching `incoming` and `outgoing` flows
- [ ] All `sequenceFlow` elements have valid `sourceRef` and `targetRef`
- [ ] `taskType` matches a valid type (`data`, `confirmation`, `signing`, `payment`, `feedback`) or a registered custom service task
- [ ] Signing tasks have `signatureConfig` with required properties
- [ ] Signature and payment data types are registered in `applicationmetadata.json`
- [ ] `policy.xml` grants the correct actions for each task (e.g., `sign`, `pay`, `confirm`, `reject`)
- [ ] Exclusive gateways have condition expressions on all outgoing flows (or a default flow)
- [ ] Custom service task `Type` property matches the BPMN `taskType`
- [ ] Service registrations in `Program.cs` for any custom implementations
