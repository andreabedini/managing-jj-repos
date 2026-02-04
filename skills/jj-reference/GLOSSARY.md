# Jujutsu Glossary

Definitions of key terms and concepts in jj.

## Core Concepts

### Change

A logical unit of work that persists through rewrites. Each change has a stable **change ID** that doesn't change even when the commit is amended, rebased, or otherwise rewritten.

**Key points:**
- Changes are what you work with conceptually
- Change IDs are 12-letter identifiers (e.g., `mzvwutvlkqwt`)
- Same change can have multiple commit IDs over time
- Use change IDs when referring to commits you might rewrite

**Example:** When you amend a commit, the change ID stays the same but the commit ID changes.

---

### Change ID

A stable 12-letter identifier for a change. Persists through rewrites like amend, rebase, squash, etc.

**Format:** lowercase letters (e.g., `mzvwutvlkqwt`)

**Contrast with commit ID:** Commit IDs (Git hashes) change on every rewrite. Change IDs stay the same.

**Usage:** Preferred for referring to commits in commands like `jj rebase -r <change-id>`.

---

### Commit

A specific snapshot of repository state. Has both a change ID (stable) and a commit ID (Git hash, changes on rewrite).

**Key points:**
- Represents exact file states at a point in time
- Has parents, author, timestamp, description
- Immutable once created (edits create new commits)

**In jj:** Commits are created automatically from working copy changes when you run commands like `jj describe` or `jj new`.

---

### Commit ID

The Git SHA-1 hash identifying a specific commit snapshot. Changes every time the commit is rewritten.

**Format:** 40-character hex string (e.g., `e5251d63...`)

**When to use:** Rarely needed. Use change IDs instead for most operations.

---

### Working Copy

The current state of files in your working directory, represented as a commit with the symbol `@`.

**Key points:**
- No staging area - working directory IS a commit
- Changes automatically tracked when you run jj commands
- Symbol: `@` in log output and revsets
- Called "working-copy commit"

**Mental model:** You're always working inside a commit, not preparing changes to commit later.

---

### Working-Copy Commit

The commit representing your working directory state. Referred to as `@`.

**Characteristics:**
- Usually has an empty description until you run `jj describe`
- Automatically updated as you modify files
- Can be finalized with `jj describe` or `jj commit`
- New working-copy commit created with `jj new`

**Pattern:**
```bash
# Make changes (@ now contains changes)
jj describe -m "description"  # Describe @
jj new                        # Create new empty @
```

---

### Bookmark

A named pointer to a commit, similar to Git branches but with important differences.

**Key differences from Git branches:**
- Bookmarks **don't automatically advance** with commits
- Bookmarks are **optional** - you can work without them
- Must be moved manually with `jj bookmark move`

**When to use:**
- Pushing to remotes (Git needs branch names)
- Marking important commits
- Coordinating with teammates

**Symbol in log:** Shown in parentheses like `(main)`

---

### Head

A commit with no descendants. Represents the tip of a line of development.

**Key points:**
- Multiple heads can exist simultaneously
- jj handles multiple heads gracefully
- Not a special state (unlike Git HEAD)

**View heads:**
```bash
jj log -r 'heads(all())'
```

---

### Operation

An atomic change to repository state. Everything you do creates an operation, logged in the **operation log**.

**Examples of operations:**
- Running `jj commit`
- Running `jj rebase`
- Running `jj bookmark move`

**Key points:**
- Operations are immutable
- Full history of operations is kept
- Use `jj op log` to view
- Use `jj undo` to reverse operations
- Much more comprehensive than Git's reflog

---

### Operation Log

Complete history of all operations performed in the repository. Your safety net for mistakes.

**View with:** `jj op log`

**Uses:**
- Undo mistakes with `jj undo`
- Restore to specific point with `jj op restore`
- Investigate what happened with `jj op show`
- View past states with `jj --at-op=<id> log`

**Key benefit:** Almost impossible to lose work. Everything is tracked.

