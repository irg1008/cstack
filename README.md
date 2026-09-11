# cstack

A Claude Code port of [pstack](https://github.com/backnotprop/pstack) by Lauren Tan.

pstack is a Cursor plugin. cstack is the same 45 skills and 2 agents repackaged as a Claude Code plugin marketplace, with the Cursor-specific paths, tool names and model slugs rewritten for Claude Code.

## Install

```bash
/plugin marketplace add irg1008/cstack
/plugin install cstack@cstack
```

Then run `/setup-cstack` once. It checks for bun (poteto-mode's orchestration scripts need it) and writes your per-role model choices.

The issue automations are a separate plugin:

```bash
/plugin install cstack-benny@cstack
```

## What's in it

**Workflow** — `poteto-mode` (the umbrella style, 22 playbooks), `architect`, `figure-it-out`, `arena`, `swarm`, `show-me-your-work`, `setup-cstack`

**Investigation** — `how`, `why`, `teach`, `recall`, `blast-radius`, `bro`

**Review** — `interrogate`, `reflect`, `no-comments`, `unslop`, `deslop`, `tdd`

**Verification** — `create-verification-skill`, `maintain-verification-skill`

**Writing** — `technical-writing`, `typescript-best-practices`, `automate-me`

**Principles** — 21 `principle-*` skills, one idea each, meant to apply automatically

**Agents** — `comment-sicko`, `poteto-agent`

## What changed from pstack

Paths and manifests, mostly. Four changes are worth knowing about.

**Model panels are less independent.** pstack's `how` critics, `arena` runners, `architect` runners and `interrogate` reviewers each spawn four subagents on four different vendors' models. Claude Code's `Agent` tool takes `opus`, `fable`, `sonnet`, `haiku` and nothing else, so panels now run four Claude models. The reviewer prompts still attack from independent angles, which is where most of the value was, but a panel here can agree with itself more readily than pstack's would.

**Cursor tool parameters became Claude Code ones.** `Task` is `Agent`. `generalPurpose` is `general-purpose`. `readonly: true` is `subagent_type: Explore`. `environment: "cloud"` is `isolation: "remote"`.

**`deslop` is vendored in.** pstack references a `deslop` skill from the separate `cursor-team-kit` Cursor plugin, which has no Claude Code equivalent. That skill is included here so cstack stands alone.

**`control-ui` and `control-cli` have no equivalent.** Those also live in `cursor-team-kit`. References to them now point at `create-verification-skill`, which generates a project-local `verify-*` skill that does the same job.

Benny carries one unported piece: Cursor's built-in automation editor has no Claude Code counterpart. The skills, prompt templates and boundaries are all here with `.claude/` paths, but you wire the trigger yourself. `plugins/cstack-benny/RUNNERS.md` covers the two paths, GitHub Actions or scheduled polling, with working workflow YAML.

## Credits

All skill content is Lauren Tan's, MIT licensed. See [backnotprop/pstack](https://github.com/backnotprop/pstack).
