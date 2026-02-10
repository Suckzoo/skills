---
name: push-that-shit
description: Commit current progress, push to remote, and manage PR lifecycle. Reads TODO.md to determine progress, creates meaningful commit messages, and manages draft/ready PR state. Use at significant milestones during /do-that-shit work.
argument-hint: [optional: wip|done|phase-name]
---

# Push That Shit

Commit current work progress, push to remote, and manage PR lifecycle.

## Step 1: Identify Ticket Number

Determine the ticket/branch identifier:

1. If argument contains ticket info, extract it
2. Otherwise, check git branch: `git branch --show-current`
3. Check directory name from `pwd`
4. If still unclear, ask the user

Extract identifiers like `VES-965`, `ves-965`, `DORA-123`, or branch names.

## Step 2: Determine Commit Type (CRITICAL — drives the entire flow)

**The commit type determines which subsequent steps execute.** In particular, `done` triggers pre-push validation (Step 3). You MUST resolve the commit type before proceeding to any later step.

### If argument is explicit
Check `$ARGUMENTS` for explicit commit type:

| Argument | Type | Use Case |
|----------|------|----------|
| `wip` | WIP commit | Context limit handoff, partial work |
| `done` | Completion | All work finished, ready for review |

### If no argument provided, infer commit type

When no argument is provided, infer the commit type:

1. **First, check existing PR status**:
   ```bash
   gh pr view --json isDraft 2>/dev/null
   ```
   - If PR exists and `isDraft: false` (already ready for review) → commit type is **`done`**
   - If PR doesn't exist or is still draft → continue to step 2

2. **Check TODO.md** - Read `~/tmp/dora-plans/{ticket-number}/TODO.md`:
   - If ALL phases have all items marked `[x]` → infer **`done`**
   - If only minor/chore items remain unmarked → use `AskUserQuestion` to ask if this is `done` or `wip`
   - If significant work remains incomplete → infer **phase commit**

3. **If still unclear** - Use `AskUserQuestion` tool to ask the user:
   - "What type of commit is this?" with options: `done`, `wip`, `phase commit`

## Step 3: Pre-Push Validation (only for `done`) — MANDATORY

**Skip this step** unless the commit type is `done` (PR ready for review).

**IMPORTANT: This step MUST run whenever the commit type is `done`, even if there are no uncommitted changes.** The purpose is to validate the entire branch diff against the base branch, not just uncommitted work. A clean working tree does NOT mean the code is validated — lint, typecheck, and format checks must still run against the full set of changes introduced by this branch.

When the commit type is `done`, run validation commands to catch obvious errors.

### Detect Changed Projects

Check which projects have modified files compared to the base branch(**IMPORTANT: base branch is `develop`**):
```bash
git diff --name-only origin/develop...HEAD
# Also include uncommitted changes:
git diff --name-only
git diff --name-only --cached
```

Map changed files to projects:
- `packages/protocol/**` → protocol
- `apps/backoffice/**` → backoffice
- `apps/web/**` → web
- `apps/api/**` → api

### Sub-step 3a: Run Codegen FIRST (if protocol changed) — DO NOT SKIP

**CRITICAL: You MUST run this sub-step BEFORE any lint/format/typecheck commands.** If you skip codegen, lint will run against stale generated files and produce false results.

**Check**: Do the changed files include anything under `packages/protocol/**`?
- **YES** → Run codegen, then proceed to Sub-step 3b:
  ```bash
  moonx protocol:build --dependents
  ```
  This regenerates all dependent files (`*.gen.go`, `*.gen.ts`, `appUrls.gen.ts`). Lint/typecheck results are MEANINGLESS without this step when protocol files changed.
- **NO** → Skip directly to Sub-step 3b.

### Sub-step 3b: Run Validation Commands

For each detected project with changes, run format + lint commands. Format commands auto-fix issues; lint/typecheck catch errors.

| Project | Commands to Run |
|---------|-----------------|
| protocol | `moonx protocol:format` |
| backoffice | `moonx backoffice:format && moonx backoffice:lint && moonx backoffice:typecheck` |
| web | `moonx web:format && moonx web:lint && moonx web:typecheck` |
| api | `moonx api:lint` |

### Handle Validation Results

1. **If lint/typecheck fails**: Report the errors to the user and ask whether to:
   - Fix the issues and retry
   - Push anyway (not recommended)
   - Abort the push

