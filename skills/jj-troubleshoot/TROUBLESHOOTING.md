# Jujutsu Troubleshooting Guide

Solutions to common problems, error messages, and unexpected behaviors in jj.

## Common Error Messages

### "Bookmark is conflicted"

**Error:**
```
Error: Refused to push a conflicted bookmark 'main'
Hint: Run `jj bookmark list` to see bookmarks.
```

**Cause:** Local and remote bookmarks have diverged, and jj cannot automatically resolve which commit the bookmark should point to.

**Solution:**

1. List bookmarks to see conflict:
   ```bash
   jj bookmark list
   ```
   Look for `??` marker next to bookmark name.

2. View commits involved:
   ```bash
   jj log
   ```
   Both commits sharing the bookmark will be visible.

3. Choose resolution:
   - **If one commit is correct:**
     ```bash
     jj bookmark move main --to <correct-change-id>
     ```
   - **If both needed, create merge:**
     ```bash
     jj new <local-commit> <remote-commit>
     jj describe -m "Merge divergent main"
     jj bookmark move main --to @
     ```

4. Verify resolution:
   ```bash
   jj bookmark list
   ```
   No more `??` marker.

5. Push:
   ```bash
   jj git push --bookmark main
   ```

**Prevention:** Fetch regularly and rebase/merge promptly.

---

### "Divergent changes"

**Warning:**
```
Added 1 commits, modified 1 commits, removed 1 commits
Added working copy commit as mzvwutvl e5251d63 (divergent) (empty)
```

**Cause:** Two visible commits share the same change ID. Usually from concurrent operations or complex rebasing.

**Understanding:** This is often harmless. It happens when:
- You edit a commit while also rebasing it
- Multiple workspaces modify the same change
- Race conditions in concurrent operations

**Solutions:**

**Option 1: Squash to merge (if both have useful changes)**
```bash
jj squash -r <one-divergent-id> --into <other-divergent-id>
```

**Option 2: Abandon one (if duplicate)**
```bash
jj abandon <unwanted-divergent-id>
```

**Option 3: Continue working (often fine)**
- If one is empty or not needed, just continue
- jj will handle it automatically in many cases

**Verification:**
```bash
jj log
```
Should show resolved state.

**Prevention:** Generally unavoidable and harmless. Don't worry unless it causes actual problems.

---

### "Stale working copy"

**Error:**
```
Error: The working copy is stale (not updated since operation abc123)
Hint: Run `jj workspace update-stale` to update it
```

**Cause:** Working copy hasn't been updated to reflect recent operations. Can happen after operations that don't automatically update it.

**Solution:**

1. Update working copy:
   ```bash
   jj workspace update-stale
   ```

2. Or trigger snapshot with:
   ```bash
   jj status
   ```

3. Retry your command.

**Prevention:** Run `jj status` regularly to keep working copy current.

---

### "Non-fast-forward push rejected"

**Error:**
```
Error: Refusing to push since the remote's main has moved forward
Hint: Try fetching and rebasing your changes
```

**Cause:** Remote has commits not present locally. Git won't let you overwrite remote history.

**Solution:**

1. Fetch updates:
   ```bash
   jj git fetch
   ```

2. View divergence:
   ```bash
   jj log
   ```
   Look at `main` vs `main@origin`.

3. Rebase your changes:
   ```bash
   jj rebase -s <your-first-commit> -d main@origin
   ```

4. Or create merge:
   ```bash
   jj new main@origin main
   jj describe -m "Merge remote main"
   jj bookmark move main --to @
   ```

5. Push again:
   ```bash
   jj git push --bookmark main
   ```

**Prevention:** Fetch before pushing, especially in collaborative projects.

---

### "No such revision"

**Error:**
```
Error: Revision "<change-id>" doesn't exist
```

**Cause:** The specified change ID, bookmark, or commit hash doesn't exist or was mistyped.

**Solutions:**

1. Check spelling of change ID/bookmark
2. View available commits:
   ```bash
   jj log
   ```
3. View bookmarks:
   ```bash
   jj bookmark list
   ```
4. If commit was deleted/abandoned, check operation log:
   ```bash
   jj op log
   jj undo  # If recent mistake
   ```

**Prevention:** Copy change IDs from `jj log` output rather than typing manually.

---

### "Merge conflict"

**Not really an error!** In jj, conflicts are first-class and don't block operations.

**Indicator:**
```
jj log
```
Shows `×` marker next to conflicted commit.

**Solution:** See "Resolving Conflicts" section below.

---

## Common Problems

### Problem: Accidentally Undid After Push

**Situation:** Ran `jj undo` after `jj git push`. Now remote is ahead of local.

**Why it's bad:** Remote still has commits you've undone locally. Creates inconsistent state.

**Solution:**

**Option 1: Redo (if just did undo)**
```bash
jj redo
```
Restores the pushed state.

**Option 2: Re-push (if redo not available)**
```bash
# Recreate state that was pushed
jj bookmark move <name> --to <pushed-commit>
# May need force push
jj git push --bookmark <name> --force
```

