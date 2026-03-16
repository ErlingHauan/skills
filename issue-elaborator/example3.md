Enable Studio development of MCP Process tooling

## Summary

Document and prepare the groundwork for a future MCP tool related to Altinn Studio process handling, with the explicit goal of enabling the Altinn Studio team to develop this independently after handover.

This issue is **post-beta** and is not intended to block initial beta rollout.

## Context

Altinn Studio applications define their business process in a `process.bpmn` file.  
Each process must contain at least one task, and tasks can be either interactive (data tasks) or server-side service tasks.

Altinn supports custom process tasks through interfaces such as `IProcessTask`, and service owners can implement their own service tasks. However, upcoming changes in Altinn Studio 9.0 are expected to introduce breaking changes to these interfaces.

Because of this, early coupling to current process/task interfaces carries risk.

## Intent

The intent of this issue is **not to implement a tool**, but to:

- Capture known constraints and risks around process/task tooling
- Clarify potential MCP use cases related to process inspection or reasoning
- Provide a starting point for the Altinn Studio team to design and implement such a tool after handover

## Potential future scope (for Studio)

Examples of questions a future MCP tool *could* help with:
- Inspecting and explaining process definitions in `process.bpmn`
- Reasoning about task flow and transitions
- Supporting service task integration patterns
- Validating or visualizing process structure

These are illustrative only, not commitments.

## Constraints and considerations

- Breaking changes expected in Altinn Studio 9.0
- Stability of `IProcessTask` and related interfaces is uncertain
- Tool design should align with Studio-owned APIs and conventions
- Ownership and implementation should live with the Studio team

## Out of scope

- Implementing a production MCP tool
- Any work that affects beta readiness
- Tight coupling to pre-9.0 process/task interfaces

## Status

Post-beta enablement note.

This issue is intentionally kept as documentation and context to support future Studio-owned development after handover.