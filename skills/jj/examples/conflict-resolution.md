# Conflict Resolution Walkthrough

Real-world scenario for resolving merge conflicts in jj.

## Scenario

You're working on a feature branch while a teammate merged changes to the same files in `main`. When you rebase, you get conflicts.

## Step-by-Step

### 1. Starting State
```bash
jj log -n 5
```

You see:
```
@  your-work-id "Add new API endpoint"
○  old-main-id "Previous main"
│ ○  main@origin "Update API structure"  # Teammate's work
├─╯
○  common-ancestor-id
```

### 2. Attempt Rebase
```bash
jj rebase -s @ -d main@origin
```

**Result:** Rebase completes, but conflicts are marked.

### 3. Check for Conflicts
```bash
jj log -n 3
```

You see:
```
@  your-work-id (empty) "working copy"
×  your-work-rebased-id "Add new API endpoint"  # ← × means conflict!
○  main@origin "Update API structure"
```

The `×` marker indicates a conflicted commit.

### 4. Inspect the Conflict
```bash
jj show <your-work-rebased-id>
```

Shows which files are conflicted.

### 5. Create Working Copy on Conflict
```bash
jj new <your-work-rebased-id>
```

**Result:** Working copy moved to the conflicted commit. Files now contain conflict markers.

### 6. View Conflict Markers
```bash
cat src/api/endpoints.rs
```

You see:
```rust
<<<<<<< Conflict 1 of 1
+++++++ Contents of side #1 (your changes)
pub fn create_endpoint(config: Config) -> Endpoint {
    Endpoint::new(config.url, config.auth_token)
}
------- Contents of base
pub fn create_endpoint(url: String) -> Endpoint {
    Endpoint::new(url)
}
+++++++ Contents of side #2 (teammate's changes)
pub fn create_endpoint(config: &ApiConfig) -> Endpoint {
    Endpoint::from_config(config)
}
>>>>>>> Conflict 1 of 1 ends
```

### 7. Resolve the Conflict
Edit the file to keep the best of both changes:

```rust
pub fn create_endpoint(config: &ApiConfig) -> Endpoint {
    Endpoint::new(config.url, config.auth_token)
}
```

Save the file.

### 8. Review Resolution
```bash
jj diff
```

Shows your resolution changes.

### 9. Squash Resolution into Conflicted Commit
```bash
jj squash
```

**Result:** Your resolution is merged into the conflicted commit.

### 10. Verify Conflict Resolved
```bash
jj log -n 3
```

Now see:
```
@  new-empty-id (empty) "working copy"
○  your-work-rebased-id "Add new API endpoint"  # ← × is gone!
○  main@origin "Update API structure"
```

No more `×` marker - conflict is resolved!

### 11. Continue Working or Push
```bash
# If done, create bookmark and push
jj bookmark move feature --to @-
jj git push --bookmark feature
```

## Alternative: Abandon and Redo

If conflict is too complex:
```bash
# Abandon the conflicted commit
jj abandon <your-work-rebased-id>

# Go back to before rebase
jj undo

# Try different approach (merge instead of rebase)
jj new main@origin @-
jj describe -m "Merge main into feature"
```

## Key Takeaways

- Conflicts marked with `×` in log, not blockers
- Can continue other work while conflicts exist
- Resolution workflow: new → edit → squash
- Operation log lets you undo and try again
- Conflicts less scary in jj than git