**Option 3: Fetch and reconcile**
```bash
jj git fetch
# Remote bookmark will diverge from local
jj bookmark move <name> --to <name>@origin
```

**Prevention:** Don't undo after push operations. Create forward-fixing commits instead.

---

### Problem: Mixed Git and jj Commands

**Situation:** Used both `git` and `jj` commands in colocated repository. Now bookmarks are conflicted or state is weird.

**Cause:** Both tools track state independently. Interleaving causes them to diverge.

**Solution:**

1. Import Git state to jj:
   ```bash
   jj git import
   ```

2. Check for conflicts:
   ```bash
   jj bookmark list
   jj log
   ```

3. Resolve any conflicts (see "Bookmark is conflicted" above)

4. Moving forward, use only jj commands

**Prevention:** Stick to one tool at a time. If using colocated repo, prefer jj exclusively.

---

### Problem: Can't Find Recent Changes

**Situation:** Made changes, but they don't appear in log or diff shows nothing.

**Likely causes and solutions:**

**Cause 1: Changes not snapshotted**
```bash
jj status  # Triggers snapshot
jj diff    # Now shows changes
```

**Cause 2: Looking at wrong commit**
```bash
jj log     # Find working copy (marked @)
jj diff    # Shows changes in @
```

**Cause 3: Changes were committed**
```bash
jj log     # Your changes are in @-
jj show @- # View the commit
```

**Cause 4: Changes were squashed**
```bash
jj log
jj show <parent-commit>  # Changes may be here
```

**Prevention:** Run `jj status` and `jj log` regularly to track state.

---

### Problem: Bookmark Not Moving with Commits

**Situation:** Making commits but bookmark stays on old commit.

**Explanation:** This is expected behavior! Unlike Git branches, jj bookmarks don't auto-advance.

**Solution:**

**If you want bookmark to move:**
```bash
jj bookmark move <name> --to @
```

**Better pattern:**
- Work without bookmarks for local development
- Create bookmarks only when ready to push
- Or move bookmarks manually when needed

**Prevention:** Understand bookmarks are optional pointers, not required for work.

---

### Problem: Too Many Commits in Log

**Situation:** `jj log` shows overwhelming number of commits.

**Solutions:**

**Show fewer commits:**
```bash
jj log -r ::@     # Only ancestors of working copy
jj log -r @::     # Only descendants of working copy
jj log -r @-::@   # Just last two commits
jj log -n 10      # Limit to 10 commits
```

**Show specific branches:**
```bash
jj log -r main::@         # Between main and @
jj log -r 'description(bug)'  # Commits mentioning "bug"
```

**Configure default:**
```bash
jj config set --user revsets.log '::@ | @::'
```
Now `jj log` shows only ancestors and descendants of working copy.

**Prevention:** Use revsets to filter log output. See [REFERENCE.md](REFERENCE.md) for revset syntax.

---

### Problem: Lost Commits

**Situation:** Commits seem to have disappeared.

**Solution using operation log:**

1. View operation history:
   ```bash
   jj op log
   ```

2. Find operation before commits disappeared

3. View repo at that operation:
   ```bash
   jj --at-op=<operation-id> log
   ```

4. If commits are there, restore:
   ```bash
   jj op restore <operation-id>
   ```

5. Or undo recent operations:
   ```bash
   jj undo
   # Repeat if needed
   ```

**Prevention:** Operation log makes it very hard to lose work. Don't panic - your work is almost certainly recoverable.

---

### Problem: File Deletion Not Tracked

**Situation:** Deleted files outside jj (e.g., `rm file.txt`). Running `jj diff` doesn't show deletion.

**Cause:** Working copy wasn't snapshotted after deletion.

**Solution:**

1. Trigger snapshot:
   ```bash
   jj status
   ```

2. Now deletion appears:
   ```bash
   jj diff
   ```

**Prevention:** Run `jj status` after manual file operations, or use jj for file operations when possible.

---

## Resolving Conflicts

### Identifying Conflicts

**In log output:**
```bash
jj log
```
Look for `×` marker next to commit.

**In status output:**
```bash
jj status
```
May show "conflict" markers if working copy is conflicted.

---

### Understanding Conflict Markers

jj uses 3-way conflict markers:

```
<<<<<<< Conflict 1 of 1
+++++++ Contents of side #1
changes from first parent
------- Contents of base
original content
+++++++ Contents of side #2
changes from second parent
>>>>>>> Conflict 1 of 1 ends
```

**Parts:**
- **Side #1:** Changes from one parent (e.g., your work)
- **Base:** Original content before changes
- **Side #2:** Changes from other parent (e.g., remote work)

**Resolution:** Edit file to remove markers and keep desired content.

---

### Step-by-Step Resolution

1. **Create working copy on conflict:**
   ```bash
   jj new <conflicted-change-id>
   ```

2. **Edit conflicted files:**
   - Open files in editor
   - Remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
   - Keep desired content from sides and/or base
   - Save files

