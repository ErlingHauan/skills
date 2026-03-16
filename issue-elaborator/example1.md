# Persist Studio chat threads in a database

## Summary

Persist AI assistant chat threads in a database so user conversations and context are retained across sessions in Altinn Studio.

This is required to provide a coherent and trustworthy user experience during beta.

## Context

The Studio AI assistant currently relies on in-memory or ephemeral state for conversations. This means chat history can be lost across reloads, navigation, or new sessions, leading to a fragmented and confusing user experience.

For beta usage, users must be able to return to ongoing conversations and understand prior context and decisions.

## Goal

Ensure chat threads and relevant context are stored and retrieved from persistent storage so conversations behave like a first-class Studio feature rather than an experimental session.

## In scope

- Persist chat threads and messages in a database
- Associate chat threads with the correct user, app, and Studio context
- Restore conversation state when the user returns
- Ensure persistence integrates cleanly with existing Studio backend flows

## Out of scope

- Advanced conversation analytics
- Cross-app or cross-user chat sharing
- Post-beta UX refinements
- Evaluation or quality measurement of responses

## Acceptance criteria

- Chat threads are persisted and survive page reloads and sessions
- Users can resume previous conversations without losing context
- Chat history is correctly scoped to the relevant Studio app and user
- No persistence-related errors degrade the Studio experience