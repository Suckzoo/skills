---
name: review-current-session
description: Review the current session for lessons learned, conflicts, and enhancement opportunities. Creates a timestamped lessons-learned file. Use at session end (handoff or completion).
argument-hint: [optional-ticket-number]
---

# Review Current Session

Review the entire session and capture lessons learned, conflicts, and enhancement opportunities.

## When to Use

- After completing work on a ticket
- During orchestrator handoff (context limits)
- When `/do-that-shit` finishes (either completion or early stop)

## Step 1: Identify Ticket Number

Determine the ticket/branch identifier:

1. If argument provided, use `$ARGUMENTS` as the ticket number
2. Otherwise, check git branch: `git branch --show-current`
3. If still unclear, ask the user

## Step 2: Review the Session

Reflect on the current conversation and identify:

### 2.1 Conflicts Between Directions
- Did the user's instructions conflict with CLAUDE.md guidance?
- Did the user override any default patterns or conventions?
- Were there any "do it this way instead" moments?

### 2.2 Lessons About Code Patterns
- Discovered patterns in the codebase not documented in CLAUDE.md
- Gotchas or pitfalls encountered during implementation
- Useful utilities or helpers found that weren't obvious
- Test patterns or conventions specific to this codebase

### 2.3 Enhancement Opportunities
- Missing documentation that would help future work
- CLAUDE.md gaps or outdated information
- Tooling improvements (skills, hooks, etc.)
- Repetitive tasks that could be automated

### 2.4 Decisions Made
- Any architectural or design decisions during this session
- Tradeoffs considered and why one approach was chosen

## Step 3: Create Lessons Learned File

Generate the current Unix epoch timestamp:

```bash
date +%s
```

Create file at: `~/tmp/dora-plans/{ticket-number}/lessons-learned-{epoch}.md`

Expand to absolute path (e.g., `/Users/suckzoo/tmp/dora-plans/ves-965/lessons-learned-1738765432.md`)

### File Structure

```markdown
# Lessons Learned: {ticket-number}

**Session Date**: {human readable date}
**Work Summary**: {brief 1-2 sentence summary of what was done}

## Conflicts / Overrides

{List any conflicts between system guidance and user direction}
{If none, write "None identified."}

## Code Pattern Insights

{Patterns discovered, gotchas, useful utilities}
{If none, write "None identified."}

## Enhancement Opportunities

{Suggestions for docs, CLAUDE.md, tooling, automation}
{If none, write "None identified."}

## Key Decisions

{Decisions made during this session and rationale}
{If none, write "None identified."}

## Notes

{Any other observations worth capturing}
```

## Step 4: Report Summary

Briefly tell the user:
- File created at: `{path}`
- Key highlights (1-3 bullet points of most significant findings)

## Notes

- Be concise - this is a quick capture, not a detailed report
- Focus on actionable insights that help future sessions
- If nothing significant to capture, create a minimal file noting "Clean session, no significant lessons"
- Don't over-document obvious things
