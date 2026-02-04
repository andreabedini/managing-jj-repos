# Git to Jujutsu (jj) Comparison Guide

Comprehensive translation guide for Git users transitioning to jj.

## Mental Model Differences

Understanding these fundamental differences will help you think in jj:

### No Staging Area

**Git model:**
- Working directory → Staging area (index) → Repository
- Explicitly `git add` files before committing
- Can stage partial changes

**jj model:**
- Working directory = working-copy commit
- Changes automatically tracked in `@` commit
- No explicit staging step needed
- Use `jj split` or `jj squash -i` for partial commits

**Implication:** You're always working in a commit. Running `jj describe` or `jj new` finalizes the current commit.

### No Current Branch Concept

**Git model:**
- Always on a branch (or detached HEAD)
- Commits automatically advance the current branch
- Branch moves with your work

**jj model:**
- Bookmarks (similar to branches) don't automatically advance
- Working copy is just a commit, may or may not have bookmark
- Bookmarks are optional pointers you manage explicitly

**Implication:** Create bookmarks when you need named references, especially for pushing. Move them manually with `jj bookmark move`.

### Automatic Descendant Rebase

**Git model:**
- Rebasing children requires explicit commands
- Easy to lose track of dependent branches
- Manual coordination needed

**jj model:**
- Children automatically rebase when parent is rewritten
- Entire subtrees stay consistent
- Less manual rebase management

**Implication:** Edit history freely. Descendants will follow automatically.

### First-Class Conflicts

**Git model:**
- Conflicts block operations
- Must resolve before continuing
- Worktree enters conflicted state

**jj model:**
- Conflicts are materialized in commits
- Can work on other things while conflicts exist
- Marked with `×` in log, resolved when convenient

**Implication:** Don't panic when you see conflicts. They're just markers, not blockers.

## Command Translation Table

### Basic Operations

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git init` | `jj git init` | Creates jj repo with Git backend |
| `git clone <url>` | `jj git clone <url>` | Clone with jj interface |
| `git status` | `jj status` or `jj st` | Similar output |
| `git log` | `jj log` | Better default format |
| `git diff` | `jj diff` | Diff working copy |
| `git show <commit>` | `jj show <rev>` | Show commit with diff |

### Making Commits

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git add <file>` | *Not needed* | Working copy auto-tracked |
| `git add -p` | `jj split -i` | Interactive selection |
| `git commit` | `jj commit` | Creates commit, opens editor |
| `git commit -m "msg"` | `jj commit -m "msg"` | Atomic commit with message |
| `git commit --amend` | `jj describe` | Edit current commit description |
| `git commit --amend` (content) | `jj squash` | Squash changes into parent |
| `git commit --allow-empty` | `jj new` | Create empty commit |

### Branching

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git branch` | `jj bookmark list` | List bookmarks |
| `git branch <name>` | `jj bookmark create <name>` | Create bookmark |
| `git checkout <branch>` | `jj new <bookmark>` | Create new commit on bookmark |
| `git checkout -b <name>` | `jj bookmark create <name>` | Then continue working |
| `git switch <branch>` | `jj edit <bookmark>` | Move working copy |
| `git branch -d <name>` | `jj bookmark delete <name>` | Delete bookmark only |
| `git branch -m <old> <new>` | `jj bookmark rename <old> <new>` | Rename bookmark |

### Viewing History

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git log` | `jj log` | Better default graph |
| `git log --oneline` | `jj log -T 'builtin_log_oneline'` | One-line format |
| `git log <file>` | `jj log <file>` | File history |
| `git log --follow <file>` | `jj log <file>` | Follows renames by default |
| `git blame <file>` | `jj file annotate <file>` | Show line origins |
| `git reflog` | `jj op log` | Operation history |

