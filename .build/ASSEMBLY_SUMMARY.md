# Standup Command Assembly Summary

## Agent Team Results

Three specialist agents worked in parallel to research and build the `/standup` slash command:

### ✅ Agent 1 — Git Researcher
**Output:** `.build/git-block.md`

**Research completed:**
- How Claude Code shell preprocessing works (the `!` prefix)
- Git commands for collecting standup data
- Timestamp tracking with `~/.standup_last_run`
- Fallback handling for first-time runs

**Key findings:**
- `!` prefix executes shell commands before AI processing
- Git log with `--after` flag tracks commits since last standup
- Graceful fallback to "yesterday" when no previous timestamp exists
- ISO 8601 UTC timestamp for precise tracking

### ✅ Agent 2 — Session File Researcher
**Output:** `.build/session-block.md`

**Research completed:**
- Claude Code session file format at `~/.claude/projects/`
- JSONL parsing with `jq`
- Extracting file edits, bash commands, and tasks
- Handling crashed/incomplete sessions

**Key findings:**
- Session files are JSONL with one JSON object per line
- Tool calls stored in `.data.message.message.content[]`
- Can extract Edit/Write operations, Bash commands, TaskCreate entries
- Requires `jq` for JSON parsing, gracefully handles errors

### ✅ Agent 3 — Format Researcher
**Output:** `.build/format-block.md`

**Research completed:**
- Standup best practices (Yesterday/Today/Blockers)
- Claude Code command output formatting
- User confirmation prompts
- Fallback handling for missing data

**Key findings:**
- Standard scrum format: Yesterday / Today / Blockers
- Keep under 200 words, scannable bullet points
- Aggregate commits into meaningful work items
- Interactive Jira posting confirmation

## Final Assembly

**Output:** `.claude/skills/standup/SKILL.md`

The final slash command combines all three research blocks into a complete Claude Code skill:

### YAML Frontmatter
```yaml
name: standup
description: Generate a daily standup summary with git commits, Claude Code session activity, and formatted output
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash(git *, find *, grep *, jq *, cat *, date *, echo *)
```

### Command Structure
1. **Shell Preprocessing** (using `!` prefix)
   - Git commands from Agent 1's research
   - Session file extraction from Agent 2's research

2. **Processing Instructions**
   - Parse git commits
   - Analyze session activity
   - Generate standup report

3. **Output Format**
   - Yesterday/Today/Blockers structure from Agent 3's research
   - Professional, scannable, under 200 words
   - Jira posting confirmation

### Integration Points
- ✅ Git data collection (Agent 1)
- ✅ Session file parsing (Agent 2)
- ✅ Output formatting (Agent 3)
- ✅ Shell preprocessing with `!` prefix
- ✅ Timestamp tracking
- ✅ Fallback handling
- ✅ User confirmations

## Usage

Invoke the command in Claude Code:
```
/standup
```

The command will:
1. Execute shell preprocessing to collect git + session data
2. Analyze and aggregate the data
3. Format output as Yesterday/Today/Blockers
4. Ask for Jira posting confirmation

## Requirements

- Git configured with user.email
- `jq` installed for JSON parsing (optional, graceful fallback if missing)
- Claude Code session files at `~/.claude/projects/`

## Files Created

```
.claude/skills/standup/
└── SKILL.md                    # Main slash command

.build/
├── git-block.md               # Agent 1 research
├── session-block.md           # Agent 2 research
├── format-block.md            # Agent 3 research
└── ASSEMBLY_SUMMARY.md        # This file
```

## Success Metrics

- ✅ All 3 agents completed successfully (with restarts after initial errors)
- ✅ Research documents written to `.build/`
- ✅ Final command assembled with proper YAML frontmatter
- ✅ Git + session data collection integrated
- ✅ Professional output formatting
- ✅ User confirmation prompts
- ✅ Fallback handling for edge cases

The `/standup` command is ready to use!
