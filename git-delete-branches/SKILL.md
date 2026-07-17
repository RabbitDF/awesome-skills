---
name: git-delete-branches
description: 'Delete confirmed obsolete git branches and log the action. Use after "git-identify-obsolete-branches" skill has generated a list of branches for removal. User must confirm the branch list before deletion. Prints structured logs to console.'
---

# Git Delete Branches

Deletes confirmed obsolete git branches and prints a structured log summary to console.

## When to Use This Skill

- User confirms the list of obsolete branches after running "git-identify-obsolete-branches"
- User explicitly asks to "delete branches" with a confirmed list
- Scheduled cleanup after user approval
- **IMPORTANT**: Branches must be explicitly confirmed by user before deletion

## Prerequisites

- Git repository initialized
- User has confirmed the list of branches to delete

## Safety Check

**MUST** verify user confirmation before proceeding:

1. Display the list of branches to be deleted
2. Ask user to explicitly confirm: "Do you want to proceed with deleting these [N] branches?"
3. Only proceed if user confirms with "yes", "confirm", or similar positive response
4. List includes branch names, reasons, and last activity dates

## Workflow

### 1. Verify current branch

```bash
git branch --show-current
```

**Critical**: Ensure you are NOT on a branch that will be deleted. Switch to main/master if needed:

```bash
git checkout main
```

### 2. Delete local branches

**Single branch:**
```bash
git branch -D <branch-name>
```

**Multiple branches (Bash):**
```bash
for branch in <branch1> <branch2> <branch3>; do
    git branch -D "$branch" 2>/dev/null && echo "Deleted: $branch"
done
```

**Multiple branches (PowerShell):**
```powershell
$branches = @("<branch1>", "<branch2>", "<branch3>")
foreach ($branch in $branches) {
    git branch -D $branch 2>$null
    if ($LASTEXITCODE -eq 0) { Write-Host "Deleted: $branch" }
}
```

### 3. Delete remote branches (if applicable)

```bash
git push origin --delete <branch-name>
```

**Ask user before deleting remote branches** - this affects all team members.

### 4. Print log to console

Output a structured summary:

```
=== BRANCH DELETION LOG ===
Timestamp: 2026-07-15T10:30:00Z
Repository: /path/to/repo
Initiator: user|scheduler

Deleted branches:
  - feature/old-thing (merged)     ✓
  - fix/old-bug (inactive 66 days) ✓
  - wip/experiment (merged)        ✓

Total deleted: 3
Status: success
```

### 5. Verify cleanup

```bash
git branch -vv
```

## Confirmation Template

```
=== BRANCH DELETION CONFIRMATION ===

The following branches will be deleted:

1. feature/old-thing    | Last commit: 2026-06-01 | Reason: Merged
2. fix/old-bug          | Last commit: 2026-05-10 | Reason: Inactive 66 days
3. wip/experiment       | Last commit: 2026-06-15 | Reason: Merged

Repository: <repo-path>
Action: Delete local branches only

Do you confirm deletion? (yes/no)
```

## Safety Notes

- **ALWAYS require user confirmation** before deleting any branches
- Local branches are deleted first; remote deletion requires extra confirmation
- Deleted local branches cannot be easily recovered
- Remote branch deletion affects all team members
- Never delete protected branches: main, master, develop, release/*, hotfix/*
