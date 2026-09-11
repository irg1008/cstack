# Runners

Benny's skills describe *what* each automation does. They do not start it. Cursor had a built-in automation editor; Claude Code does not, so you wire the trigger yourself.

Two paths. Pick one per automation.

## Path 1: GitHub Actions (recommended for Slack triggers)

Benny's trigger is "someone posted a new top-level message in a Slack channel". GitHub Actions cannot see that directly, so you need one hop: a Slack Events API subscription that fires a `repository_dispatch` at your repo.

1. Create a Slack app with the `message.channels` event subscription and `chat:write`, `channels:history` scopes.
2. Point its Request URL at something that forwards to GitHub's `repository_dispatch` API with the thread coordinates (channel id, `ts`) in `client_payload`. A tiny Cloudflare Worker or Lambda is enough.
3. Add a workflow that runs Claude Code on that dispatch:

```yaml
name: benny-triage
on:
  repository_dispatch:
    types: [slack-issue-report]
jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Read and follow .claude/automations/benny/skills/triage-issue-reports/SKILL.md
            for every run. Thread coordinates:
            channel=${{ github.event.client_payload.channel }}
            ts=${{ github.event.client_payload.ts }}
```

Use a second workflow for `reproduce-and-fix-issues`. Give that one `contents: write` and `pull-requests: write`, since it opens draft PRs.

Slack credentials go in repository secrets. Benny's rules forbid handing them to subagents; keep them in the workflow environment, not in prompts.

## Path 2: Scheduled polling

If you would rather not run an event forwarder, poll instead. Cheaper to set up, slower to react, and it costs a run every interval whether or not there is work.

Use the `/schedule` skill to create a cloud routine, or a `schedule:` trigger in the same workflow shape above. The skill then lists new top-level messages since the last watermark and processes each. Store the watermark in a committed file or a repo variable so restarts are idempotent, which is what `principle-make-operations-idempotent` is for.

## What stays true either way

Benny's own boundaries do not change with the runner:

- Never post a root message in the source channel. Replies stay in the original thread.
- Channel and root thread coordinates are immutable for the whole run.
- Draft pull requests only. Never merge, never deploy.
- Fail closed when channel coordinates, tracker access, the control adapter, or the feature map are missing or uncertain.
- Subagents may help but never receive Slack credentials and never post to Slack.

Read the automation's `SKILL.md` for the rest. The runner only decides when it starts.
