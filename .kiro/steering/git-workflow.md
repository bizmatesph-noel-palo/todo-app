# Git Workflow

## Task Execution Flow

Each task (spec task, feature, or fix) follows this cycle with human oversight at every gate:

### 1. Start Task
- Announce: "Starting task BTDA-XXX"
- Determine base branch:
  - Default: branch from `development`
  - If task depends on an unmerged task: branch from that task's branch
- Ask: "I'd like to create branch `{type}/BTDA-XXX-description`. Should I go ahead?"
- Wait for approval before creating the branch

### 2. Code
- Implement the changes
- Follow steering files (coding standards, architecture patterns)
- Stop when implementation is complete

### 3. Review Gate
- Show the diff or describe the changes
- Wait for human to review
- If human requests changes → make adjustments, repeat step 3
- If human approves → proceed to step 4

### 4. Commit Gate
- Ask: "Ready to commit?"
- Wait for explicit approval
- Commit with message: `type(BTDA-XXX): description`
- Push to remote

### 5. PR Gate
- Ask: "Ready for PR?"
- Wait for explicit approval
- Provide the `make pr` command:
  ```bash
  make pr title="BTDA-XXX - Description" body="type(BTDA-XXX): summary of changes"
  ```
- Human runs the command (Kiro cannot execute WSL commands reliably)

### 6. Merge (Human Only)
- Human reviews PR on GitHub
- Human merges (squash merge to development)
- Human deletes the feature branch on GitHub

### 7. Next Task
- Cycle repeats from step 1

## Branch Decisions

| Scenario | Base Branch |
|----------|-------------|
| New independent task | `development` |
| Task depends on unmerged work | The dependency branch |
| Hotfix | `master` (rare, ask first) |

## Commands Reference

```bash
# Create branch
make branch name="{type}/BTDA-XXX-description"

# Commit and push
make commit msg="type(BTDA-XXX): description"

# Create PR
make pr title="BTDA-XXX - Description" body="type(BTDA-XXX): summary"
```

## Rules

- **Never** commit without explicit human approval
- **Never** push without explicit human approval
- **Never** create a PR without explicit human approval
- **Never** merge — that's always the human's job
- **Always** show changes before asking to commit
- **Always** use conventional commit format
- **Always** PR to `development` (never directly to `master`)
