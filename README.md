# /standup

A Claude Code slash command for generating daily standup reports.

## Overview

The `/standup` command automatically generates professional standup summaries by collecting:
- **Git commits** since your last standup
- **Claude Code session activity** (files edited, tasks completed)
- **Formatted output** in Yesterday/Today/Blockers structure

## Installation

This is a Claude Code skill. To use it:

1. Clone this repository or copy `.claude/skills/standup/` to your project
2. For system-wide access, copy to `~/.claude/skills/standup/`
3. Ensure you have `jq` installed for session file parsing (optional but recommended)

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq
```

## Usage

In any Claude Code session:

```
/standup
```

The command will:
1. Collect git commits since last run (defaults to "yesterday" on first use)
2. Parse Claude Code session files for work activity
3. Generate a formatted standup report
4. Ask if you want to post to Jira

### Example Output

```
📅 STANDUP UPDATE — March 1, 2026

YESTERDAY:
• Implemented user authentication flow (commits: a1b2c3d, e4f5g6h)
• Fixed bug in password reset logic (commit: i7j8k9l)
• Code review for PR #234 - authentication refactor
• Modified 3 files in auth module via Claude Code

TODAY:
• Deploy authentication changes to staging
• Write tests for password reset flow
• Team sync at 2pm to discuss auth rollout

BLOCKERS:
• None

---
Post to Jira? (y/n):
```

## How It Works

### Shell Preprocessing (`!` prefix)

The command uses shell preprocessing to collect data before AI processing:

```bash
# Git commits since last standup
!git log --after=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday") \
  --author=$(git config user.email) --oneline

# Current working changes
!git diff --stat HEAD

# Stashed work
!git stash list

# Session file parsing
!find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$LAST_RUN"
```

### Timestamp Tracking

The command tracks when it was last run in `~/.standup_last_run`:
- First run: defaults to "yesterday"
- Subsequent runs: shows work since last `/standup` execution
- Uses ISO 8601 UTC timestamps for precision

### Session File Analysis

Parses Claude Code session files (JSONL format) to extract:
- File edits (Edit/Write tool calls)
- Tasks created and completed
- Bash commands executed
- Work context not captured in git commits

## Customization

See [CUSTOMIZATION.md](CUSTOMIZATION.md) for details on:
- Modifying output format
- Adjusting git filters
- Customizing session parsing
- Jira integration settings

## Requirements

- Git configured with `user.email`
- Claude Code (obviously!)
- `jq` for JSON parsing (optional, graceful fallback)
- Session files at `~/.claude/projects/`

## Development

This command was built by a team of 3 specialist agents:
- **Agent 1** researched git data collection
- **Agent 2** researched session file parsing
- **Agent 3** researched standup formatting

See `.build/` directory for research documentation.

## License

MIT
