# claude-standup

A Claude Code slash command that auto-generates your daily standup from git commits and Claude Code session activity — no manual log-digging required.

## What It Does

Run `/standup` in any Claude Code session and get a ready-to-share standup in seconds. It:

- Pulls **git commits** since your last standup
- Reads **Claude Code session files** for files edited and tasks worked on
- Formats everything into a clean **Yesterday / Today / Blockers** report
- Optionally posts to **Jira** (or you copy-paste it yourself)

## 30-Second Setup

```bash
# 1. Clone the repo
git clone https://github.com/AyanPutatunda/claude-standup.git

# 2. Copy the skill to Claude Code's global skills directory
cp -r claude-standup/.claude/skills/standup ~/.claude/skills/standup

# 3. (Optional) Install jq for richer session parsing
brew install jq   # macOS
```

That's it. Open any Claude Code session and type `/standup`.

## Example Output

```
📅 STANDUP UPDATE — March 1, 2026

YESTERDAY:
• Initialized claude-standup project repository (commit: d9f726e)
• Implemented /standup slash command with multi-agent research team
  (commit: 79bca8d)
• Built standup skill pipeline: git data collection, session activity
  extraction, and formatted report generation
• Developed build artifacts for standup blocks via Claude Code:
  format-block.md, git-block.md, session-block.md

TODAY:
• Test and refine the /standup skill end-to-end
• Explore Jira integration for automated standup posting
• Review agent team output quality and tune prompt formatting

BLOCKERS:
• None

---
Post to Jira? (y/n):
```

## Jira Integration

Jira sync is **optional**. After the report is displayed, you'll be asked `Post to Jira? (y/n):`. Say `n` and you can copy-paste the output manually. Say `y` to configure posting to a Jira project.

## Requirements

- [Claude Code](https://claude.ai/code)
- Git with `user.email` configured
- `jq` for session file parsing (optional — graceful fallback if missing)

## License

MIT
