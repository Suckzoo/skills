---
name: plan-that-shit
description: Plan implementation work through a thorough interview process. Use when starting work on a ticket, planning a feature, or when the user wants to discuss and document requirements before coding.
argument-hint: [optional-ticket-number]
---

# Plan That Shit

Conduct a thorough interview to understand requirements and create implementation plans for a ticket or feature.

## Step 1: Identify Ticket Number

Determine the ticket/branch identifier:

1. If argument provided, use `$ARGUMENTS` as the ticket number
2. Otherwise, check git branch: `git branch --show-current`
3. Check directory name from `pwd`
4. If still unclear, ask the user

Extract identifiers like `VES-965`, `ves-965`, `DORA-123`, or branch names.

## Step 2: Get Initial Context

Ask the user to briefly explain what they want to implement. This is just the starting point for the interview.

## Step 3: Explore Codebase

Before interviewing, spawn an Explore agent to understand the relevant parts of the codebase.

**Exploration goals:**

1. **Find related code**: Search for existing implementations similar to what the user wants
2. **Identify patterns**: Look for established patterns in the codebase (e.g., how similar features are structured)
3. **Locate files to modify**: Identify which files will likely need changes
4. **Check CLAUDE.md**: Read relevant CLAUDE.md files for guidance on patterns and conventions
5. **Find examples to follow**: Locate similar features that can serve as templates

**Spawn Explore agent with prompt like:**
```
Explore the codebase to help plan: {user's feature description}

Find:
1. Similar existing implementations (search for related keywords/patterns)
2. Files that will likely need modification
3. Relevant CLAUDE.md guidance (root and service-specific)
4. Established patterns for this type of feature
5. Any gotchas or conventions specific to this area

Return a summary of findings to inform planning.
```

**Use findings to:**
- Inform interview questions (ask about discovered patterns)
- Pre-populate TODO.md with known file paths
- Reference similar implementations in SPEC.md
- Avoid suggesting approaches that conflict with existing patterns

## Step 4: Conduct Thorough Interview

Use `AskUserQuestion` to interview the user with 2-4 questions at a time. Ask non-obvious questions that dig deeper. Continue until all important aspects are covered.

**Areas to explore (adapt based on feature type):**

### Architecture & Data Flow
- Data sources (database, API, cache)?
- System layers/components involved?
- Existing patterns to follow?

### API Design (if applicable)
- New endpoint or modify existing?
- Request/response shape?
- Query/path parameters?
- Auth requirements?

### Caching & Performance
- Caching needed? Where? TTL?
- Sync vs async?
- Rate limiting?

### Edge Cases & Error Handling
- Dependency failure behavior?
- Missing/incomplete data handling?
- Timeout/retry logic?

### Business Logic
- Thresholds, rules, calculations?
- Frontend vs backend logic split?
- Feature flags?

### Data Model
- Schema changes?
- New fields/migrations?
- Entity relationships?

### Testing
- Test coverage level?
- Unit vs integration?

### Scope
- What's explicitly out of scope?
- External dependencies?

**Interview guidelines:**
- Ask non-obvious, probing questions
- Offer reasonable options in questions
- Group related questions (2-4 per turn)
- Continue until user confirms complete
- Probe short answers for more detail

## Step 5: Enter Plan Mode for Approval

After gathering all information, use `EnterPlanMode` to draft the implementation plan for user approval.

1. **Enter plan mode**: Use the `EnterPlanMode` tool
2. **Write the plan**: Create the plan file with:
   - Overview of what will be implemented
   - Architecture/approach summary
   - Phased TODO list with file paths
   - Key decisions made
3. **Exit plan mode**: Use `ExitPlanMode` to present the plan for user approval
4. **Handle feedback**: If user requests changes, revise the plan and re-submit

This ensures the user explicitly approves the implementation approach before files are saved.

## Step 6: Save Outputs

Create directory and save files to `~/tmp/dora-plans/{ticket-number}/`:

### SPEC.md
Technical specification:
- Overview/summary
- Architecture details
- API design
- Technical specs
- Implementation details
- Error handling
- Out of scope items

### TODO.md
Phased implementation checklist:
- Clear phases (Schema, Protocol, Service, Controller, etc.)
- Actionable checkbox items
- File paths for changes
- Commands to run
- Task dependencies

### DECISIONS.md
Key decisions made (if significant):
- Decision points and choices
- Rationale
- Alternatives considered

## Step 7: Summary

After saving, summarize for user:
- Ticket number identified
- Key decisions made
- Files created with paths
- Suggested next steps
