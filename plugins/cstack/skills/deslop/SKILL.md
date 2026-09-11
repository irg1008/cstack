---
name: deslop
description: Remove AI-generated code slop from a branch or diff. Use when the user says "deslop", "remove AI slop", "clean up the AI code", or asks to strip over-commenting, unnecessary defensive try/catch, `as any` casts, or styling that doesn't match the surrounding file. Run before opening a PR or committing agent-written code.
---

# Remove AI code slop

Check the diff against main, and remove all AI generated slop introduced in this branch.

This includes:

- Extra comments that a human wouldn't add or is inconsistent with the rest of the file
- Extra defensive checks or try/catch blocks that are abnormal for that area of the codebase (especially if called by trusted / validated codepaths)
- Casts to `any` to get around type issues
- Any other style that is inconsistent with the file

Respect the codebase's own dialect — match the voice of the surrounding code rather than importing patterns from elsewhere.

Report at the end with only a 1-3 sentence summary of what you changed.
