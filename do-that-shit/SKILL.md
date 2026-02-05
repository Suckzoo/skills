---
name: do-that-shit
description: Execute implementation work following a pre-made plan. Reads specs, decisions, and todos from ~/tmp/dora-plans/, then spawns agents to implement. Use after plan-that-shit has created the plan files.
argument-hint: [optional-ticket-number]
---

# Do That Shit

Execute a pre-planned implementation by spawning agents that work through the TODO list.

## Step 1: Identify Ticket Number

Determine the ticket/branch identifier:

1. If argument provided, use `$ARGUMENTS` as the ticket number
2. Otherwise, check git branch: `git branch --show-current`
3. Check directory name from `pwd`
4. If still unclear, ask the user

Extract identifiers like `VES-965`, `ves-965`, `DORA-123`, or branch names.

## Step 2: Load Plan Files

Determine the plan files directory path: `~/tmp/dora-plans/{ticket-number}/`

**Expand to absolute path** (e.g., `/Users/suckzoo/tmp/dora-plans/ves-965/`) - you will need this for the agent prompt.

Read all plan files:
- **SPEC.md** - Technical specification and requirements
- **TODO.md** - Implementation checklist with phases
- **DECISIONS.md** - Key decisions and rationale (if exists)

If files don't exist, inform user and suggest running `/plan-that-shit` first.

**Save the absolute directory path** - you will pass it to the agent in Step 4.

## Step 3: Assess Current State

Review TODO.md to determine:
- Which items are already checked off `[x]`
- Any WIP notes indicating partial progress
- The next uncompleted phase/item to work on

Summarize briefly: "Resuming from Phase X, item Y" or "Starting fresh from Phase 1"

## Step 4: Spawn Execution Agent

Use the Task tool to spawn an agent with the following prompt structure.

**IMPORTANT**: Replace `{plan_files_dir}` with the actual absolute path (e.g., `/Users/suckzoo/tmp/dora-plans/ves-965`).