---

### Divergent Change

When multiple visible commits share the same change ID.

**Causes:**
- Concurrent modifications in multiple workspaces
- Complex rebase scenarios
- Race conditions in operations

**Indicator:** Log shows `(divergent)` annotation

**Usually harmless:** Often resolves itself or can be fixed with `jj squash` or `jj abandon`.

---

### Revset

jj's query language for selecting commits. Much more powerful than Git revision syntax.

**Examples:**
- `@` - Working copy
- `::@` - Ancestors of working copy
- `main..@` - Commits in @ but not main
- `description(bug)` - Commits mentioning "bug"
- `author(alice)` - Commits by Alice

**Used in:** Most commands that accept revisions (`-r`, `-s`, `-d` flags)

**See:** [REFERENCE.md](REFERENCE.md) for complete syntax

---

### Colocated Repository

A repository where jj and Git share the working directory. Both `.jj` and `.git` directories exist.

**Created with:** `--colocate` flag

**Benefits:**
- Can use Git tools that read `.git` directly
- Gradual transition from Git to jj
- Git commands work (but avoid mixing)

**Cautions:**
- Don't interleave git and jj commands
- Run `jj git import` if you use git commands
- Can cause confusion

---

### Remote Bookmark

A bookmark from a remote repository, shown as `<name>@<remote>`.

**Examples:**
- `main@origin`
- `feature@upstream`

**Key points:**
- Updated by `jj git fetch`
- Reference point for local bookmarks
- Can diverge from local bookmarks (creates conflict)

---

### Tracked Bookmark

A local bookmark that follows a remote bookmark.

**Create with:** `jj bookmark track <name>@<remote>`

**Behavior:** Stays in sync with remote bookmark when fetching.

---

### Conflicted Bookmark

A bookmark that points to multiple commits simultaneously. Marked with `??` in output.

**Causes:**
- Local and remote bookmarks diverged
- Concurrent modifications

**Cannot push** until resolved.

**Resolve with:** `jj bookmark move <name> --to <commit>`

---

## History & Rewriting

### Ancestor

A commit reachable by following parent links backwards.

**Revset:** `ancestors(x)` or `::x`

---

### Descendant

A commit reachable by following child links forward.

**Revset:** `descendants(x)` or `x::`

---

### Parent

The immediate predecessor(s) of a commit.

**Revset symbols:**
- `@-` - Parent of @
- `@--` - Grandparent of @

**Note:** Merge commits have multiple parents.

---

### Child

Immediate successors of a commit.

**Revset symbols:**
- `@+` - Children of @
- `@++` - Grandchildren of @

---

### Merge Commit

A commit with multiple parents, created when integrating parallel lines of development.

**Create with:**
```bash
jj new <parent1> <parent2>
```

**Key point:** jj always creates explicit merge commits (no "fast-forward").

---

### Rebase

Moving a commit (and optionally its descendants) to a new parent.

**Forms:**
- `jj rebase -r` - Move single commit (children stay with old subtree)
- `jj rebase -s` - Move commit and descendants
- `jj rebase -b` - Move descendants only

**Key feature:** Descendants automatically rebase when parent is rewritten.

---

### Squash

Combining changes from one commit into another (usually the parent).

**Command:** `jj squash`

**Effect:** Source commit becomes empty (usually abandoned), target gets combined changes.

---

### Split

Dividing a commit into multiple commits.

**Command:** `jj split`

**Uses:**
- Separate mixed changes
- Create focused commits from large changes
- Interactive or by file selection

---

### Abandon

Hiding a commit from history. Descendants are rebased onto abandoned commit's parents.

**Command:** `jj abandon`

**Key point:** Doesn't delete changes - descendants preserve them. Just removes the commit node.

---

## Conflicts

### Conflict

When changes from different sources can't be automatically merged. In jj, conflicts are materialized in files and commits.

**Markers in log:** `×` symbol

