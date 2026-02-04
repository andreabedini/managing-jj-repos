# Feature Branch Workflow Example

Real-world scenario for creating and pushing a feature branch with jj.

## Scenario

You need to implement a new authentication feature, create multiple commits, and push to GitHub for a pull request.

## Step-by-Step

### 1. Start Fresh
```bash
# Make sure you're synced with main
jj git fetch
jj new main@origin
```

**Result:** Working copy (`@`) is now a child of remote main.

### 2. Implement First Part
```bash
# Edit files: add auth module structure
vim src/auth/mod.rs
vim src/auth/types.rs

# Check changes
jj diff

# Describe the commit
jj describe -m "Add authentication module structure"

# Create new commit for next changes
jj new
```

**Result:** First commit is finalized, new empty working copy created.

### 3. Add Tests
```bash
# Edit files: add tests
vim src/auth/tests.rs

# Describe and create new commit
jj describe -m "Add authentication tests"
jj new
```

**Result:** Second commit created.

### 4. Add Integration
```bash
# Edit files: integrate auth into app
vim src/main.rs
vim src/config.rs

# Describe final commit
jj describe -m "Integrate authentication into app"
```

**Result:** Three commits ready to push.

### 5. Review History
```bash
jj log -n 5
```

You should see:
```
@  working-copy-id (empty) "working copy"
○  integration-id "Integrate authentication into app"
○  tests-id "Add authentication tests"
○  structure-id "Add authentication module structure"
○  main@origin
```

### 6. Create Bookmark and Push
```bash
# Create bookmark on current commit (parent of @)
jj bookmark create auth-feature -r @-

# Verify bookmark created
jj bookmark list

# Push to remote
jj git push --bookmark auth-feature
```

**Result:** All three commits pushed to `auth-feature` branch on GitHub.

### 7. Create Pull Request
Go to GitHub and create PR from `auth-feature` to `main`.

## Common Variations

### Realized You Made a Mistake
```bash
# Before pushing, squash a fix into earlier commit
jj new <commit-with-mistake>
# make fixes
jj squash  # Squashes into parent (the commit with mistake)
```

### Want to Reorder Commits
```bash
# Move tests commit to come after integration
jj rebase -r <tests-id> -d <integration-id>
```

### Want to Split a Commit
```bash
# Split the integration commit into two
jj split -r <integration-id>
# In TUI: select some changes for first commit
```

## Key Takeaways

- No branch creation needed upfront - just work
- Bookmarks created only when ready to push
- Easy to refine history before pushing
- Multiple commits pushed in one operation
