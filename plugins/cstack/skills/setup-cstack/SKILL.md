---
name: setup-cstack
description: One-time setup for cstack. Installs the bun toolchain that poteto-mode's orchestration scripts need, and writes the per-role model rule that overrides the skill defaults. Use for /setup-cstack, "set up cstack", "configure cstack models", or when a cstack script fails to run.
---

# Setup cstack

Two jobs, in order. Run step 1 once per machine. Re-run step 2 whenever you want different model choices.

## 1. Install the script toolchain

`poteto-mode` ships TypeScript tools that the orchestrate, babysit and shipping playbooks call: `scripts/orch/orch.ts` (unit and board state) and `scripts/watch-pr/` (PR polling). They run on bun.

Check for bun:

```
bun --version
```

If it is missing, tell the user to install it and stop. Do not install it for them.

- macOS and Linux: `curl -fsSL https://bun.sh/install | bash`
- Windows: `powershell -c "irm bun.sh/install.ps1 | iex"`

Once bun is present, nothing else is required. `scripts/bootstrap.ts` hashes `package.json` and `bun.lock`, compares against `node_modules/.poteto-mode-tools-install-key`, and runs `bun install` itself when they drift. Every entry point imports it, so dependencies self-heal on first call.

To verify the install eagerly, run this from the plugin's `skills/poteto-mode/scripts/` directory:

```
bun test orch watch-pr
```

A green run means the toolchain is ready. A failure here is a real problem; report it rather than working around it.

## 2. Write the model rule

Write `~/.claude/cstack-models.md`, referenced from `~/.claude/CLAUDE.md` so it applies to every session. The skills read it and fall back to their inline defaults when a line is absent, so this is an override layer, not a requirement.

### 2a. Know the available models

Claude Code's `Agent` tool accepts a fixed set, not arbitrary vendor slugs:

| value | use it for |
| --- | --- |
| `opus` | hardest reasoning, precise instruction-following, cross-cutting design |
| `fable` | judgment and prose |
| `sonnet` | general code work, the default worker |
| `haiku` | trivial mechanical edits, cheap fan-out |
| `inherit-parent` / `auto` | run on the parent chat model; omit `model` on the Agent call |

This is the whole set. Never write a value outside it.

This is the one place cstack diverges from pstack in substance. pstack's panel roles (`how` critics, `arena` runners, `architect` runners, `interrogate` reviewers) draw diversity from four different vendors' models. Claude Code has one vendor, so a panel gets four Claude models instead. Diversity now comes mostly from the reviewer prompts, which already attack from independent angles, rather than from the model families. Panels stay useful; they are less independent than pstack's. Say so if a user asks why a panel agreed with itself.

### 2b. Load current state

The defaults are the rule shape in step 2d. If `~/.claude/cstack-models.md` already exists, read it and treat its values as the current choices. Otherwise start from those defaults.

### 2c. Map and confirm

Show every role with its current model. Ask whether to accept as-is or change specific roles, offering the five values above. Prefer AskUserQuestion over free text.

For panel roles the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the fan-out. `arena cross-judge pool` is also a list, but Arena selects one value from it, preferring one that differs from the parent's model. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 2d. Write the rule

Overwrite the whole file so re-runs stay idempotent.

```
# cstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent call's `model`).
# Alias entries in a panel list still count toward its fan-out.
feature, refactoring: sonnet
bug-fix: opus
perf-issue: opus
hillclimb: opus
judgment and prose: fable
hardest tasks: opus
how explorer: sonnet
how explainer: fable
how critics: fable, opus, sonnet, haiku
why investigators: sonnet
why synthesizer: fable
reflect tooling: opus
reflect judgment, divergent, synthesizer: fable
arena runners: fable, opus, sonnet, haiku
arena cross-judge pool: fable, opus, sonnet
swarm workers: sonnet
architect runners: fable, opus, sonnet, haiku
interrogate reviewers: fable, opus, sonnet, haiku
```

Then make sure `~/.claude/CLAUDE.md` points at it. Append this line if it is not already there:

```
@~/.claude/cstack-models.md
```

### 2e. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

## 3. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill`. On no, move on without pushing.
