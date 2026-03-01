# Git Data Collection Block

## Shell Preprocessing with `!` Prefix

The `!` prefix in Claude Code slash commands indicates **shell preprocessing**. Commands prefixed with `!` are executed in the shell before the main command runs, allowing you to:

- Collect dynamic data from the system
- Execute git commands to gather repository state
- Pipe outputs or chain commands with bash operators
- Store results that will be available to Claude's processing

This is essential for standup reports because it gathers real-time git activity without manual copy-pasting.

## Git Commands for Standup Data

```bash
# Get commits since last standup (defaults to "yesterday" if first run)
!git log --after=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday") --author=$(git config user.email) --oneline

# Show current changes stats
!git diff --stat HEAD

# List any stashed work
!git stash list

# Update timestamp for next run
!echo $(date -u +%Y-%m-%dT%H:%M:%SZ) > ~/.standup_last_run
```

## Command Breakdown

### 1. Commits Since Last Standup
```bash
!git log --after=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday") --author=$(git config user.email) --oneline
```

**Purpose**: Shows all commits you've made since the last standup run.

**How it works**:
- `cat ~/.standup_last_run 2>/dev/null` attempts to read the timestamp from the last run
- If the file doesn't exist (first run), stderr is suppressed and the fallback triggers
- `|| echo "yesterday"` provides a sensible default for first-time usage
- `--author=$(git config user.email)` filters to only your commits
- `--oneline` provides compact output perfect for standup summaries

### 2. Current Working Changes
```bash
!git diff --stat HEAD
```

**Purpose**: Shows uncommitted changes in your working directory.

**Why it matters**: Captures work-in-progress that hasn't been committed yet, giving a complete picture of what you're actively working on.

### 3. Stashed Work
```bash
!git stash list
```

**Purpose**: Lists any work you've temporarily stashed.

**Why it matters**: Prevents "lost" work from being forgotten. If you context-switched and stashed changes, this surfaces them in your standup.

### 4. Timestamp Tracking
```bash
!echo $(date -u +%Y-%m-%dT%H:%M:%SZ) > ~/.standup_last_run
```

**Purpose**: Records the current timestamp for the next standup run.

**How it works**:
- Generates an ISO 8601 UTC timestamp
- Writes it to `~/.standup_last_run`
- Next run will use this timestamp instead of "yesterday"
- Creates a precise boundary between standup periods

## Benefits

- **No manual tracking**: Git data is collected automatically
- **Accurate time windows**: Tracks exactly since your last standup, not just "yesterday"
- **Complete visibility**: Captures committed work, WIP, and stashed changes
- **First-run friendly**: Gracefully falls back to "yesterday" if no previous timestamp exists
