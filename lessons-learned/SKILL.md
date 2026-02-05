---
name: lessons-learned
description: Analyze lessons-learned files for the current ticket and suggest enhancements to docs, CLAUDE.md, or codebase patterns.
argument-hint: [optional-ticket-number]
---

# Lessons Learned

Consolidate lessons-learned files for the current ticket into actionable enhancement suggestions.

## Step 1: Identify Ticket Number

Determine the ticket/branch identifier:

1. If argument provided, use `$ARGUMENTS` as the ticket number
2. Otherwise, check git branch: `git branch --show-current`
3. If still unclear, ask the user

## Step 2: Gather Lessons Learned Files

Find lessons-learned files for this ticket:

```bash
ls ~/tmp/dora-plans/{ticket-number}/lessons-learned-*.md
```

Read all found files.

If no files found, inform user and stop.

## Step 3: Categorize Findings

Group insights from all files into categories:

### 3.1 CLAUDE.md Updates
- Missing patterns or conventions
- Outdated information
- Gaps in documentation

### 3.2 Codebase Documentation
- Missing README sections
- Undocumented utilities or helpers
- API documentation gaps

### 3.3 Tooling / Automation
- Repetitive tasks that could be skills
- Hook opportunities
- Script improvements

### 3.4 Code Patterns
- Recurring gotchas to document
- Best practices discovered
- Anti-patterns to avoid

### 3.5 User Preference Conflicts
- Consistent overrides of system guidance
- Patterns where user prefers different approach

## Step 4: Explore Codebase for Context

Use the Explore agent or direct file reads to:

1. **Check current CLAUDE.md** files for overlap/gaps:
   - Root `CLAUDE.md`
   - Service-specific `apps/*/CLAUDE.md`

2. **Verify suggestions are still relevant**:
   - Has the issue already been fixed?
   - Is the pattern still used?

3. **Find related documentation**:
   - Existing READMEs
   - Inline documentation
   - Comments that could be promoted to docs

## Step 5: Generate Enhancement Report

Create a summary report. Present to user (don't write to file unless asked).

### Report Structure

```markdown
# Enhancement Suggestions

**Reviewed**: {N} lessons-learned files from {date range or ticket list}

## High Priority

{Most impactful suggestions - things that would save significant time}

### 1. {Enhancement title}
**Category**: CLAUDE.md / Docs / Tooling / Code Pattern
**Source**: lessons-learned from {ticket(s)}
**Current state**: {what exists now}
**Suggestion**: {specific change}
**Impact**: {why this matters}

## Medium Priority

{Useful but not urgent}

## Low Priority / Nice to Have

{Minor improvements}

## Already Addressed

{Items from lessons-learned that have been fixed since}

## Conflicts to Resolve

{Cases where user preferences consistently differ from documented guidance}
{Recommend: update docs to match actual practice, or discuss with user}
```

## Step 6: Offer Actions

Ask user what they'd like to do:

1. **Apply specific suggestions** - Make the changes now
2. **Create issues/tickets** - Track as future work
3. **Archive processed lessons** - Move lessons-learned files to `archived/` subdirectory
4. **Dismiss** - Just informational, no action needed

## Step 7: Cleanup (if requested)

If user chooses to archive:

```bash
mkdir -p ~/tmp/dora-plans/{ticket}/archived
mv ~/tmp/dora-plans/{ticket}/lessons-learned-*.md ~/tmp/dora-plans/{ticket}/archived/
```

## Notes

- Focus on patterns that appear multiple times - single occurrences may be one-offs
- CLAUDE.md improvements are highest value - they help all future sessions
- Be specific in suggestions - "improve docs" is too vague, "add section on X pattern in apps/api/CLAUDE.md" is actionable
- Don't suggest changes that conflict with user's established preferences