3. **Review resolution:**
   ```bash
   jj diff
   ```

4. **Squash resolution into conflicted commit:**
   ```bash
   jj squash
   ```

5. **Verify conflict resolved:**
   ```bash
   jj log
   ```
   The `×` marker should be gone.

---

### Alternative: Abandon and Redo

If conflict is too complex:

```bash
# Abandon conflicted commit
jj abandon <conflicted-change-id>

# Redo operation more carefully
# E.g., rebase one commit at a time
```

---

### Conflicts in Rebases

**Situation:** Rebase creates conflicts.

**Solution:**

1. Rebase proceeds anyway (conflicts marked with `×`)
2. Resolve each conflicted commit:
   ```bash
   jj new <conflicted-id>
   # resolve
   jj squash
   ```
3. Continue with dependent commits (may need resolution too)

**Or abandon and try different approach:**
```bash
jj undo  # Undo rebase
# Try different base or strategy
```

---

## Performance Issues

### Slow `jj status` or `jj diff`

**Cause:** Large working directory, many files, or slow filesystem.

**Solutions:**

**Check what's slow:**
```bash
time jj status
```

**Exclude paths from tracking:**
Configure `.gitignore` to exclude large directories or build artifacts.

**Use colocated repo:**
If using separate Git repo, try `--colocate`:
```bash
jj git init --colocate
```

**Check for filesystem issues:**
- Slow network drives
- Antivirus scanning
- Large files in working copy

---

### Large Operation Log

**Situation:** `jj op log` shows many operations, commands slow down.

**Solution:**

Currently, jj doesn't auto-prune operation log. Manual pruning:

```bash
# This doesn't exist yet, but in future versions:
# jj op gc
```

**Workaround:** Operation log size rarely causes issues in practice. If it does, report as bug.

---

## Integration Issues

### GitHub Authentication Fails

**Error during push:**
```
Error: Authentication failed
```

**Solution:**

1. Ensure Git credentials are configured:
   ```bash
   git config --global credential.helper store
   # Or use SSH keys
   ```

2. Test Git push directly:
   ```bash
   cd .git  # If colocated
   git push
   # Fix any credential issues
   ```

3. jj uses Git's credential system, so fixing Git auth fixes jj auth

---

### Colocated Repository Confusion

**Situation:** Have both `.git` and `.jj` directories, unsure which tool to use.

**Guidance:**

- **Prefer jj commands** in colocated repos
- **If using git command:** Run `jj git import` afterward
- **Check state:** `jj status` and `jj log` show jj's view

**If too confusing:**
- Move to non-colocated: Delete `.git`, use `jj git export` when needed
- Or move to Git-only: Delete `.jj`, use Git exclusively

---

## Recovery Scenarios

### Completely Lost - Start Over

**Last resort if totally confused:**

1. **Save current state:**
   ```bash
   jj op log > ~/jj-op-log-backup.txt
   jj log > ~/jj-log-backup.txt
   ```

2. **View operation history to find good state:**
   ```bash
   jj op log
   ```

3. **Restore to known good operation:**
   ```bash
   jj op restore <good-operation-id>
   ```

4. **Or start fresh (DESTRUCTIVE):**
   ```bash
   # Backup first!
   cp -r .jj .jj.backup
   # Reinitialize
   rm -rf .jj
   jj git init
   ```

---

### Accidentally Force Pushed Wrong Thing

**Situation:** Force pushed to remote, overwrote important work.

**Solutions:**

**If remote is GitHub/GitLab:**
1. Check repository history/events for previous commit hashes
2. Restore from remote's reflog (if available)
3. Contact repository admin for recovery

**If you have local operation log:**
1. Find operation before bad push:
   ```bash
   jj op log
   ```
2. Restore:
   ```bash
   jj op restore <operation-id>
   ```
3. Verify state:
   ```bash
   jj log
   ```
4. Re-push correctly:
   ```bash
   jj git push --bookmark <name> --force
   ```

**Prevention:** Always verify before force push:
```bash
jj show  # Review what you're pushing
```

---

## Getting Help

### Diagnostic Commands

When reporting issues or seeking help, provide:

```bash
jj --version                    # jj version
jj config list                  # Configuration
jj status                       # Current state
jj log -n 20                    # Recent history
jj op log -n 10                 # Recent operations
jj bookmark list                # Bookmarks
```

### Further Resources

- **Official docs:** https://martinvonz.github.io/jj/
- **GitHub issues:** https://github.com/martinvonz/jj/issues
- **Discord:** Check jj repository for invite link

### Related Skill Files

- [SKILL.md](SKILL.md) - Main overview
- [WORKFLOWS.md](WORKFLOWS.md) - Step-by-step guides
- [GIT-COMPARISON.md](GIT-COMPARISON.md) - Git to jj translation
- [REFERENCE.md](REFERENCE.md) - Command reference
- [GLOSSARY.md](GLOSSARY.md) - Terminology