**Key difference from Git:** Conflicts don't block operations. You can continue working and resolve later.

---

### Conflict Markers

Text markers in files showing conflicting changes.

**Format:**
```
<<<<<<< Conflict 1 of 1
+++++++ Contents of side #1
changes from first side
------- Contents of base
original content
+++++++ Contents of side #2
changes from second side
>>>>>>> Conflict 1 of 1 ends
```

**Resolution:** Edit file to remove markers and keep desired content.

---

### Conflicted Commit

A commit containing unresolved conflicts. Marked with `×` in `jj log`.

**Resolution workflow:**
1. `jj new <conflicted-id>` - Create working copy on conflict
2. Edit files to resolve
3. `jj squash` - Squash resolution into conflicted commit

---

### 3-Way Conflict

A conflict showing three versions: base (original), side #1 (one set of changes), side #2 (other set of changes).

**In jj:** Standard conflict format. Provides context for informed resolution.

---

## Workflow Terms

### Snapshot

Capturing the current state of working directory. jj automatically snapshots before operations.

**When snapshotting happens:**
- Running most jj commands
- Before rewrites
- Automatically to preserve work

**Trigger manually:** `jj status` or `jj diff`

---

### Workspace

A working directory associated with the repository. Can have multiple workspaces.

**Commands:**
- `jj workspace add` - Create additional workspace
- `jj workspace list` - List workspaces

**Use case:** Work on multiple features simultaneously without stashing.

---

### Stale Working Copy

When working directory hasn't been updated to reflect recent operations.

**Error message:** "Working copy is stale"

**Fix:** `jj workspace update-stale` or `jj status`

---

## Git Integration

### Git Backend

jj's storage layer that uses Git's object format. Provides interoperability with Git.

**Benefits:**
- Work with Git remotes (GitHub, GitLab)
- Use Git tools when needed
- Migrate from Git gradually

---

### Git Import

Updating jj's view from Git repository state.

**Command:** `jj git import`

**When needed:** After using git commands in colocated repo (non-colocated repos do this automatically on fetch).

---

### Git Export

Updating Git repository with jj's commits and bookmarks.

**Command:** `jj git export`

**When needed:** Non-colocated repos before using Git tools.

---

## Comparison to Git

### Staging Area vs. Working Copy

**Git model:**
- Working directory → Staging area (index) → Repository
- Must `git add` before committing

**jj model:**
- Working directory = working-copy commit
- No explicit staging
- Changes automatically tracked

---

### Branch vs. Bookmark

**Git branch:**
- Automatically advances with commits
- Must be on a branch (or detached HEAD)
- Branch moves as you commit

**jj bookmark:**
- Doesn't automatically advance
- Optional - can work without bookmarks
- Must move manually

---

### Detached HEAD vs. Normal State

**Git:** Detached HEAD is a special (sometimes scary) state.

**jj:** No such concept. Every commit is equal. Working copy can be on any commit.

---

### Reflog vs. Operation Log

**Git reflog:**
- Per-reference history
- Limited information
- Can be lost

**jj operation log:**
- Complete operation history
- Full repository state at each operation
- Comprehensive safety net

---

## Symbols & Notation

| Symbol | Meaning |
|--------|---------|
| `@` | Working-copy commit |
| `@-` | Parent of working copy |
| `@+` | Children of working copy |
| `×` | Conflicted commit |
| `??` | Conflicted bookmark |
| `::` | Ancestry relationship |
| `..` | Range (roughly) |
| `~` | Negation in revsets |
| `\|` | Union in revsets |
| `&` | Intersection in revsets |

---

## See Also

- [SKILL.md](SKILL.md) - Main skill documentation
- [REFERENCE.md](REFERENCE.md) - Command reference with revset details
- [GIT-COMPARISON.md](GIT-COMPARISON.md) - Detailed Git comparisons
- [WORKFLOWS.md](WORKFLOWS.md) - Practical workflows
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problem solutions
