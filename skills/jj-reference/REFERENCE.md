# Jujutsu Command Reference

Comprehensive reference for all jj commands with common options and usage patterns.

## Command Categories

- [Initialization & Git Integration](#initialization--git-integration)
- [Viewing & Navigation](#viewing--navigation)
- [Creating & Editing Commits](#creating--editing-commits)
- [Rewriting History](#rewriting-history)
- [Bookmarks](#bookmarks)
- [Tags](#tags)
- [Working Copy Management](#working-copy-management)
- [Recovery & Operations](#recovery--operations)
- [Configuration](#configuration)
- [Revset Syntax](#revset-syntax)

---

## Initialization & Git Integration

### `jj git init`

Initialize new jj repository with Git backend.

**Usage:**
```bash
jj git init [<directory>]
jj git init --colocate [<directory>]
```

**Options:**
- `--colocate` - Create colocated workspace (shares directory with Git)
- `<directory>` - Directory to initialize (default: current directory)

**Examples:**
```bash
jj git init                    # Initialize in current directory
jj git init my-project         # Initialize new directory
jj git init --colocate         # Colocated with Git
```

---

### `jj git clone`

Clone repository from Git remote.

**Usage:**
```bash
jj git clone <source> [<destination>]
jj git clone --colocate <source> [<destination>]
```

**Options:**
- `--colocate` - Create colocated workspace
- `<source>` - URL or path to clone from
- `<destination>` - Directory to clone into (default: derived from source)

**Examples:**
```bash
jj git clone https://github.com/user/repo
jj git clone --colocate https://github.com/user/repo
jj git clone git@github.com:user/repo.git my-dir
```

---

### `jj git fetch`

Fetch changes from Git remotes.

**Usage:**
```bash
jj git fetch [options]
```

**Options:**
- `--remote <name>` - Fetch from specific remote (default: all remotes)
- `--branch <name>` - Fetch specific branch

**Examples:**
```bash
jj git fetch                       # Fetch all remotes
jj git fetch --remote origin       # Fetch from origin
jj git fetch --branch main         # Fetch main branch
```

---

### `jj git push`

Push changes to Git remote.

**Usage:**
```bash
jj git push [options]
```

**Options:**
- `--bookmark <name>` - Push specific bookmark
- `--all` - Push all bookmarks
- `--change <id>` - Push specific change
- `--remote <name>` - Push to specific remote (default: origin)
- `--force` - Force push (overwrite remote)

**Examples:**
```bash
jj git push --bookmark main           # Push main bookmark
jj git push --all                     # Push all bookmarks
jj git push --change abc123           # Push specific change
jj git push --bookmark main --force   # Force push
```

---

### `jj git import`

Import Git refs to jj (non-colocated repos).

**Usage:**
```bash
jj git import
```

Updates jj's view of bookmarks and commits from Git repository.

---

### `jj git export`

Export jj commits to Git (non-colocated repos).

**Usage:**
```bash
jj git export
```

Updates Git repository with jj's commits and bookmarks.

---

### `jj git remote`

Manage Git remotes.

**Usage:**
```bash
jj git remote add <name> <url>
jj git remote remove <name>
jj git remote rename <old> <new>
jj git remote list
```

**Examples:**
```bash
jj git remote add origin https://github.com/user/repo
jj git remote list
jj git remote remove origin
```

---

## Viewing & Navigation

### `jj status` / `jj st`

Show working copy status.

**Usage:**
```bash
jj status [options]
```

**Options:**
- `-r <revset>` - Show status of specific revision

**Output:**
- Working copy commit ID
- Parent commit(s)
- Changed files
- Conflicts (if any)

**Examples:**
```bash
jj status          # Show working copy status
jj st              # Short alias
```

---

### `jj log`

Show commit history as a graph.

**Usage:**
```bash
jj log [options] [paths...]
```

**Options:**
- `-r <revset>` - Show specific revisions (default: all visible)
- `-n <number>` - Limit number of commits
- `-T <template>` - Use custom template
- `--reversed` - Show in reverse order (oldest first)
- `paths...` - Filter by file paths

**Examples:**
```bash
jj log                          # Show all history
jj log -n 10                    # Show last 10 commits
jj log -r ::@                   # Show ancestors of @
jj log -r main..@               # Show commits between main and @
jj log src/                     # Show commits touching src/
jj log -T 'builtin_log_oneline' # One-line format
```

**Symbols in output:**
- `@` - Working copy commit
- `×` - Conflicted commit
- `??` - Conflicted bookmark

---

### `jj show`

Show commit with diff.

**Usage:**
```bash
jj show [<revision>]
```

**Options:**
- `<revision>` - Revision to show (default: `@`)

**Examples:**
```bash
jj show           # Show working copy
jj show @-        # Show parent
jj show abc123    # Show specific change
```

---

### `jj diff`

Show changes in commits.

**Usage:**
```bash
jj diff [options] [paths...]
```

**Options:**
- `-r <revset>` - Show diff of specific revision (default: `@`)
- `--from <rev>` - Show changes from revision
- `--to <rev>` - Show changes to revision
- `--summary` - Show summary only (no line-by-line diff)
- `paths...` - Filter by file paths

**Examples:**
```bash
jj diff                    # Diff working copy
jj diff -r @-              # Diff parent commit
jj diff --from @- --to @   # Diff between two commits
jj diff src/main.rs        # Diff specific file
jj diff --summary          # Show changed files only
```

---

### `jj diffedit`

Touch up the content changes in a revision using a diff editor.

**Usage:**
```bash
jj diffedit [OPTIONS] [FILESETS]...
```

**Options:**
- `-r <REVSET>` - Revision to touch up (default: `@`)
- `--from <REVSET>` - Show changes from this revision
- `--to <REVSET>` - Edit changes in this revision
- `--tool <NAME>` - Specify diff editor to use
- `--restore-descendants` - Preserve content (not diff) when rebasing descendants

**Examples:**
```bash
jj diffedit                        # Touch up working copy
jj diffedit -r abc123              # Touch up specific commit
jj diffedit --from @- --to @       # Edit changes between two commits
```

For moving changes between revisions interactively, prefer `jj squash -i`. For restoring entire files, use `jj restore`.

---

### `jj interdiff`

Show differences between the diffs of two revisions — i.e., compare what each commit *does* rather than comparing file contents directly.

Technically, jj rebases `--from` onto `--to`'s parents, then diffs the result against `--to`. This means differences in parent history don't pollute the comparison.

**Usage:**
```bash
jj interdiff --from <REVSET> --to <REVSET> [FILESETS]...
```

**Key options:**
- `-f, --from <REVSET>` - First revision (default: `@`)
- `-t, --to <REVSET>` - Second revision (default: `@`)
- `--git` - Git-format diff output
- `--stat` - Histogram of changes
- `--summary / -s` - Files changed only, no line diff
- `--name-only` - Paths only
- `--color-words` - Word-level diff
- `-w` - Ignore all whitespace
- `[FILESETS]...` - Restrict to specific paths

**Contrast with `jj diff --from A --to B`:**
- `jj diff --from A --to B` compares file trees — it includes everything between A and B's parents
- `jj interdiff --from A --to B` compares patches — if A and B touch the same files in the same way, output is empty, regardless of where they sit in history

**Examples:**
```bash
# Compare how a change has evolved since it was last pushed
jj interdiff --from push-xyz@origin --to push-xyz

# Check if two divergent versions of a commit are equivalent (empty = identical patches)
jj interdiff --from COMMIT_A --to COMMIT_B

# Same, git-format output for piping/scripting
jj interdiff --from COMMIT_A --to COMMIT_B --git
```

**Use `jj evolog -p` instead** when you want the full evolution of a single change across all its rewrites.

---

### `jj evolog`

Show evolution of a change over time.

**Usage:**
```bash
jj evolog [options]
```

**Options:**
- `-r <revset>` - Show evolution of specific revision
- `-n <number>` - Limit number of versions

Shows how a change ID has evolved through rewrites.

**Examples:**
```bash
jj evolog -r @       # Show evolution of current commit
jj evolog -r abc123  # Show evolution of change
```

---

### `jj file annotate`

Show line-by-line origins (like `git blame`).

**Usage:**
```bash
jj file annotate <path>
```

**Examples:**
```bash
jj file annotate src/main.rs
```

---

## Creating & Editing Commits

### `jj new`

Create new empty commit.

**Usage:**
```bash
jj new [<parents>...]
```

**Options:**
- `<parents>...` - Parent commit(s) (default: `@`)
- `-m <message>` - Set commit description
- `--no-edit` - Don't open editor

**Examples:**
```bash
jj new                  # New commit after @
jj new @-               # New commit after parent
jj new main@origin      # New commit after remote main
jj new @- main          # New merge commit
```

**Common pattern:**
```bash
# Make changes
jj describe -m "First commit"
jj new
# Make more changes
jj describe -m "Second commit"
```

---

### `jj commit`

Create commit from working copy with description.

**Usage:**
```bash
jj commit [options]
```

**Options:**
- `-m <message>` - Set commit message
- `--no-edit` - Don't open editor

Equivalent to `jj describe` followed by `jj new`.

**Examples:**
```bash
jj commit -m "Add feature"    # Atomic commit
jj commit                      # Opens editor for message
```

---

### `jj describe`

Edit commit description.

**Usage:**
```bash
jj describe [options]
```

**Options:**
- `-r <revset>` - Describe specific revision (default: `@`)
- `-m <message>` - Set message directly
- `--no-edit` - Clear description

**Examples:**
```bash
jj describe -m "Fix bug in auth"     # Set description of @
jj describe -r @- -m "Update docs"   # Set description of parent
jj describe                          # Open editor
```

---

### `jj edit`

Move working copy to different commit.

**Usage:**
```bash
jj edit <revision>
```

Makes the specified revision the working copy. Current working copy changes are snapshotted first.

**Examples:**
```bash
jj edit @-          # Move to parent
jj edit main        # Move to main
jj edit abc123      # Move to specific commit
```

**Warning:** Changes your working directory files!

---

### `jj next`

Move the working copy to a child revision.

**Usage:**
```bash
jj next [OPTIONS] [OFFSET]
```

**Options:**
- `[OFFSET]` - How many revisions to move forward (default: 1)
- `-e, --edit` - Edit the target commit directly instead of creating a new commit on top
- `--conflict` - Jump to the next conflicted descendant

**Examples:**
```bash
jj next          # Move to child (new commit on top)
jj next 3        # Move forward 3 revisions
jj next --edit   # Edit the child commit directly
jj next --conflict  # Jump to next conflict
```

---

### `jj prev`

Move the working copy to a parent revision.

**Usage:**
```bash
jj prev [OPTIONS] [OFFSET]
```

**Options:**
- `[OFFSET]` - How many revisions to move backward (default: 1)
- `-e, --edit` - Edit the parent commit directly
- `--conflict` - Jump to the previous conflicted ancestor

**Examples:**
```bash
jj prev          # Move to parent
jj prev 2        # Move back 2 revisions
jj prev --edit   # Edit the parent directly
```

---

### `jj metaedit`

Modify the metadata of a revision without changing its content.

**Usage:**
```bash
jj metaedit [OPTIONS] [REVSETS]...
```

**Options:**
- `--update-change-id` - Generate a new change ID for the revision
- `-m <MESSAGE>` - Update the description
- `--update-author` - Update author to the configured user
- `--author <AUTHOR>` - Set author to the provided string (e.g. `"Name <email>"`)
- `--update-author-timestamp` - Update author date to now
- `--author-timestamp <TIMESTAMP>` - Set author date (RFC2822 or RFC3339)
- `--force-rewrite` - Rewrite commit even if no metadata changed (updates committer timestamp)

**Examples:**
```bash
jj metaedit --update-change-id @       # Resolve divergence by assigning fresh change ID
jj metaedit --update-author            # Fix author on working copy
jj metaedit --author "Name <e@mail>"   # Set specific author
jj metaedit -m "Better description"    # Update description without opening editor
```

---

### `jj abandon`

Hide commit from history.

**Usage:**
```bash
jj abandon [<revisions>...]
```

**Options:**
- `<revisions>...` - Commits to abandon (default: `@`)

Descendants are rebased onto the abandoned commit's parents.

**Examples:**
```bash
jj abandon              # Abandon working copy
jj abandon @-           # Abandon parent
jj abandon abc123       # Abandon specific commit
```

---

## Rewriting History

### `jj squash`

Squash changes into another commit.

**Usage:**
```bash
jj squash [options]
```

**Options:**
- `-r <revset>` - Squash specific revision (default: `@`)
- `--into <dest>` - Squash into specific commit (default: parent)
- `-i` / `--interactive` - Interactively select changes
- `-m <message>` - Set description of result

**Examples:**
```bash
jj squash                      # Squash @ into parent
jj squash -r abc123            # Squash commit into its parent
jj squash --into xyz789        # Squash @ into specific commit
jj squash -i                   # Interactively select changes
```

**Common pattern:**
```bash
# Fix earlier commit
jj new <commit-to-fix>
# Make fixes
jj squash  # Squash fixes into parent
```

---

### `jj split`

Split commit into multiple commits.

**Usage:**
```bash
jj split [options] [paths...]
```

**Options:**
- `-r <revset>` - Split specific revision (default: `@`)
- `-i` / `--interactive` - Interactively select changes (TUI)
- `paths...` - Files to put in first commit (rest in second)

**Examples:**
```bash
jj split                          # Interactive split of @
jj split file1.txt file2.txt      # Split by files
jj split -r abc123                # Split specific commit
```

**In TUI:**
- Space: Toggle change selection
- A: Select all
- Q: Confirm and split

---

### `jj rebase`

Move commits to different parents.

**Usage:**
```bash
jj rebase [options]
```

**Options:**
- `-r <revset>` - Rebase single commit (children stay)
- `-s <revset>` - Rebase commit and descendants
- `-b <revset>` - Rebase descendants of commit
- `-d <destination>` - New parent destination

**Examples:**
```bash
# Rebase current commit onto main
jj rebase -d main

# Rebase commit and descendants
jj rebase -s abc123 -d main

# Rebase single commit (children rebase onto result)
jj rebase -r abc123 -d xyz789

# Rebase all descendants of commit
jj rebase -b abc123 -d main
```

**Common patterns:**
```bash
# Update feature branch on main
jj rebase -s <first-feature-commit> -d main@origin

# Cherry-pick commit
jj rebase -r <commit> -d @

# Move commit between two others
jj rebase -r <commit> -d <new-parent>
```

---

### `jj duplicate`

Create copy of commits.

**Usage:**
```bash
jj duplicate [<revisions>...]
```

**Examples:**
```bash
jj duplicate abc123           # Duplicate commit
jj duplicate -r main..@       # Duplicate range
```

Use `jj rebase` to move duplicate to desired location.

---

### `jj parallelize`

Make commits siblings instead of stack.

**Usage:**
```bash
jj parallelize [<revisions>...]
```

Converts a linear stack into parallel (sibling) commits with the same parent.

**Examples:**
```bash
jj parallelize @---::@        # Make last 3 commits siblings
```

---

### `jj revert`

Apply the reverse of one or more revisions (replaces the old `jj backout`).

**Usage:**
```bash
jj revert -r <REVSETS> <--onto <REVSETS>|--insert-after <REVSETS>|--insert-before <REVSETS>>
```

**Options:**
- `-r <REVSETS>` - The revision(s) to reverse
- `-o, --onto <REVSETS>` - Apply reverse on top of these revisions (aliases: `-d`, `--destination`)
- `-A, --insert-after <REVSETS>` - Insert after these revisions (aliases: `--after`)
- `-B, --insert-before <REVSETS>` - Insert before these revisions (aliases: `--before`)

**Examples:**
```bash
jj revert -r abc123 --onto @        # Apply reverse on top of working copy
jj revert -r abc123 --insert-after main  # Insert after main
```

---

### `jj absorb`

Move changes from a revision into the stack of mutable ancestors where those lines were last modified.

**Usage:**
```bash
jj absorb [OPTIONS] [FILESETS]...
```

**Options:**
- `-f, --from <REVSET>` - Source revision (default: `@`)
- `-t, --into <REVSETS>` - Destination revisions (default: `mutable()`, ancestors only)
- `[FILESETS]...` - Restrict to specific paths

If all changes are absorbed and the source has no description, it is automatically abandoned. Review with `jj op show -p`.

**Examples:**
```bash
jj absorb                  # Absorb all working copy changes into appropriate ancestors
jj absorb src/lib.rs       # Absorb changes to one file only
jj absorb --from @         # Explicit source
```

---

### `jj arrange`

Interactively arrange the commit graph using a TUI.

**Usage:**
```bash
jj arrange [REVSETS]...
```

**Options:**
- `[REVSETS]...` - Revisions to arrange (default: `revsets.arrange` config)

---

### `jj simplify-parents`

Remove redundant parent edges — parents that are already reachable through other parents.

**Usage:**
```bash
jj simplify-parents [OPTIONS]
```

**Options:**
- `-r, --revision <REVSETS>` - Simplify these specific revisions
- `-s, --source <REVSETS>` - Simplify these revisions and their mutable descendants

Default when no flags given: `reachable(@, mutable())`.

**Examples:**
```bash
jj simplify-parents             # Clean up entire mutable stack
jj simplify-parents -r abc123   # Clean up one commit
```

---

### `jj fix`

Apply code formatters or other file-content tools across mutable revisions.

**Usage:**
```bash
jj fix [OPTIONS] [FILESETS]...
```

**Options:**
- `-s, --source <REVSETS>` - Fix this revision and all descendants (default: `reachable(@, mutable())`)
- `--include-unchanged-files` - Also fix files not modified by the revision
- `[FILESETS]...` - Restrict to specific paths

Requires configuration in `.jj/config.toml` or `~/.jjconfig.toml`:
```toml
[fix.tools.clang-format]
command = ["/usr/bin/clang-format", "--assume-filename=$path"]
patterns = ["glob:'**/*.cc'", "glob:'**/*.h'"]
```

Tools must produce deterministic output (jj deduplicates identical inputs). Review with `jj op show -p`.

**Examples:**
```bash
jj fix                          # Format all mutable revisions
jj fix -s abc123                # Format from a specific commit down
jj fix src/                     # Format only src/ files
```

---

### `jj resolve`

Resolve conflicts in files.

**Usage:**
```bash
jj resolve [<paths>...]
```

**Options:**
- `paths...` - Files to resolve

Helps resolve conflicts by marking them resolved.

**Examples:**
```bash
jj resolve src/main.rs
jj resolve --list             # List conflicted files
```

---

## Bookmarks

### `jj bookmark create`

Create new bookmark.

**Usage:**
```bash
jj bookmark create <name> [options]
```

**Options:**
- `-r <revset>` - Create at specific revision (default: `@`)

**Examples:**
```bash
jj bookmark create feature           # Create at @
jj bookmark create main -r abc123    # Create at specific commit
```

---

### `jj bookmark move`

Move bookmark to different commit.

**Usage:**
```bash
jj bookmark move <name> --to <revision>
```

**Examples:**
```bash
jj bookmark move main --to @         # Move main to working copy
jj bookmark move feature --to @-     # Move feature to parent
```

---

### `jj bookmark delete`

Delete bookmark.

**Usage:**
```bash
jj bookmark delete <name>
```

**Examples:**
```bash
jj bookmark delete old-feature
```

**Note:** Only deletes the bookmark pointer, not commits.

---

### `jj bookmark list`

List bookmarks.

**Usage:**
```bash
jj bookmark list [options]
```

**Options:**
- `-a` / `--all` - Include remote bookmarks
- `-r <revset>` - Filter by revision

**Examples:**
```bash
jj bookmark list              # List local bookmarks
jj bookmark list -a           # Include remotes
```

**Output symbols:**
- `??` - Conflicted bookmark

---

### `jj bookmark track`

Track remote bookmark locally.

**Usage:**
```bash
jj bookmark track <name>@<remote>
```

**Examples:**
```bash
jj bookmark track main@origin
jj bookmark track feature@upstream
```

---

### `jj bookmark untrack`

Stop tracking remote bookmark.

**Usage:**
```bash
jj bookmark untrack <name>
```

**Examples:**
```bash
jj bookmark untrack main
```

---

### `jj bookmark rename`

Rename bookmark.

**Usage:**
```bash
jj bookmark rename <old> <new>
```

**Examples:**
```bash
jj bookmark rename old-name new-name
```

---

## Tags

Tags are immutable named pointers to commits (unlike bookmarks, which move). They map to Git tags in colocated repos.

### `jj tag set`

Create or update a tag.

**Usage:**
```bash
jj tag set <name> [-r <REVSET>]
```

**Examples:**
```bash
jj tag set v1.0.0              # Tag working copy
jj tag set v1.0.0 -r abc123    # Tag specific commit
```

---

### `jj tag list`

List tags.

**Usage:**
```bash
jj tag list
```

---

### `jj tag delete`

Delete a tag.

**Usage:**
```bash
jj tag delete <name>
```

---

## Working Copy Management

### `jj workspace add`

Create additional workspace.

**Usage:**
```bash
jj workspace add --name <name> <path>
```

**Examples:**
```bash
jj workspace add --name feature ../feature-workspace
```

---

### `jj workspace list`

List all workspaces.

**Usage:**
```bash
jj workspace list
```

Shows workspace names and their working copy commits.

---

### `jj workspace forget`

Remove workspace.

**Usage:**
```bash
jj workspace forget <name>
```

---

### `jj workspace update-stale`

Update stale working copy.

**Usage:**
```bash
jj workspace update-stale
```

Use when you see "stale working copy" error.

---

### `jj restore`

Restore files to revision state.

**Usage:**
```bash
jj restore [options] [paths...]
```

**Options:**
- `--from <rev>` - Restore from revision
- `--to <rev>` - Restore to revision (default: `@`)
- `paths...` - Files to restore

**Examples:**
```bash
jj restore src/main.rs           # Restore file in @
jj restore --from @- .           # Restore all from parent
```

---

## Recovery & Operations

### `jj undo`

Undo recent operation.

**Usage:**
```bash
jj undo [options]
```

**Options:**
- `-n <number>` - Undo multiple operations

**Examples:**
```bash
jj undo           # Undo last operation
jj undo -n 3      # Undo last 3 operations
```

**Warning:** Don't undo after push operations!

---

### `jj redo`

Redo undone operation.

**Usage:**
```bash
jj redo [options]
```

**Options:**
- `-n <number>` - Redo multiple operations

**Examples:**
```bash
jj redo           # Redo last undo
```

---

### `jj op log`

Show operation history.

**Usage:**
```bash
jj op log [options]
```

**Options:**
- `-n <number>` - Limit number of operations

**Examples:**
```bash
jj op log              # Show all operations
jj op log -n 10        # Show last 10 operations
```

**Output:** Shows operation IDs, timestamps, descriptions.

---

### `jj op show`

Show details of specific operation.

**Usage:**
```bash
jj op show <operation-id>
```

**Examples:**
```bash
jj op show abc123
```

---

### `jj op restore`

Restore repository to specific operation.

**Usage:**
```bash
jj op restore <operation-id>
```

**Examples:**
```bash
jj op restore abc123
```

Reverts entire repository state to that operation.

---

### `jj --at-op`

Run command at past operation state.

**Usage:**
```bash
jj --at-op=<operation-id> <command>
```

**Examples:**
```bash
jj --at-op=abc123 log      # View log as it was
jj --at-op=abc123 show     # View commit as it was
```

Doesn't change current state, just views past.

---

## Configuration

### `jj config list`

List all configuration settings.

**Usage:**
```bash
jj config list [options]
```

**Options:**
- `--user` - Show user config only
- `--repo` - Show repo config only

---

### `jj config set`

Set configuration value.

**Usage:**
```bash
jj config set [--user|--repo] <name> <value>
```

**Options:**
- `--user` - Set in user config (~/.jjconfig.toml)
- `--repo` - Set in repo config (.jj/repo/config.toml)

**Examples:**
```bash
jj config set --user user.name "Your Name"
jj config set --user user.email "you@example.com"
jj config set --user ui.editor "vim"
jj config set --user ui.default-command "log"
```

---

### `jj config get`

Get configuration value.

**Usage:**
```bash
jj config get <name>
```

**Examples:**
```bash
jj config get user.name
```

---

### `jj config edit`

Edit configuration file in editor.

**Usage:**
```bash
jj config edit [--user|--repo]
```

**Examples:**
```bash
jj config edit --user      # Edit user config
jj config edit --repo      # Edit repo config
```

---

## Revset Syntax

Revsets are jj's query language for selecting commits.

### Symbols

| Syntax | Meaning |
|--------|---------|
| `@` | Working copy commit |
| `<bookmark>` | Commit pointed to by bookmark |
| `<change-id>` | Commit with change ID |
| `<commit-hash>` | Commit with hash |
| `root()` | Virtual root commit |

### Parents & Children

| Syntax | Meaning |
|--------|---------|
| `@-` | Parent of @ |
| `@--` | Grandparent of @ |
| `@+` | Children of @ |
| `@++` | Grandchildren of @ |

### Ranges & Ancestry

| Syntax | Meaning |
|--------|---------|
| `::@` | All ancestors of @ (including @) |
| `@::` | All descendants of @ (including @) |
| `x::y` | Ancestors of y that are descendants of x |
| `x..y` | Commits reachable from y but not x |

### Boolean Operations

| Syntax | Meaning |
|--------|---------|
| `x \| y` | Union (x or y) |
| `x & y` | Intersection (x and y) |
| `~x` | Negation (not x) |

### Functions

| Function | Meaning |
|----------|---------|
| `description(pattern)` | Commits with pattern in description |
| `author(pattern)` | Commits by author |
| `committer(pattern)` | Commits by committer |
| `mine()` | Your commits |
| `empty()` | Commits with no changes |
| `heads(x)` | Heads (commits with no descendants) in x |
| `roots(x)` | Roots (commits with no parents) in x |
| `ancestors(x)` | All ancestors of x |
| `descendants(x)` | All descendants of x |
| `connected(x)` | Commits connected to x |
| `all()` | All visible commits |
| `visible_heads()` | All head commits |

### Practical Examples

```bash
# All ancestors of working copy
jj log -r ::@

# Commits between main and working copy
jj log -r main::@

# Your commits not in main
jj log -r 'mine() & ~::main'

# Commits mentioning "bug"
jj log -r 'description(bug)'

# Empty commits
jj log -r 'empty()'

# Commits by Alice
jj log -r 'author(alice)'

# Recent commits (last 10)
jj log -n 10

# All heads
jj log -r 'heads(all())'

# Descendants of main
jj log -r 'main::'
```

---

## Template Syntax

Templates customize output format.

### Built-in Templates

| Template | Description |
|----------|-------------|
| `builtin_log_oneline` | One-line log format |
| `builtin_log_compact` | Compact log format |
| `builtin_log_comfortable` | Default log format |

**Usage:**
```bash
jj log -T 'builtin_log_oneline'
```

### Custom Templates

**Set default:**
```bash
jj config set --user templates.log 'builtin_log_oneline'
```

---

## See Also

- [SKILL.md](SKILL.md) - Main skill overview
- [WORKFLOWS.md](WORKFLOWS.md) - Step-by-step workflows
- [GIT-COMPARISON.md](GIT-COMPARISON.md) - Git to jj translation
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Problem solutions
- [GLOSSARY.md](GLOSSARY.md) - Terminology definitions