### Undoing Changes

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git reset --soft HEAD~1` | `jj edit @-` | Move to parent |
| `git reset --hard HEAD~1` | `jj abandon @` | Remove current commit |
| `git reset --hard <commit>` | `jj edit <rev>` + `jj abandon` | Two steps |
| `git checkout -- <file>` | `jj restore <file>` | Restore file |
| `git revert <commit>` | `jj backout -r <rev>` | Create inverse commit |
| `git reflog` + reset | `jj undo` | Undo last operation |

### Rewriting History

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git rebase -i` | `jj rebase` / `jj squash` / `jj split` | Multiple focused commands |
| `git rebase <base>` | `jj rebase -d <base>` | Rebase current commit |
| `git rebase --onto` | `jj rebase -r <rev> -d <dest>` | More explicit |
| `git cherry-pick <commit>` | `jj rebase -r <rev> -d @` | Rebase specific commit |
| `git rebase --continue` | *Not needed* | No interactive rebase state |
| `git rebase --abort` | `jj undo` | Undo the rebase operation |

### Merging

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git merge <branch>` | `jj new <base> <branch>` | Create merge commit |
| `git merge --no-ff` | `jj new <base> <branch>` | jj always creates explicit merge |
| `git merge --squash` | `jj squash -r <branch>` | Squash branch into current |
| `git merge --abort` | `jj undo` | Undo merge operation |

### Remote Operations

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git fetch` | `jj git fetch` | Fetch all remotes |
| `git fetch origin` | `jj git fetch --remote origin` | Fetch specific remote |
| `git pull` | `jj git fetch` + rebase/merge | Two-step process |
| `git push` | `jj git push --bookmark <name>` | Must specify bookmark |
| `git push -u origin <branch>` | `jj git push --bookmark <name>` | Sets up tracking |
| `git push --force` | `jj git push --force --bookmark <name>` | Force push |
| `git remote add` | `jj git remote add <name> <url>` | Add remote |

### Stashing

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git stash` | `jj new` | Current commit becomes "stash" |
| `git stash pop` | `jj squash -r <stash-commit>` | Squash stash into current |
| `git stash apply` | `jj new <stash-commit>` | Create new commit from stash |
| `git stash list` | `jj log` | All commits visible in log |

### Tags

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git tag <name>` | `jj bookmark create <name>` | Bookmarks can serve as tags |
| `git tag -a <name>` | `jj bookmark create <name>` | Add description in commit |
| `git tag -d <name>` | `jj bookmark delete <name>` | Delete tag |

### Inspection

| Git Command | jj Equivalent | Notes |
|-------------|---------------|-------|
| `git diff` | `jj diff` | Diff working copy |
| `git diff --cached` | *Not applicable* | No staging area |
| `git diff <commit>` | `jj diff -r <rev>` | Diff specific commit |
| `git diff <a> <b>` | `jj diff --from <a> --to <b>` | Diff between commits |
| `git show <commit>` | `jj show <rev>` | Show commit |

## Common Git Patterns to jj

### Feature Branch Workflow

**Git:**
```bash
git checkout -b feature
# make changes
git add .
git commit -m "Add feature"
git push -u origin feature
```

**jj:**
```bash
# make changes
jj describe -m "Add feature"
jj bookmark create feature
jj git push --bookmark feature
```

### Amending Last Commit

**Git:**
```bash
# make changes
git add .
git commit --amend --no-edit
```

**jj:**
```bash
# make changes
jj squash  # Squashes @ into parent
```

### Interactive Rebase

**Git:**
```bash
git rebase -i HEAD~3
# pick, squash, edit in editor
```

**jj:**
```bash
# Squash specific commit
jj squash -r <change-id>

# Split specific commit
jj split -r <change-id>

# Move commit to new parent
jj rebase -r <change-id> -d <new-parent>
```

### Fixing Older Commit

**Git:**
```bash
git commit --fixup <commit>
git rebase -i --autosquash <base>
```

**jj:**
```bash
# Make changes
jj squash --into <change-id>
# Descendants automatically rebase
```

### Cherry-Pick

**Git:**
```bash
git cherry-pick <commit>
```

**jj:**
```bash
jj rebase -r <change-id> -d @
```

### Viewing What Changed

**Git:**
```bash
git log --oneline --graph --all
```

**jj:**
```bash
jj log  # Already shows graph by default
```

### Undoing Last Commit

**Git:**
```bash
git reset --soft HEAD~1
```

