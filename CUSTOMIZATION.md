# Customization

## Change the Output Format

Open `commands/standup.md` and find the template under **"Generate Standup Report"**. Replace the Yesterday/Today/Blockers structure with your team's format. Example alternatives:

**Scrum format (default):**
```
YESTERDAY / TODAY / BLOCKERS
```

**Shape Up format:**
```
SHIPPED / BUILDING / STUCK
```

**Async-first format:**
```
DONE / NEXT / NEEDS REVIEW
```

Just swap the section headers and update the formatting rules below them to match.

## Change the Time Window

By default, `/standup` looks back to your last run (or "yesterday" on first use). To look back further:

```bash
echo "2026-02-28T00:00:00Z" > ~/.standup_last_run
```

Then run `/standup` — it will pull everything since that date.

## Jira Integration

Jira sync is optional and off by default. To enable, provide your project key when prompted with `Post to Jira? (y/n):`. No config file needed.
