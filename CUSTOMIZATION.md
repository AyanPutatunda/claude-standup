# Customization

## Change the standup format

Open `~/.claude/commands/standup.md` and find the output format section.
Replace the Yesterday / Today / Blockers template with your team's format:

**Scrum:** What I completed / What I'm working on / Impediments
**Async:** Done / In Progress / Needs Review / Blocked on
**Engineering manager:** Shipped / In Flight / At Risk / Decisions needed

Save the file. Changes apply immediately on next `/standup` run.
Pull requests for new formats welcome.

## Change the time window

```bash
echo "2026-02-28T00:00:00Z" > ~/.standup_last_run
```

Run `/standup` — it will pull everything since that date.

## Jira / Linear integration

Jira sync is optional and off by default. Provide your project key when prompted with `Post to Jira? (y/n):`. Linear support: open a PR.
