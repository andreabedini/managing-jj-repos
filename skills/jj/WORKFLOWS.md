# Jujutsu Workflows

Step-by-step checklists for common jj operations, organized by skill level.

## Level 1: Beginner Workflows

### First-Time Setup

Initial configuration for new jj users.

**Task Progress:**
- [ ] Install jj (via package manager or from source)
- [ ] Configure user identity:
  ```bash
  jj config set --user user.name "Your Name"
  jj config set --user user.email "you@example.com"
  ```
- [ ] Configure editor (optional):
  ```bash
  jj config set --user ui.editor "vim"
  # or "nano", "code --wait", "emacs", etc.
  ```
- [ ] Set default command (optional):
  ```bash
  jj config set --user ui.default-command "log"
  # Now running `jj` alone shows log
  ```
- [ ] Understand core concepts:
  - [ ] Working copy is an automatic commit (`@`)
  - [ ] No staging area exists
  - [ ] Change IDs persist through rewrites
  - [ ] Bookmarks don't auto-advance

**Success criteria:** Config commands run without errors, `jj config list` shows your settings.

---

### Initialize Repository

Create a new jj repository.

**Option A: New repository**
- [ ] Navigate to project directory
- [ ] Run: `jj git init`
- [ ] Verify: `jj status` should show empty working copy
- [ ] Add initial files
- [ ] Run: `jj describe -m "Initial commit"`

**Option B: Clone existing repository**
- [ ] Run: `jj git clone <url> [directory-name]`
- [ ] Navigate into cloned directory
- [ ] Run: `jj log` to view history
- [ ] Run: `jj status` to verify clean state

**Option C: Colocated with Git**
- [ ] Use `--colocate` flag: `jj git init --colocate` or `jj git clone --colocate <url>`
- [ ] Verify both `.jj` and `.git` directories exist
- [ ] Understand: Can use Git tools, but prefer jj commands

**Success criteria:** Repository initialized, `jj log` shows history, `jj status` works.

---

### Daily Commit Workflow

Basic workflow for making and committing changes.

**Task Progress:**
- [ ] Make changes in your working directory (edit, create, delete files)
- [ ] Check what changed: `jj status`
- [ ] Review changes: `jj diff`
- [ ] Describe the working-copy commit:
  ```bash
  jj describe -m "Add user authentication module"
  ```
- [ ] Create new empty working-copy commit:
  ```bash
  jj new
  ```
- [ ] Verify commit is in history: `jj log`

**Alternative - atomic commit:**
- [ ] Make changes
- [ ] Run: `jj commit -m "Add user authentication module"`
  - This describes `@` and creates new `@` in one command

**Success criteria:**
- Your changes are in a described commit (not the working copy)
- New empty working-copy commit exists
- `jj log` shows your commit with proper description

---

### View History and Status

Essential commands for understanding repository state.

**Task Progress:**
- [ ] View commit history: `jj log`
  - Shows graph, change IDs, bookmarks, descriptions
  - Working copy marked with `@`
  - Conflicts marked with `×`
- [ ] View specific commit: `jj show <change-id>`
- [ ] View working copy changes: `jj diff`
- [ ] View specific commit changes: `jj diff -r <change-id>`
- [ ] View file history: `jj log <file-path>`
- [ ] Check working copy status: `jj status`

**Success criteria:** You can understand what commits exist and what has changed.

---

### Push to Remote

Share your commits with a remote repository.

**Task Progress:**
- [ ] Ensure commits are described (not empty descriptions)
- [ ] Create bookmark at commit to push:
  ```bash
  jj bookmark create feature-name
  ```
- [ ] Verify what you're pushing: `jj show`
- [ ] Push the bookmark:
  ```bash
  jj git push --bookmark feature-name
  ```
- [ ] Verify push succeeded (check remote or look for success message)

**For GitHub PR workflow:**
- [ ] Create bookmark: `jj bookmark create my-feature`
- [ ] Push: `jj git push --bookmark my-feature`
- [ ] Create PR on GitHub using `my-feature` as the branch name

**Success criteria:**
- Bookmark exists on remote
- GitHub/GitLab shows your commits
- No error messages about conflicts or rejections

---

### Pull Updates from Remote

Fetch and integrate changes from remote repository.

**Task Progress:**
- [ ] Fetch latest changes:
  ```bash
  jj git fetch
  ```
- [ ] View what was fetched: `jj log`
  - Look for `main@origin` or other remote bookmarks
- [ ] Decide integration strategy:
  - **If continuing your work:** Rebase (see next step)
  - **If integrating parallel work:** Merge (see alternative)