**jj:**
```bash
jj edit @-  # Move working copy to parent
```

### Completely Remove Last Commit

**Git:**
```bash
git reset --hard HEAD~1
```

**jj:**
```bash
jj abandon @
```

## Common Git User Misconceptions

### "Detached HEAD is dangerous"

**In Git:** Detached HEAD means you're not on a branch. Easy to lose commits.

**In jj:** There's no "attached" mode. Every commit stands on its own. Bookmarks are optional pointers.

**Takeaway:** Don't worry about detached state. Create bookmarks only when you need named references.

### "I need to create a branch before working"

**In Git:** Best practice to create branch before commits.

**In jj:** You can work without bookmarks. Create them later when you need to push or name something.

**Takeaway:** Bookmarks are for external coordination (pushing, sharing), not for daily work.

### "Rewriting published history is dangerous"

**In Git:** True - can cause problems for collaborators.

**In jj:** Still true, but less dangerous. Change IDs help track rewritten commits. Better tools for handling divergence.

**Takeaway:** Still avoid rewriting pushed commits, but jj makes recovery easier if you do.

### "I'll lose work without branches"

**In Git:** Unreferenced commits get garbage collected.

**In jj:** Operation log tracks everything. `jj undo` works even after garbage collection (until operation log is pruned).

**Takeaway:** Much harder to lose work in jj. Operation log is your safety net.

### "Conflicts must be resolved immediately"

**In Git:** Conflicts block operations. Must resolve before continuing.

**In jj:** Conflicts are just markers. Work on other things, resolve when convenient.

**Takeaway:** Don't let conflicts disrupt your workflow. They'll wait.

### "I need to stage changes carefully"

**In Git:** Staging area is key workflow element.

**in jj:** No staging area. Use `jj split` for selective commits after making changes.

**Takeaway:** Make all your changes, then split/organize as needed.

## Workflow Comparison

### Git Feature Branch Workflow

```bash
git checkout main
git pull
git checkout -b my-feature
# make changes
git add .
git commit -m "Implement feature"
# more changes
git add .
git commit -m "Add tests"
git push -u origin my-feature
# create PR
```

### jj Equivalent Workflow

```bash
jj git fetch
jj new main@origin
# make changes
jj describe -m "Implement feature"
jj new
# more changes
jj describe -m "Add tests"
jj bookmark create my-feature
jj git push --bookmark my-feature
# create PR
```

**Key differences:**
- No explicit branch creation needed upfront
- No staging step
- Bookmark created only for pushing
- Fewer commands overall

## Advanced Concepts

### Revsets vs Git Revision Syntax

**Git:**
- `HEAD`, `HEAD~1`, `HEAD^2`
- `<branch>`, `<tag>`, `<commit-hash>`
- `<rev>^..<rev>` for ranges

**jj:**
- `@` (working copy), `@-` (parent), `@--` (grandparent)
- `<bookmark>`, `<change-id>`, `<commit-hash>`
- `::@` (ancestors), `@::` (descendants)
- Powerful query language: `description(bug)`, `author(alice)`, `mine()`

### Change IDs vs Commit Hashes

**Git:** Only has commit hashes. Rewriting creates new hashes.

**jj:** Has both change IDs (stable) and commit hashes (change on rewrite).

**When to use:**
- Change ID: Referring to a commit you might rewrite
- Commit hash: Exact snapshot reference (rare)

## Migration Tips

1. **Start with colocated repo** - Use `--colocate` to keep Git repo alongside jj
2. **Keep using Git commands initially** - Run `jj git import` to sync to jj
3. **Gradually shift to jj commands** - Move one workflow at a time
4. **Learn revsets** - More powerful than Git revision syntax
5. **Embrace working copy as commit** - Stop thinking about staging
6. **Use operation log** - Your safety net for mistakes
7. **Create bookmarks sparingly** - Only when needed for remotes

## Further Reading

- See [WORKFLOWS.md](WORKFLOWS.md) for step-by-step guides
- See [GLOSSARY.md](GLOSSARY.md) for terminology definitions
- See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues
