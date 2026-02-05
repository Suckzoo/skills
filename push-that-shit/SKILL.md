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

## Step 2: Determine Commit Type

Check `$ARGUMENTS` to determine the commit type:

| Argument | Type | Use Case |
|----------|------|----------|
| `wip` | WIP commit | Context limit handoff, partial work |
| `done` | Completion | All work finished, ready for review |
| (none or phase name) | Phase commit | Normal phase completion |

## Step 3: Gather Context

### Check Git Status
```bash
git status
git diff --stat
```

If there are no changes to commit, inform the user and skip to PR management.

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

## Step 4: Create Commit

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
Stage all changes:
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

## Step 5: Push to Remote

Push the branch:
```bash
git push -u origin HEAD
```

## Step 6: Manage PR

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

## Step 7: Report Results

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
→ Creates commit: `feat: complete VES-965 GPU utilization metrics`
→ Pushes to origin
→ Marks PR ready for review
→ Updates PR description with full details

## Notes

- This skill does NOT execute implementation work - use `/do-that-shit` for that
- Always stage specific files when possible to avoid committing unrelated changes
- The TODO.md file is used to understand progress, not modified by this skill
- PR descriptions are derived from SPEC.md and TODO.md progress
