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

Read all plan and progress files:
- **SPEC.md** - Technical specification and requirements
- **TODO.md** - Implementation checklist with phases
- **DECISIONS.md** - Key decisions and rationale (if exists)
- **HANDOFF.md** - Brief summary of the latest agent's work.

If files don't exist, inform user and suggest running `/plan-that-shit` first.

**Save the absolute directory path** - you will pass it to the agent in Step 4.

## Step 3: Assess Current State

Review TODO.md to determine:
- Which items are already checked off `[x]`
- Any WIP notes indicating partial progress
- The next uncompleted phase/item to work on

Summarize briefly: "Resuming from Phase X, item Y" or "Starting fresh from Phase 1"

## Step 4: Spawn Execution Agent

Use the Task tool to spawn an agent with `max_turns: 50` and the following prompt structure.

Do NOT EVER USE Sonnet nor Haiku. Prefer Opus usage.

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
| HANDOFF.md | {plan_files_dir}/HANDOFF.md | Write the summary of your work when you terminate - A MUST-DO ITEM |

**FIRST ACTION**: Before doing ANY implementation work, use the Read tool to verify you can access `{plan_files_dir}/TODO.md`. If you cannot read this file, STOP and report the error.

---

## Specifications
{contents of SPEC.md}

## Current TODO List
{contents of TODO.md}

## Decisions Made So Far
{contents of DECISIONS.md, if exists, otherwise "No decisions recorded yet."}

## Previous Agent's Handoff Message
{contents of HANDOFF.md, if exists, otherwise "No handoff message exists."}

---

## Your Instructions

1. **IMPORTANT: You CANNOT use AskUserQuestion**. If you need clarification or a decision, you must:
   - Save your current progress to `{plan_files_dir}/TODO.md`
   - Create or update `{plan_files_dir}/BLOCKED.md` with:
     - What you were trying to do
     - The specific question or clarification needed
     - Any context the orchestrator needs to help
   - **MANDATORY** End your job. You MUST terminate your job.
   - The orchestrator will handle the question and spawn a new agent with the answer

2. **MANDATORY: Update TODO.md after EVERY phase**:
   - BEFORE starting work on a new phase, update `{plan_files_dir}/TODO.md` to mark the previous phase's items as complete
   - This is NOT optional. Do NOT proceed to the next phase without updating the file first
   - Mark completed items: `[x]`
   - Add WIP notes for partial work: `<!-- WIP: description of partial state -->`
   - Example workflow:
     - Complete Phase 1 work
     - Edit `{plan_files_dir}/TODO.md` to mark Phase 1 items as `[x]`
     - THEN start Phase 2

3. **Completion**: Whenever you finish a phase:
   - Update `{plan_files_dir}/TODO.md` to mark ALL items as `[x]`
   - **MANDATORY** End your job. You MUST terminate your job.

4. **Turn counting (max 50 turns)**:
   - You have a hard limit of 50 turns before being stopped automatically
   - **Count your turns**: Start at turn 1. Each time you respond with tool calls, increment your count. Track this mentally (e.g., "Turn 12: implementing X...")
   - **From turn 40 onwards**: Start wrapping up and save all progress:
     - **MANDATORY** End your job. You MUST terminate your job.

5. **MANDATORY: When the user rejects your tool call**:
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
     - **MANDATORY** End your job. You MUST terminate your job.

6. **CRITICALLY MANDATORY: Whenever you terminate your job, before you terminate**:
   - Update `{plan_files_dir}/TODO.md` with current progress and WIP notes
   - Update `{plan_files_dir}/DECISIONS.md` with any new decisions made
   - If blocked, update `{plan_files_dir}/BLOCKED.md`
   - Write a handoff summary to `${plan_files_dir}/HANDOFF.md`, So the orchestrator can spawn a fresh agent to continue
   - After these, terminate yourself. Die.

---

**REMEMBER**: The TODO.md file at `{plan_files_dir}/TODO.md` is your checkpoint system. If you don't update it, progress cannot be tracked or resumed. Treat updates as MANDATORY, not optional.

**REMEMBER 2**: Rethink before you act: every turn, check if you must terminate or not. And DO NOT FORGET the INSTRUCTION 6.

---

Start from: {current state assessment}
```

## Step 5: Handle Agent Return

**IMPORTANT**: As the orchestrator, minimize your own token usage. Do NOT manually verify work (running tests, checking git status, reading files to confirm changes). Trust the agent's report or spawn another agent to verify if needed.

When the agent finishes, evaluate the result based on its report (which is obviously HANDOFF.md):

### First, push current changes with /push-that-shit:
1. You MUST USE the /push-that-shit skill.

### **IMPORTANT: check if blocked and needs clarification**:
1. Read `{plan_files_dir}/BLOCKED.md` if the agent indicates it was blocked
2. Present the question/issue to the user using AskUserQuestion
3. Once user responds:
   - Record the decision in `{plan_files_dir}/DECISIONS.md`
   - Delete or clear `{plan_files_dir}/BLOCKED.md`
   - Go back to Step 3 (assess state and spawn new agent with the clarification)

### Then, update progress in the corresponding linear ticket:
1. By reading `${plan_files_dir}/HANDOFF.md`, `${plan_files_dir}/TODO.md` update progress by making a comment on the ticket.
2. If there's a decision made in the previous step(which is resolving BLOCKED.md), note the issue/question and its resolution/decision.

### If every TODO item is completed:
1. Report the agent's summary to the user
2. Suggest next steps (tests, PR, `/push-that-shit done`, etc.)

### If there's something not done:
1. Summarize what the agent reported as accomplished
2. Ask user: "Agent completed X items and saved progress. Continue with a new agent?"
3. If yes, go back to Step 3 (assess state and spawn new agent)


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
