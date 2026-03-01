# Customization

## Change the standup format

Open `~/.claude/commands/standup.md` and find the section titled **"Generate Standup Report"**.
Replace the Yesterday / Today / Blockers template with your team's format:

**Scrum:** What I completed / What I'm working on / Impediments
**Async:** Done / In Progress / Needs Review / Blocked on
**Engineering manager:** Shipped / In Flight / At Risk / Decisions needed

Save the file. Changes apply immediately on next `/standup` run.

## Change the time window

Look back further than yesterday:
```bash
echo "2026-02-28T00:00:00Z" > ~/.standup_last_run
```

Reset to default (yesterday):
```bash
rm ~/.standup_last_run
```

## Per-project install

To use a different format for a specific repo, copy the file into that project instead:

```bash
mkdir -p .claude/commands
cp ~/.claude/commands/standup.md .claude/commands/standup.md
```

Edit `.claude/commands/standup.md` in that repo. Claude Code will use the project-level version over the global one.

## Jira / Linear integration

Jira sync is optional and off by default. Provide your project key when prompted with `Post to Jira? (y/n):`.

Linear is not built in — to add it, edit your local `~/.claude/commands/standup.md` and replace the Jira block with your Linear API call.