2. **If all validations pass**: Proceed to the next step.

Note: Format changes will be included in the commit created in Step 5.

## Step 4: Gather Context

### Check Git Status
```bash
git status
git diff --stat
```

If there are no changes to commit, inform the user and skip Steps 5-6 (commit and push). **Do NOT skip Step 7 (PR management)** — the PR may still need description updates or status changes.

### Load Plan Files
Read from `~/tmp/dora-plans/{ticket-number}/`:
- **TODO.md** - To understand completed phases
- **SPEC.md** - For PR description context

Expand path to absolute (e.g., `/Users/suckzoo/tmp/dora-plans/ves-965/`).

### Analyze Progress
From TODO.md, identify:
- Completed phases (all items `[x]`)
- Current phase in progress
- Remaining work

## Step 5: Create Commit

### For Phase Completion (default)
Stage specific files related to the phase work:
```bash
git add <specific-files>
```

Create commit with descriptive message:
```bash
git commit -m "$(cat <<'EOF'
<type>: <brief description>

<optional: bullet points of what was done>
EOF
)"
```

**Commit types**: feat, fix, refactor, test, chore, docs

### For WIP Commit (argument: `wip`)
Stage all changes:
```bash
git add -A
```

Create WIP commit:
```bash
git commit -m "$(cat <<'EOF'
wip: {ticket-number} - {current state description}

Progress:
- Completed: {list completed phases}
- In progress: {current work}
- Remaining: {what's left}
EOF
)"
```

### For Completion Commit (argument: `done`)
Stage all changes (including any formatting fixes from Step 3):
```bash
git add -A
```

Create final commit:
```bash
git commit -m "$(cat <<'EOF'
feat: complete {ticket-number} - {feature description}

{Summary of implementation from SPEC.md}
EOF
)"
```

**IMPORTANT**: Do NOT include any co-author credit in commit messages.

## Step 6: Push to Remote

Push the branch:
```bash
git push -u origin HEAD
```

## Step 7: Manage PR

### Check PR Status
```bash
gh pr view --json state,isDraft,url,number 2>/dev/null
```

### If No PR Exists
Create a draft PR:
```bash
gh pr create --draft --title "{ticket-number}: {short description from SPEC}" --body "$(cat <<'EOF'
## Summary
Work in progress for {ticket-number}.

{Brief description from SPEC.md}

## Status
- [ ] Implementation in progress
- [ ] Tests
- [ ] Ready for review

---
_Draft PR - will be updated when ready for review_
EOF
)"
```

### If PR Exists and Commit Type is `done`
Mark PR ready for review:
```bash
gh pr ready
```

Update PR description with full details:
```bash
gh pr edit --body "$(cat <<'EOF'
## Summary
{Overview from SPEC.md}

## Changes
{Key changes made, derived from completed TODO phases}

## Testing
{Test approach / what tests were added}

## Checklist
- [x] Implementation complete
- [x] Tests added/updated
- [ ] Ready for review
EOF
)"
```

### If PR Exists and Commit Type is `wip`
Optionally update PR title to indicate WIP status (if not already):
```bash
gh pr edit --title "WIP: {ticket-number}: {description}"
```

## Step 8: Report Results

Summarize what was done:
- Commit hash and message summary
- Push status
- PR status (created/updated/marked ready)
- PR URL

## Examples

### Commit after completing Phase 1
```
/push-that-shit
```
→ Creates commit: `feat: add workspace lastStartedAt schema field`
→ Pushes to origin
→ Creates draft PR if needed

### WIP commit due to context handoff
```
/push-that-shit wip
```
→ Creates commit: `wip: VES-965 - schema and provider complete`
→ Pushes to origin
→ Updates PR if exists

### Final commit when all work done
```
/push-that-shit done
```
→ Runs format/lint/typecheck on changed projects
→ Creates commit: `feat: complete VES-965 GPU utilization metrics`
→ Pushes to origin
→ Marks PR ready for review
→ Updates PR description with full details

## Notes

- This skill does NOT execute implementation work - use `/do-that-shit` for that
- Always stage specific files when possible to avoid committing unrelated changes
- The TODO.md file is used to understand progress, not modified by this skill
- PR descriptions are derived from SPEC.md and TODO.md progress
