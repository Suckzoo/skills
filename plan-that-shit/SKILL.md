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

## Step 3: Conduct Thorough Interview

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

## Step 4: Save Outputs

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

## Step 5: Summary

After saving, summarize for user:
- Ticket number identified
- Key decisions made
- Files created with paths
- Suggested next steps
