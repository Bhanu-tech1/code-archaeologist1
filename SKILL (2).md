---
name: excavate-history
description: "Analyzes git commit history to surface age, churn, authorship patterns, and buried context from commit messages"
allowed-tools: Bash Read
metadata:
  author: code-archaeologist
  version: "1.0.0"
  category: analysis
---

# Excavate History

Extracts the narrative hidden in git history: which files change most, who
knows which parts of the system, and what the commit messages reveal about
technical debt and past decisions.

## When to Use

Use this skill when the user wants to understand:
- How old different parts of the codebase are
- Which files have the most churn (frequent change = instability or importance)
- What the git log reveals about past architectural decisions
- Which contributors have context on specific modules

## Instructions

### Step 1: Gather Commit Statistics

Run these commands against the target repository directory:

```bash
# Overall repository age and commit count
git log --oneline | wc -l
git log --format="%ai" | tail -1  # First commit date
git log --format="%ai" | head -1  # Most recent commit date

# Top 20 most-changed files (churn analysis)
git log --name-only --format="" | sort | uniq -c | sort -rn | head -20

# Commit frequency by month (trend analysis)
git log --format="%ai" | cut -c1-7 | sort | uniq -c

# Files not touched in over 2 years
git log --format="%ai %H" -- <file> | head -1
```

### Step 2: Classify Files by Age Tier

Assign each significant file to an age tier based on last modification:
- **Fossil** (>5 years since last commit): Highest archaeological interest
- **Ancient** (2–5 years): Likely stable or forgotten
- **Mature** (6 months–2 years): Active maintenance zone
- **Recent** (<6 months): Current development

### Step 3: Analyze Commit Messages for Debt Signals

Search commit history for known debt indicators:

```bash
# Debt signal keywords in commit messages
git log --oneline --grep="TODO\|FIXME\|hack\|workaround\|temporary\|TEMP\|quick fix\|hotfix" --ignore-case
```

Classify each match:
- **Explicit debt**: Message contains "TODO", "FIXME", "hack", "workaround"
- **Crisis commits**: Messages with "hotfix", "emergency", "CRITICAL", "prod down"
- **Revert storms**: Multiple reverts on same file/feature within short window

### Step 4: Identify Knowledge Silos

```bash
# Contributors per file (last 2 years)
git log --since="2 years ago" --format="%ae" -- <file> | sort -u | wc -l
```

Flag files touched by only one contributor in the last 2 years as
**knowledge silos** — single points of failure for institutional memory.

### Step 5: Surface Buried Context

For CRITICAL and HIGH severity files, extract the 5 most recent commit
messages and include them verbatim in the findings. Commit messages often
contain the *only* written explanation for a non-obvious implementation.

## Output Format

```
## History Excavation Report

**Repository Age:** X years (first commit: YYYY-MM-DD)
**Total Commits:** N
**Active Contributors (last 12 months):** N

### Churn Leaders (top files by change frequency)
| File | Changes | Last Modified | Age Tier | Sole Author? |
|------|---------|---------------|----------|--------------|
| ... | ... | ... | ... | ... |

### Debt Signal Commits
| Commit | Date | Message | Severity |
|--------|------|---------|----------|
| abc1234 | ... | "quick fix for prod crash" | HIGH |

### Knowledge Silos
Files with only one contributor in the past 2 years: [list]

### Fossil Files
Files not modified in 5+ years still present in active codebase: [list]
```
