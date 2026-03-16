---
name: react-unnecessary-effects
description: >
  Review React code for unnecessary useEffect usage and suggest better alternatives.
  Use this skill when the user asks to review React components for effect issues,
  when refactoring React code that uses useEffect, or when the user is writing
  new useEffect code that might not need to be an effect.
---

# You Might Not Need an Effect

Effects are an escape hatch from the React paradigm. They let you synchronize components with external systems like browser APIs, third-party widgets, or network requests. If there's no external system involved, you probably don't need an Effect.

## When to apply this skill

Scan for these two broad categories of unnecessary Effects:

1. **Effects that transform data for rendering** — if a value can be calculated from existing props or state, calculate it during render instead. This avoids an extra render pass where the component first renders with a stale value, then re-renders with the updated one.

2. **Effects that respond to user events** — if code should run because the user did something (clicked a button, submitted a form), put it in the event handler. Effects are for code that runs because the component was *displayed*, not because the user *interacted*.

## How to review

Read the component's `useEffect` calls one by one. For each, ask: **"Why does this code run?"**

- If the answer is "because prop/state X changed and I need to compute Y" → it's derived state, compute it during render.
- If the answer is "because the user clicked/submitted/dragged" → move it to the event handler.
- If the answer is "because this component appeared on screen and I need to sync with an external system" → this is a legitimate Effect.

When you find an unnecessary Effect, suggest the specific alternative pattern. See `references/patterns.md` for the full catalog of patterns with before/after examples.

## Quick reference of patterns

| # | Anti-pattern | Better alternative |
|---|---|---|
| 1 | State + Effect to derive a value from props/state | Compute during render |
| 2 | State + Effect to cache expensive computation | `useMemo` |
| 3 | Effect to reset all state when a prop changes | `key` prop on the component |
| 4 | Effect to adjust some state when a prop changes | Derive the value during render |
| 5 | Effect to share logic between event handlers | Extract a shared function, call from handlers |
| 6 | Effect to send a POST on form submit | Move to event handler (keep analytics Effects) |
| 7 | Chain of Effects that trigger each other | Calculate all next state in the event handler |
| 8 | Effect for one-time app initialization | Top-level variable guard or module-level init |
| 9 | Effect to notify parent of state changes | Call parent callback in the event handler alongside `setState` |
| 10 | Effect to pass data up to parent | Lift the data fetching to the parent |
| 11 | Effect to subscribe to an external store | `useSyncExternalStore` |
| 12 | Effect to fetch data without cleanup | Add cleanup with `ignore` flag to prevent race conditions |

## Key principles

- **Derived state is not state.** If `fullName = firstName + " " + lastName`, there is no reason for `fullName` to be in `useState`. Compute it directly. This is faster, simpler, and prevents values from getting out of sync.

- **State updates during events, not after.** React batches state updates within the same event handler into a single render. When you move the logic from an Effect into the event handler, you eliminate an extra render pass.

- **The `key` prop resets component identity.** When you need to fully reset a component's internal state because a prop changed (e.g., different user profile), render it with `key={prop}` instead of using an Effect to manually clear state.

- **Lift state up instead of syncing state across components.** If two components need the same state, move it to their common parent rather than using Effects to keep separate copies in sync.

- **`useSyncExternalStore` exists for a reason.** Subscribing to browser APIs or external stores is a common need — use the purpose-built hook instead of manual Effect subscriptions.
