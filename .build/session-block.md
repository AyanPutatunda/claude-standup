# Session Data Collection Block

## Overview

Claude Code session files are stored as JSONL (JSON Lines) files in `~/.claude/projects/`. Each line is a JSON object representing a progress event in the conversation. These files capture all tool calls made during a session, including file edits, bash commands, and task management.

## Session File Location

Session files follow this structure:
```
~/.claude/projects/
  -Users-<username>-<project-path>/
    <session-uuid>.jsonl          # Main session file
    <session-uuid>/
      subagents/
        agent-<id>.jsonl          # Subagent session files (if using agents)
```

## Session File Format

Each line in a `.jsonl` file is a JSON object with this structure:

```json
{
  "type": "progress",
  "timestamp": "2026-03-01T17:29:42.483Z",
  "sessionId": "47811228-450f-4096-b1a4-fd950f1c8ab2",
  "cwd": "/Users/username/project",
  "gitBranch": "main",
  "data": {
    "message": {
      "type": "assistant",
      "message": {
        "content": [
          {
            "type": "tool_use",
            "name": "Write",
            "input": {
              "file_path": "/path/to/file",
              "content": "..."
            }
          }
        ]
      }
    }
  }
}
```

## Extracting Work Activity

### Goal
Find what work was done since the last standup by parsing session files modified after `~/.standup_last_run`.

### Approach

#### 1. Find Recent Session Files

```bash
# Find all .jsonl files modified since last standup
LAST_RUN=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday")
find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$LAST_RUN"
```

#### 2. Extract Tool Calls

Use `jq` to parse JSONL and extract relevant tool calls:

**File Edits:**
```bash
grep -a '"name":"Edit"' <session-file> | \
  jq -r '.data.message.message.content[]? | select(.name == "Edit") | .input.file_path' | \
  sort -u
```

**File Writes:**
```bash
grep -a '"name":"Write"' <session-file> | \
  jq -r '.data.message.message.content[]? | select(.name == "Write") | .input.file_path' | \
  sort -u
```

**Bash Commands:**
```bash
grep -a '"name":"Bash"' <session-file> | \
  jq -r '.data.message.message.content[]? | select(.name == "Bash") | .input.command' | \
  head -20
```

**Task Creates:**
```bash
grep -a '"name":"TaskCreate"' <session-file> | \
  jq -r '.data.message.message.content[]? | select(.name == "TaskCreate") | "\(.input.subject): \(.input.description)"'
```

**Task Updates:**
```bash
grep -a '"name":"TaskUpdate"' <session-file> | \
  jq -r '.data.message.message.content[]? | select(.name == "TaskUpdate" and .input.status == "completed") | .input.taskId'
```

#### 3. Complete Extraction Script

```bash
#!/bin/bash
# Extract work activity from Claude Code sessions since last standup

LAST_RUN=$(cat ~/.standup_last_run 2>/dev/null || echo "yesterday")
SESSIONS=$(find ~/.claude/projects/ -name "*.jsonl" -type f -newermt "$LAST_RUN")

if [ -z "$SESSIONS" ]; then
  echo "No Claude Code sessions found since last standup"
  exit 0
fi

echo "=== Claude Code Activity ==="
echo

# Track modified files
FILES_MODIFIED=()
for session in $SESSIONS; do
  # Extract Write operations
  while IFS= read -r file; do
    [ -n "$file" ] && FILES_MODIFIED+=("$file")
  done < <(grep -a '"name":"Write"' "$session" 2>/dev/null | \
    jq -r '.data.message.message.content[]? | select(.name == "Write") | .input.file_path')

  # Extract Edit operations
  while IFS= read -r file; do
    [ -n "$file" ] && FILES_MODIFIED+=("$file")
  done < <(grep -a '"name":"Edit"' "$session" 2>/dev/null | \
    jq -r '.data.message.message.content[]? | select(.name == "Edit") | .input.file_path')
done

# Deduplicate and display
if [ ${#FILES_MODIFIED[@]} -gt 0 ]; then
  echo "Files modified:"
  printf '%s\n' "${FILES_MODIFIED[@]}" | sort -u | sed 's/^/  - /'
  echo
fi

# Extract tasks worked on
echo "Tasks worked on:"
for session in $SESSIONS; do
  grep -a '"name":"TaskCreate"' "$session" 2>/dev/null | \
    jq -r '.data.message.message.content[]? | select(.name == "TaskCreate") | "  - \(.input.subject)"'
done

# Extract notable bash commands (exclude common read-only commands)
echo
echo "Notable commands executed:"
for session in $SESSIONS; do
  grep -a '"name":"Bash"' "$session" 2>/dev/null | \
    jq -r '.data.message.message.content[]? | select(.name == "Bash") | .input.command' | \
    grep -v "^ls " | grep -v "^cat " | grep -v "^find " | \
    head -5 | sed 's/^/  - /'
done
```

### Edge Cases

**Crashed/Incomplete Sessions:**
- Session files are written incrementally, so partial sessions still contain valid JSONL
- Use `jq` with error suppression: `jq -r '...' 2>/dev/null`
- Always use `grep -a` to handle binary data that may appear in session files

**Missing Fields:**
- Use `jq` optional access: `.content[]?` instead of `.content[]`
- Provide defaults: `.input.file_path // "unknown"`

**Large Sessions:**
- Limit output with `head -N` or `tail -N`
- Deduplicate file lists with `sort -u`
- Summarize bash commands (exclude read-only operations like `ls`, `cat`, `find`)

**Subagent Sessions:**
- Include subagent files: `find ... -name "*.jsonl"` will recursively find them
- Subagent sessions are in `<session-uuid>/subagents/agent-<id>.jsonl`

### Summarization Strategy

**Group by Type:**
1. Files modified (Edit/Write operations) → "Modified X files"
2. Tasks created/completed → "Worked on: [task subjects]"
3. Notable commands → Only show destructive/meaningful operations (git, npm, docker)

**Deduplication:**
- Sort and unique file paths
- Combine similar bash commands (multiple `git commit` → "Made git commits")

**Filtering:**
- Skip exploratory commands (`ls`, `cat`, `grep`, `find`)
- Focus on write operations and significant state changes
- Limit to top 10-15 items to keep output scannable

### Output Format

Transform extracted data into standup-friendly bullets:

```
Yesterday:
  - Modified 3 files in auth module (session.py, login.py, middleware.py)
  - Implemented user authentication flow
  - Fixed bug in password reset logic
  - Ran tests and deployed to staging
```

This data complements git commits by showing work-in-progress, exploratory work, and context that may not be captured in commit messages.