**Option A: Rebase (recommended for simple updates)**
- [ ] Rebase your commits onto remote:
  ```bash
  jj rebase -s @ -d main@origin
  ```
- [ ] Verify: `jj log` shows your commits on top of remote

**Option B: Merge**
- [ ] Create merge commit:
  ```bash
  jj new main@origin @-
  ```
- [ ] Describe merge: `jj describe -m "Merge main into feature"`
- [ ] Verify: `jj log` shows merge commit with two parents

**Success criteria:**
- Your commits incorporate remote changes
- No unexpected conflicts
- `jj log` shows clean history

---

## Level 2: Intermediate Workflows

### Collaboration Workflow

Complete workflow for working with teammates.

**Task Progress:**
- [ ] Start work on new feature:
  ```bash
  jj new main@origin
  ```
- [ ] Make changes and commit:
  ```bash
  jj describe -m "Implement feature X"
  jj new
  ```
- [ ] Make more changes:
  ```bash
  jj describe -m "Add tests for feature X"
  ```
- [ ] Fetch updates from teammates:
  ```bash
  jj git fetch
  ```
- [ ] Check for divergence: `jj log`
- [ ] If `main@origin` moved, rebase:
  ```bash
  jj rebase -s <first-feature-commit> -d main@origin
  ```
- [ ] Create bookmark and push:
  ```bash
  jj bookmark create feature-x
  jj git push --bookmark feature-x
  ```
- [ ] Create PR on GitHub/GitLab
- [ ] After PR approval, update local:
  ```bash
  jj git fetch
  jj bookmark move main --to main@origin
  ```

**Success criteria:**
- Feature commits are on remote
- PR is created
- Teammates can see your work

---

### Fixing Mistakes in Recent Commits

Correct errors in commits you just made.

**Fix commit description:**
- [ ] Identify commit to fix: `jj log`
- [ ] Edit description:
  ```bash
  jj describe -r <change-id> -m "Corrected description"
  ```

**Fix commit content:**
- [ ] Make the correcting changes in working copy
- [ ] Squash into the commit needing fixes:
  ```bash
  jj squash --into <change-id>
  ```
- [ ] Verify: `jj show <change-id>` includes fixes

**Undo recent operation:**
- [ ] View operation log: `jj op log`
- [ ] Undo last operation: `jj undo`
- [ ] Or undo multiple: `jj undo -n 3`
- [ ] Verify: `jj log` shows previous state

**Success criteria:**
- Mistakes corrected
- History looks as intended
- No lost work

---

### Split Commits

Separate changes in a commit into multiple commits.

**Split working copy:**
- [ ] Ensure working copy has multiple logical changes
- [ ] Run interactive split:
  ```bash
  jj split
  ```
- [ ] In TUI, select changes for first commit (space to toggle)
- [ ] Press 'q' or follow prompts to complete
- [ ] Describe first commit: `jj describe -r @- -m "First part"`
- [ ] Describe second commit: `jj describe -m "Second part"`
- [ ] Verify: `jj log` shows two commits

**Split specific commit:**
- [ ] Identify commit: `jj log`
- [ ] Split by files:
  ```bash
  jj split -r <change-id> -i file1.txt file2.txt
  ```
  - Selected files go to first commit, rest to second
- [ ] Or split interactively:
  ```bash
  jj split -r <change-id>
  ```
- [ ] Describe both resulting commits
- [ ] Verify: `jj log` shows split

**Success criteria:**
- One commit becomes two or more
- Each commit has focused changes
- Descendants automatically rebased

---

### Squash Commits

Combine multiple commits into one.

**Squash working copy into parent:**
- [ ] Make changes in working copy
- [ ] Run: `jj squash`
- [ ] Verify: `jj log` shows changes merged into parent

**Squash specific commit into parent:**
- [ ] Identify commit: `jj log`
- [ ] Squash: `jj squash -r <change-id>`
- [ ] Verify: Changes merged into parent
- [ ] Note: Descendants automatically rebase

**Squash interactively (select changes):**
- [ ] Run: `jj squash -i`
- [ ] In TUI, select changes to squash
- [ ] Press 'q' to complete
- [ ] Verify: `jj diff` shows remaining changes

**Squash into specific commit:**
- [ ] Make changes
- [ ] Run: `jj squash --into <target-change-id>`
- [ ] Verify: Changes appear in target commit

**Success criteria:**
- Multiple commits combined
- History cleaner
- No lost changes

---

### Rebase Commits

Move commits to different parents.

