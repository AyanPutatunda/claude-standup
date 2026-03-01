---
name: standup
description: Generate a daily standup summary with git commits, Claude Code session activity, and formatted output. Use when preparing for team standups or status updates.
disable-model-invocation: false
user-invocable: true
argument-hint: [optional-date]
allowed-tools: Read, Glob, Grep, Bash(git *, find *, grep *, jq *, cat *, date *, echo *)
---

# Daily Standup Report Generator

Generate a comprehensive standup summary by collecting:
1. Git commits since last standup
2. Claude Code session activity (files edited, tasks completed)
3. Formatted output in Yesterday/Today/Blockers structure

## Shell Preprocessing Commands

The following commands execute BEFORE the AI processes results (using `!` prefix for shell preprocessing):

### Git Data Collection

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

### Claude Code Session Data Collection

Extract work activity from Claude Code session files:

```bash
# Find session files modified since last standup
!LAST_RUN=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday"); find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$LAST_RUN" 2>/dev/null || echo ""

# For each session file found, extract file modifications
!for session in $(find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$(cat ~/.standup_last_run 2>/dev/null || echo 'yesterday')" 2>/dev/null); do grep -a '"name":"Write"' "$session" 2>/dev/null | jq -r '.data.message.message.content[]? | select(.name == "Write") | .input.file_path' 2>/dev/null; grep -a '"name":"Edit"' "$session" 2>/dev/null | jq -r '.data.message.message.content[]? | select(.name == "Edit") | .input.file_path' 2>/dev/null; done | sort -u | head -20

# Extract tasks worked on
!for session in $(find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$(cat ~/.standup_last_run 2>/dev/null || echo 'yesterday')" 2>/dev/null); do grep -a '"name":"TaskCreate"' "$session" 2>/dev/null | jq -r '.data.message.message.content[]? | select(.name == "TaskCreate") | .input.subject' 2>/dev/null; done | head -10
```

## Processing Instructions

After collecting the data above, process it as follows:

### 1. Analyze Git Commits
- Parse the commit messages to understand work completed
- Group related commits by feature/task
- Extract key accomplishments
- Note any commit hashes for reference

### 2. Analyze Session Activity
- Review files modified through Claude Code
- Identify tasks created and completed
- Understand the context of the work
- Combine with git commits for complete picture

### 3. Generate Standup Report

Format the output using this structure:

```
📅 STANDUP UPDATE — [Today's Date]

YESTERDAY:
• [Aggregate commits into meaningful work items]
• [Include commit hashes: abc123, def456]
• [Add Claude Code session work if relevant]
• [Maximum 5-6 bullet points]

TODAY:
• [Generate forward-looking tasks based on yesterday's work]
• [Include planned deployments, reviews, or follow-ups]
• [Keep items actionable and specific]

BLOCKERS:
• None [or list specific blockers if any detected]

---
Post to Jira? (y/n):
```

### Formatting Rules

**Yesterday Section:**
- Use action verbs: Implemented, Fixed, Refactored, Deployed, Reviewed
- Format: `• [Action] [what was done] (commits: [hash1], [hash2])`
- Group related commits under single bullet
- Include both git commits AND session work
- Example: `• Implemented user authentication (commits: abc123, def456)`

**Today Section:**
- Infer from incomplete work or branch names
- Include typical follow-ups: testing, deployment, code review
- Format: `• [Action verb] [what will be done]`
- Example: `• Deploy authentication changes to staging`

**Blockers Section:**
- Default to `• None` if nothing detected
- Be specific if blockers exist
- Format: `• [Description of blocker and impact]`

**Tone Guidelines:**
- **Brief**: Keep under 200 words total
- **Factual**: Base on actual activity
- **Action-oriented**: Strong action verbs
- **Scannable**: Bullet points, clear sections
- **Professional**: Suitable for team visibility

### Fallback Handling

If NO git commits AND NO session activity found:

```
📅 STANDUP UPDATE — [Today's Date]

YESTERDAY:
• No commits found — add updates manually

TODAY:
• [To be added]

BLOCKERS:
• None

---
Post to Jira? (y/n):
```

### Jira Integration

After displaying the standup report:
1. Ask: `Post to Jira? (y/n):`
2. Wait for user input
3. On 'y': Confirm they want to post and where (project key)
4. On 'n': Display "Standup saved locally - you can copy and paste manually"

### Edge Cases

**No ~/.standup_last_run file:**
- Git will default to "yesterday"
- Session search will use "yesterday"
- First run is gracefully handled

**No git commits:**
- Show session activity if available
- Use fallback format if no activity at all
- Prompt user to add updates manually

**Large output:**
- Limit to top 5-6 items per section
- Summarize rather than list everything
- Keep total output scannable

**Session file parsing errors:**
- Suppress jq errors with `2>/dev/null`
- Continue even if some sessions can't be parsed
- Focus on successfully parsed data

## Example Output

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

## Notes

- The `!` prefix executes shell commands before AI processing
- `~/.standup_last_run` tracks the last execution timestamp
- Session files are in `~/.claude/projects/` as JSONL format
- Uses `jq` for JSON parsing (requires jq to be installed)
- Gracefully handles missing files and first-time runs
- Combines git commits + Claude Code session work for complete picture
