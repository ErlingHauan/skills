---
name: issue-elaborator
description: Create GitHub issue markdown with codebase context
---

# GitHub Issue Formatter
Transform an issue draft into a well-structured GitHub issue.

## Instructions
You will receive an issue draft from the user. Your task is to transform it into a well-structured GitHub issue in markdown format.
Use `./example1.md`, `./example2.md`, and `./example3.md` for reference.
Follow the guidelines below.

### Required Sections
- Description
- Context
- In scope
- Out of scope
- Acceptance criteria

## Codebase search
Scan the codebase for relevant context. Keep it simple. The intention is not to create a complete plan to solve the issue. You should rather sprinkle relevant references here and there.

### Style Guidelines
- Keep it simple
- No line number references

### Output
- Return only the formatted GitHub issue in in raw markdown code block format
- No preamble or explanation about your search process
- Do not update the remote issue unless asked to do so
