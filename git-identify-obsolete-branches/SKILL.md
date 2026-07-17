---
name: git-identify-obsolete-branches
description: 'Identify obsolete git branches that can be safely removed. Use when asked to "identify obsolete branches", "find stale branches", "list deletable branches", or when running scheduled branch cleanup. Scans branches merged into main or inactive for more than 30 days.'
---

# Git Identify Obsolete Branches

Scans the git repository to identify branches that can be safely removed based on merge status and inactivity period.

## When to Use This Skill

- User asks to "identify obsolete branches"
- User wants to "find stale branches" or "list deletable branches"
- Scheduled daily/weekly branch cleanup
- After a PR is merged and branches need review
- User wants to review branches before deletion

## Prerequisites

- Git repository initialized
- Network access to fetch latest remote information
- `main` or `master` branch as the base branch

## Workflow

### 1. Fetch latest remote information

```bash
git fetch --all --prune
```

### 2. Get the base branch name (main or master)

```bash
git branch --show-current
```

### 3. Identify branches merged into main/master

```bash
git branch --merged main
```

### 4. Identify branches merged into main/master (alternative)

```bash
git branch --merged master
```

### 5. Find branches inactive for 30+ days

**Bash:**
```bash
for branch in $(git for-each-ref --format='%(refname:short)' refs/heads/); do
  last_commit=$(git log -1 --format='%ai' "$branch" 2>/dev/null)
  if [ -n "$last_commit" ]; then
    days_since=$(($(date +%s) - $(date -d "$last_commit" +%s) / 86400))
    if [ "$days_since" -gt 30 ]; then
      echo "$branch|$last_commit|$days_since"
    fi
  fi
done
```

**PowerShell:**
```powershell
git for-each-ref --format='%(refname:short)' refs/heads/ | ForEach-Object {
    $branch = $_
    $lastCommit = git log -1 --format='%ai' $branch 2>$null
    if ($lastCommit) {
        $commitDate = [DateTime]::Parse($lastCommit.Trim())
        $daysSince = ((Get-Date) - $commitDate).Days
        if ($daysSince -gt 30) {
            "$branch|$lastCommit|$daysSince"
        }
    }
}
```

### 6. Combine merged and inactive branches

Filter the output to show branches that are:
- Merged into main/master, OR
- Inactive for more than 30 days

**Exclude protected branches** (main, master, develop, release/*, hotfix/*)

### 7. Generate report

Create a structured report with:
- Branch name
- Last commit date
- Days since last activity
- Reason (merged/inactive)

## Output Format

```
=== OBSOLETE BRANCHES REPORT ===
Generated: [timestamp]

Branch Name          | Last Commit   | Days Inactive | Reason
---------------------|---------------|---------------|------------------
feature/old-thing    | 2026-06-01    | 44            | Merged
wip/experiment       | 2026-05-10    | 66            | Inactive > 30 days
fix/old-bug          | 2026-06-15    | 30            | Merged

Total branches identified: 3
```

## Safety Notes

- This skill ONLY identifies branches - it does NOT delete anything
- Always exclude protected branches from deletion candidates
- Present the report to the user for confirmation before deletion
- Branches that have never been pushed to remote are marked as "local-only"