**Rebase current commit onto new parent:**
- [ ] Identify new parent: `jj log`
- [ ] Rebase:
  ```bash
  jj rebase -d <destination-change-id>
  ```
- [ ] Verify: `jj log` shows commit moved

**Rebase entire subtree:**
- [ ] Identify source (root of subtree): `jj log`
- [ ] Rebase:
  ```bash
  jj rebase -s <source-change-id> -d <destination>
  ```
- [ ] Verify: Entire subtree moved

**Rebase single commit (leaving children):**
- [ ] Run:
  ```bash
  jj rebase -r <change-id> -d <destination>
  ```
- [ ] Children rebase onto commit's new location
- [ ] Verify: `jj log` shows topology change

**Success criteria:**
- Commit(s) have new parent(s)
- History structure matches intent
- No unexpected conflicts

---

### Conflict Resolution

Handle and resolve merge conflicts.

**Task Progress:**
- [ ] Identify conflict in log:
  ```bash
  jj log
  ```
  - Look for `×` marker next to conflicted commit
- [ ] Create working copy on conflicted commit:
  ```bash
  jj new <conflicted-change-id>
  ```
- [ ] Files now contain conflict markers - open in editor
- [ ] Understand 3-way conflict markers:
  ```
  <<<<<<< Conflict 1 of 1
  +++++++ Contents of side #1
  left side changes
  ------- Contents of base
  original content
  +++++++ Contents of side #2
  right side changes
  >>>>>>> Conflict 1 of 1 ends
  ```
- [ ] Edit files to resolve (remove markers, keep desired content)
- [ ] Review resolution: `jj diff`
- [ ] Squash resolution into conflicted commit:
  ```bash
  jj squash
  ```
- [ ] Verify conflict resolved: `jj log`
  - `×` marker should be gone

**Alternative - abandon and redo:**
- [ ] If conflict is too messy, abandon conflicted commit:
  ```bash
  jj abandon <conflicted-change-id>
  ```
- [ ] Redo the operation that caused conflict more carefully

**Success criteria:**
- No `×` markers in `jj log`
- Files have no conflict markers
- Changes from both sides are properly integrated

---

### Handle Divergent Bookmarks

Fix conflicted bookmarks (marked with `??`).

**Task Progress:**
- [ ] Identify conflicted bookmark:
  ```bash
  jj bookmark list
  ```
  - Look for `??` marker
- [ ] View commits involved:
  ```bash
  jj log
  ```
- [ ] Decide on resolution strategy:
  - **If one side is correct:** Move bookmark to it
  - **If both sides needed:** Create merge commit

**Option A: Move to correct commit**
- [ ] Choose correct commit from log
- [ ] Move bookmark:
  ```bash
  jj bookmark move <name> --to <correct-change-id>
  ```
- [ ] Verify: `jj bookmark list` no longer shows `??`

**Option B: Merge both sides**
- [ ] Create merge commit:
  ```bash
  jj new <local-commit> <remote-commit>
  ```
- [ ] Describe merge: `jj describe -m "Merge divergent changes"`
- [ ] Move bookmark to merge:
  ```bash
  jj bookmark move <name> --to @
  ```
- [ ] Verify: `jj bookmark list` shows no conflict

**Success criteria:**
- Bookmark no longer conflicted
- Can push bookmark: `jj git push --bookmark <name>`

---

## Level 3: Advanced Workflows

### Reorder Commits in History

Change the order of commits.

**Task Progress:**
- [ ] View current history: `jj log`
- [ ] Identify commit to move: note its change ID
- [ ] Identify new location for commit
- [ ] Rebase commit to new position:
  ```bash
  jj rebase -r <change-id-to-move> -d <new-parent-id>
  ```
- [ ] Note: Children of moved commit automatically rebase
- [ ] Verify: `jj log` shows new order
- [ ] Handle any conflicts that arise

**Success criteria:**
- Commits in desired order
- All commits still present
- Descendants properly rebased

---

### Make Commits Independent (Parallelize)

Convert a stack of commits into parallel branches.

**Task Progress:**
- [ ] Identify commit stack: `jj log`
- [ ] Run parallelize:
  ```bash
  jj parallelize <commit-range>
  ```
- [ ] Example: `jj parallelize @---::@` makes last 3 commits siblings
- [ ] Verify: `jj log` shows commits as siblings with same parent

**Use case:** Useful when commits don't actually depend on each other.

**Success criteria:**
- Commits are siblings instead of stack
- Can be merged/rebased independently

---

### Duplicate Commits

Create copy of a commit elsewhere.

