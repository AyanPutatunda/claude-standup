# Standup Output Format Specification

## Overview
The standup command generates a structured daily update following the standard scrum standup format: Yesterday / Today / Blockers.

## Output Structure

```
📅 STANDUP UPDATE — [Date]

YESTERDAY:
• Implemented user authentication (commits: abc123, def456)
• Fixed bug in payment processing
• Code review for PR #234

TODAY:
• Continue work on payment processing
• Deploy authentication changes
• Team sync at 2pm

BLOCKERS:
• None / [specific blocker if any]

---
Post to Jira? (y/n):
```

## Formatting Guidelines

### 1. Header
- Include date in format: `YYYY-MM-DD` or `Month DD, YYYY`
- Use emoji (📅) for visual consistency

### 2. Yesterday Section
- Aggregate git commits into meaningful work items
- Combine related commits under single bullet point
- Include commit hashes in parentheses for reference
- Format: `• [Action verb] [what was done] (commits: [hash1], [hash2])`
- Examples:
  - `• Implemented user authentication (commits: abc123, def456)`
  - `• Fixed performance issue in dashboard (commit: 789xyz)`

### 3. Today Section
- Generate forward-looking tasks based on:
  - Incomplete work from yesterday
  - Typical follow-up activities (testing, deployment, reviews)
  - Branch names if they indicate planned work
- Keep items actionable and specific
- Format: `• [Action verb] [what will be done]`

### 4. Blockers Section
- Default to `• None` if no blockers detected
- When present, be specific and actionable
- Format: `• [Description of blocker and impact]`

### 5. Jira Confirmation Prompt
- Always end with: `Post to Jira? (y/n):`
- Wait for user input before posting
- On 'y': Post to configured Jira project
- On 'n': Exit gracefully with message "Standup saved locally"

## Fallback Handling

When no git commits are found:
```
📅 STANDUP UPDATE — [Date]

YESTERDAY:
• No commits found — add updates manually

TODAY:
• [To be added]

BLOCKERS:
• None

---
Post to Jira? (y/n):
```

## Tone Guidelines

- **Brief**: Keep total output under 200 words
- **Factual**: Base on actual git activity
- **Action-oriented**: Use strong action verbs (Implemented, Fixed, Refactored, Deployed)
- **Scannable**: Use bullet points, consistent formatting
- **Professional**: Suitable for team/manager visibility

## Commit Aggregation Rules

1. Group commits by feature/task when possible
2. Ignore trivial commits (typos, formatting) unless only commits available
3. Max 5-6 bullet points in Yesterday section
4. Prefer clarity over completeness
5. Use commit messages to infer work context

## Copy-Paste Ready

- No special characters that break in Jira/Slack
- Consistent indentation
- Clear section breaks
- Remove emoji if posting to systems that don't support them
