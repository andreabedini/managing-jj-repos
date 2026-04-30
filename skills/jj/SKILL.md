---
name: jj
description: Jujutsu (jj) version control operations including commits, rebasing, bookmarks, conflict resolution, and working copy management. Use when user mentions jj, jujutsu, change IDs, working copy, bookmarks, revsets, or jj version control operations. Also use for basic jj workflows and daily operations.
---

# Jujutsu (jj) Version Control

Quick-access skill for daily jj operations. For Git migration help, see `/jj-migrate`. For troubleshooting, see `/jj-troubleshoot`.

## Quick Reference

### Essential Commands
```bash
jj status                # Show working copy status
jj log                   # View commit history
jj describe -m "msg"     # Set description of working copy
jj commit -m "msg"       # Create commit with message
jj new [parent]          # Create new empty commit
jj absorb                # Move working copy changes into the ancestor that last touched each line
jj next [N]              # Move working copy to child (new empty commit on top)
jj prev [N]              # Move working copy to parent
jj git fetch             # Fetch from remote
jj git push --bookmark <name>  # Push bookmark
```

### Core Concepts

**Working copy is a commit:**
- No staging area
- Changes automatically tracked in `@`
- Use `jj describe` to finalize, `jj new` to start fresh

**Change IDs persist:**
- 12-letter stable identifiers
- Survive rewrites (amend, rebase, etc.)
- Use these for referring to commits

**Automatic rebasing:**
- Children automatically rebase when parent changes
- Less manual coordination needed

### Common Workflows

See [WORKFLOWS.md](WORKFLOWS.md) for detailed step-by-step checklists covering:
- Daily commit workflow
- Push to remote
- Collaboration (fetch + rebase/merge)
- Split/squash commits
- Rebase commits
- Conflict resolution

### Quick Command Reference

| Task                    | Command                            |
|-------------------------|-------------------------------------|
| View history           | `jj log`                            |
| View specific commit   | `jj show <change-id>`               |
| Make changes           | Edit files, `jj describe -m "..."`  |
| Create new commit      | `jj new`                            |
| Amend description      | `jj describe -r <id> -m "..."`      |
| Squash into parent     | `jj squash -r <change-id>`          |
| Absorb into ancestors  | `jj absorb`                         |
| Split commit           | `jj split -r <change-id>`           |
| Rebase onto new parent | `jj rebase -r <id> -d <dest>`       |
| Create bookmark        | `jj bookmark create <name>`         |
| Move bookmark          | `jj bookmark move <name> --to <id>` |
| Push bookmark          | `jj git push --bookmark <name>`     |
| Undo last operation    | `jj undo`                           |
| View operation log     | `jj op log`                         |

### Revset Examples

```bash
@                          # Current working copy
@-                         # Parent
@--                        # Grandparent
::@                        # All ancestors of @
@::                        # All descendants of @
main..@                    # Commits in @ but not in main
description(bug)           # Commits mentioning "bug"
author(alice)              # Commits by alice
mine()                     # Your own commits
files(src/)                # Commits touching src/ (combine with & | ~)
conflicts()                # Commits with unresolved conflicts
mutable()                  # All mutable (rewritable) commits
reachable(@, mutable())    # Your current working stack
merges()                   # Merge commits
bookmarks()                # Commits pointed to by any bookmark
latest(x, 5)               # Most recent 5 commits in set x
author_date(after:"yesterday")  # Commits authored since yesterday
```

## Safety Warnings

⚠️ **Don't use `jj undo` after `jj git push`**
- Creates stale remote information
- Instead: create forward fixes or use `jj bookmark move`

⚠️ **Don't mix git and jj commands**
- Causes state inconsistencies in colocated repos
- Stick to one tool per session

⚠️ **Conflicted bookmarks (??) cannot be pushed**
- Resolve with `jj bookmark move <name> --to <commit>`

## Progressive Disclosure

**For comprehensive details:**
- **[WORKFLOWS.md](WORKFLOWS.md)** - Step-by-step checklists (beginner/intermediate/advanced)
- **[REFERENCE.md](../jj-reference/REFERENCE.md)** - Complete command reference with all options
- **[GLOSSARY.md](../jj-reference/GLOSSARY.md)** - Term definitions

**For Git users:**
- **`/jj-migrate`** - Git to jj migration help

**For problems:**
- **`/jj-troubleshoot`** - Troubleshooting and recovery

## Examples

See [examples/](examples/) directory for real-world scenarios:
- Feature branch workflow
- Conflict resolution walkthrough
- History rewriting patterns

## When to Load Other Skills

- **Load `/jj-migrate`** when user mentions Git migration, git commands, or transitioning from Git
- **Load `/jj-troubleshoot`** when user encounters errors, unexpected behavior, or needs recovery
- Reference documentation (REFERENCE.md, GLOSSARY.md) loaded automatically when needed

## Best Practices

1. **Run `jj status` regularly** - Ensures working copy snapshotted
2. **Use change IDs for rewriting** - They persist through rewrites
3. **Create bookmarks only when pushing** - Not required for local work
4. **Fetch frequently** - Keeps you synced
5. **Resolve conflicts promptly** - Easier than letting them stack
6. **Use operation log for recovery** - `jj op log` is your safety net