**Task Progress:**
- [ ] Identify commit to duplicate: `jj log`
- [ ] Duplicate:
  ```bash
  jj duplicate <change-id>
  ```
- [ ] New commit created with same changes
- [ ] Rebase duplicate to desired location:
  ```bash
  jj rebase -r <new-duplicate-id> -d <destination>
  ```
- [ ] Verify: `jj log` shows both original and duplicate

**Use case:** Applying same changes to multiple branches.

**Success criteria:**
- Two commits with same changes exist
- Located where intended

---

### Work with Multiple Working Copies

Create and manage multiple working copies (workspaces).

**Create additional workspace:**
- [ ] From main repo:
  ```bash
  jj workspace add --name feature-workspace ../path/to/workspace
  ```
- [ ] Navigate to new workspace
- [ ] Verify separate working copy: `jj status`
- [ ] Work independently in each workspace

**List workspaces:**
- [ ] Run: `jj workspace list`
- [ ] Shows all workspaces and their working-copy commits

**Switch between workspaces:**
- [ ] Navigate to different workspace directory
- [ ] Each has independent working copy
- [ ] Share same operation log

**Use case:** Work on multiple features without stashing or committing incomplete work.

**Success criteria:**
- Multiple workspaces exist
- Can work in each independently
- Changes isolated per workspace

---

### Advanced Recovery

Recover from complex mistakes using operation log.

**View detailed operation history:**
- [ ] Run: `jj op log`
- [ ] Note operation IDs (e.g., `abc123`)
- [ ] See what each operation did

**Restore to specific operation:**
- [ ] Find desired operation ID
- [ ] Restore:
  ```bash
  jj op restore <operation-id>
  ```
- [ ] Entire repo state reverts to that point
- [ ] Verify: `jj log` shows history as it was

**View repo state at past operation:**
- [ ] Without changing current state:
  ```bash
  jj --at-op=<operation-id> log
  ```
- [ ] Useful for investigating what happened
- [ ] Current state unchanged

**Undo multiple operations:**
- [ ] Run: `jj undo -n 5` (undoes last 5 operations)
- [ ] Or repeatedly: `jj undo` (once per operation)
- [ ] Verify: `jj op log` shows undo operations

**Success criteria:**
- Recovered from mistake
- Repository in desired state
- Understand what went wrong

---

### Maintain Clean Remote Bookmarks

Keep remote bookmarks in sync.

**Task Progress:**
- [ ] Fetch regularly: `jj git fetch`
- [ ] List all bookmarks: `jj bookmark list -a`
- [ ] Track relevant remote bookmarks:
  ```bash
  jj bookmark track main@origin
  ```
- [ ] Move local bookmark to match remote:
  ```bash
  jj bookmark move main --to main@origin
  ```
- [ ] Delete obsolete local bookmarks:
  ```bash
  jj bookmark delete old-feature
  ```
- [ ] Verify: `jj bookmark list` shows clean state

**Success criteria:**
- Local bookmarks match intended remotes
- No stale bookmarks
- Clear which bookmarks track remotes

---

## Workflow Tips

### General Best Practices

- **Run `jj status` frequently** - Ensures working copy snapshotted
- **Use change IDs when rewriting** - They persist through rewrites
- **Create bookmarks only when pushing** - They're not required for local work
- **Fetch before starting new work** - Reduces conflicts later
- **Describe commits as you go** - Easier than doing it later
- **Review before pushing** - Use `jj show` to verify

### When to Use Which Workflow

**Beginner workflows:**
- Daily work, basic commits, simple pushes
- Use these until comfortable with concepts

**Intermediate workflows:**
- Team collaboration, fixing mistakes, organizing commits
- Use when working with others or cleaning history

**Advanced workflows:**
- Complex history manipulation, multiple features, recovery
- Use when you understand jj's model well

### Avoiding Common Mistakes

- **Don't undo after push** - Create forward fixes instead
- **Don't mix git and jj** - Stick to one tool
- **Don't forget to fetch** - Keeps you in sync
- **Don't panic on conflicts** - They're markers, not blockers

## Further Reading

- [SKILL.md](SKILL.md) - Main skill overview
- [GIT-COMPARISON.md](../jj-migrate/GIT-COMPARISON.md) - Git to jj translations
- [TROUBLESHOOTING.md](../jj-troubleshoot/TROUBLESHOOTING.md) - Fix common problems
- [REFERENCE.md](../jj-reference/REFERENCE.md) - Complete command reference
- [GLOSSARY.md](../jj-reference/GLOSSARY.md) - Term definitions