```
You are implementing ticket {ticket-number}. Work through the TODO list autonomously.

## CRITICAL: Plan File Locations

You MUST update these files to track your progress. Use these EXACT paths:

| File | Path | Purpose |
|------|------|---------|
| TODO.md | {plan_files_dir}/TODO.md | Track completed/pending items - UPDATE THIS |
| SPEC.md | {plan_files_dir}/SPEC.md | Reference only (read-only) |
| DECISIONS.md | {plan_files_dir}/DECISIONS.md | Record decisions - CREATE if missing |
| BLOCKED.md | {plan_files_dir}/BLOCKED.md | Write here when blocked and need clarification |

**FIRST ACTION**: Before doing ANY implementation work, use the Read tool to verify you can access `{plan_files_dir}/TODO.md`. If you cannot read this file, STOP and report the error.

---

## Specifications
{contents of SPEC.md}

## Current TODO List
{contents of TODO.md}

## Decisions Made So Far
{contents of DECISIONS.md, if exists, otherwise "No decisions recorded yet."}

---

## Your Instructions

1. **Work autonomously**: Proceed through TODO items without pausing for approval. Only stop when you hit genuine ambiguity or a tool rejection you cannot resolve.

2. **IMPORTANT: You CANNOT use AskUserQuestion**. If you need clarification or a decision, you must:
   - Save your current progress to `{plan_files_dir}/TODO.md`
   - Create or update `{plan_files_dir}/BLOCKED.md` with:
     - What you were trying to do
     - The specific question or clarification needed
     - Any context the orchestrator needs to help
   - Return immediately with a summary for the orchestrator
   - The orchestrator will handle the question and spawn a new agent with the answer

3. **MANDATORY: Update TODO.md after EVERY phase**:
   - BEFORE starting work on a new phase, update `{plan_files_dir}/TODO.md` to mark the previous phase's items as complete
   - This is NOT optional. Do NOT proceed to the next phase without updating the file first
   - Mark completed items: `[x]`
   - Add WIP notes for partial work: `<!-- WIP: description of partial state -->`
   - Example workflow:
     - Complete Phase 1 work
     - Edit `{plan_files_dir}/TODO.md` to mark Phase 1 items as `[x]`
     - THEN start Phase 2

4. **MANDATORY: Checkpoint every 30-40 tool calls**:
   - Count your tool usage. After approximately 30-40 tool calls, STOP and:
     - Update `{plan_files_dir}/TODO.md` with current progress
     - Update `{plan_files_dir}/DECISIONS.md` with any new decisions made
   - Then continue working
   - This ensures progress is never lost even if you hit context limits unexpectedly

5. **MANDATORY: Context limit check (15% remaining = STOP)**:
   - If you have less than 15% context remaining, you MUST STOP IMMEDIATELY
   - Do NOT attempt to squeeze in more work
   - Before stopping:
     - Update `{plan_files_dir}/TODO.md` with current progress and any WIP notes
     - Update `{plan_files_dir}/DECISIONS.md` with any new decisions
     - Search Linear for the ticket (e.g., VES-965) using list_issues
     - If found, add a comment to the Linear ticket with your progress:
       - Which task/phase you started from
       - Summary of what you accomplished
       - What remains to be done
     - Return a handoff summary to the orchestrator

6. **Completion**: When all TODO items are done:
   - Update `{plan_files_dir}/TODO.md` to mark ALL items as `[x]`
   - If Linear ticket exists, add a completion comment with summary of work
   - Return a summary of work completed to the orchestrator
   - (The orchestrator will handle git commit/push/PR via `/push-that-shit done`)

7. **When the user rejects your tool call**:
   - If the fix is obvious from their rejection message, apply the fix and retry
   - If the rejection reason is UNCLEAR or you don't know how to proceed:
     - Do NOT guess or retry blindly
     - Save your progress to `{plan_files_dir}/TODO.md` with WIP notes
     - Create/update `{plan_files_dir}/BLOCKED.md` with:
       ```markdown
       ## Blocked: Tool Rejection
       **Tool**: {which tool was rejected}
       **Action attempted**: {what you were trying to do}
       **Rejection message**: {the rejection message}
       **Question for user**: {what you need clarified to proceed}
       **Context**: {any relevant context}
       ```
     - Return immediately to the orchestrator with this information

---

**REMEMBER**: The TODO.md file at `{plan_files_dir}/TODO.md` is your checkpoint system. If you don't update it, progress cannot be tracked or resumed. Treat updates as MANDATORY, not optional.

---

Start from: {current state assessment}
```

## Step 5: Handle Agent Return

**IMPORTANT**: As the orchestrator, minimize your own token usage. Do NOT manually verify work (running tests, checking git status, reading files to confirm changes). Trust the agent's report or spawn another agent to verify if needed.

When the agent returns, evaluate the result based on its report:

### If handoff due to context limits:
1. Summarize what the agent reported as accomplished
2. Ask user: "Agent completed X items and saved progress. Continue with a new agent?"
3. If yes, go back to Step 3 (assess state and spawn new agent)

### If blocked and needs clarification:
1. Read `{plan_files_dir}/BLOCKED.md` if the agent indicates it was blocked
2. Present the question/issue to the user using AskUserQuestion
3. Once user responds:
   - Record the decision in `{plan_files_dir}/DECISIONS.md`
   - Delete or clear `{plan_files_dir}/BLOCKED.md`
   - Go back to Step 3 (assess state and spawn new agent with the clarification)

### If completed:
1. Report the agent's summary to the user
2. Suggest next steps (tests, PR, `/push-that-shit done`, etc.)

## Step 6: Self Context Check

If YOUR (orchestrator's) context is nearing limits:
1. Ensure all plan files are up to date
2. Inform user: "My context is getting full. Plan files are saved. You can start a new conversation and run `/do-that-shit {ticket}` to continue."

## File Update Conventions

### TODO.md Updates
```markdown
## Phase 1: Schema Changes
- [x] Add new field to User model
- [x] Create migration
- [ ] Update indexes <!-- WIP: index created, needs testing -->

## Phase 2: API Layer
- [ ] Add endpoint handler
```

### DECISIONS.md Updates
```markdown
## During Implementation

### Caching Strategy (2024-01-15)
**Context**: Needed to decide TTL for user preferences cache
**Decision**: 5 minute TTL with stale-while-revalidate
**Rationale**: Balances freshness with reduced DB load
```
