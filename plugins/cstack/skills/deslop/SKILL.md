---
name: deslop
description: Remove AI-generated code slop from a branch or diff. Use when the user says "deslop", "remove AI slop", "clean up the AI code", or asks to strip over-commenting, unnecessary defensive try/catch, `as any` casts, deep nesting, or styling that doesn't match the surrounding file. Run before opening a PR or committing agent-written code.
---

# Remove AI code slop

Check the diff against main, and remove all AI generated slop introduced in this branch.

## Focus areas

- Extra comments that a human wouldn't add, or that are inconsistent with the rest of the file
- Extra defensive checks or try/catch blocks that are abnormal for that area of the codebase, especially on trusted or already-validated codepaths
- Casts to `any` used only to get around type issues
- Deeply nested code that early returns would flatten
- Any other pattern inconsistent with the file and the surrounding codebase

Respect the codebase's own dialect. Match the voice of the surrounding code rather than importing patterns from elsewhere.

## Guardrails

- Keep behavior unchanged unless you are fixing a clear bug.
- Prefer minimal, focused edits over broad rewrites.
- Report at the end with only a 1-3 sentence summary of what you changed.
