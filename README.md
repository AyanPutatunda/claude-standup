# claude-standup

A Claude Code slash command that auto-generates your daily standup from git commits and Claude Code session activity — no manual log-digging required.

## What It Does

Run `/standup` in any Claude Code session and get a ready-to-share standup in seconds. It:

- Pulls **git commits** since your last standup
- Reads **Claude Code session files** for files edited and tasks worked on
- Formats everything into a clean **Yesterday / Today / Blockers** report — TODAY is suggested based on context, so review it before sharing
- Optionally posts to **Jira** (or you copy-paste it yourself)

On first run it looks back to "yesterday". After that it tracks from where it last ran, so you only see new work each time.

## 30-Second Setup

```bash
# 1. Clone the repo
git clone https://github.com/AyanPutatunda/claude-standup.git

# 2. Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# 3. Copy the command
cp claude-standup/commands/standup.md ~/.claude/commands/standup.md

# 4. (Optional) Install jq for richer session parsing
brew install jq        # macOS
sudo apt-get install jq  # Ubuntu/Debian
```

That's it. Open any Claude Code session and type `/standup`.

> **Works across all your projects.** Installing to `~/.claude/commands/` makes `/standup` available globally in every repo you work in.

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

## Customize It

The standup format, time window, and git author filter are all editable
in `~/.claude/commands/standup.md`. See [CUSTOMIZATION.md](CUSTOMIZATION.md)
for common team formats and time window options.

The file is plain markdown — no build step, no package manager. Edit it and the next `/standup` run picks up your changes immediately.

## For Developers

This repo is read-only (no PRs accepted). The intended workflow is:

1. Copy `commands/standup.md` to `~/.claude/commands/standup.md`
2. Edit that file directly — it's your local copy, customize freely
3. To share your version: fork the repo, commit your edits, share your fork URL

The entire command is one markdown file. Fork it, own it, make it yours.

## Requirements

- [Claude Code](https://claude.ai/code)
- Git with `user.email` configured (`git config --global user.email "you@example.com"`)
- `jq` for session file parsing (optional — graceful fallback if missing)

## License

MIT
