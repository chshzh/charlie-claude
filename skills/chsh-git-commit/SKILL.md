---
name: chsh-git-commit
description: Use when the user asks to prepare commits, review changes for committing, commit all changes, or decide how to split work into multiple Git commits.
---

# Git Commit Workflow

## Critical Rules

1. **Inspect before committing**: Always run `git diff .` to review the full worktree before proposing or creating any commits.
2. **Plan first, commit second**: Present a commit plan (grouped files, rationale, suggested messages) and wait for explicit user approval before running `git commit`.
3. **Split by logical change**: Separate commits by concern — do not mix unrelated fixes, refactors, docs, formatting-only changes, or generated files unless the user explicitly requests it.
4. **No interactive commands**: Use only non-interactive Git commands.
5. **Never assume permission**: Treat any request to "commit changes" as permission to inspect and plan, not as permission to execute `git commit`.

## Workflow

### Step 1 — Inspect the worktree

```bash
git diff .
git status --short
git diff --cached   # only if staged changes exist
```

### Step 2 — Propose a commit plan

Present a table or list showing:
- **Group**: the set of files for each proposed commit
- **Rationale**: why these files belong together
- **Suggested commit message**: following Conventional Commits style (`type(scope): short description`)

Common types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `style`, `build`

### Step 3 — Wait for approval

After presenting the plan, use the `AskQuestion` tool to collect approval. Always offer these options:

```
Question: "Proceed with this commit plan?"
Options:
  - "Approve — commit as planned"
  - "Edit commit messages first"
  - "Re-group the commits"
  - "Cancel"
```

Do not run `git commit` until the user selects **Approve**. Treat any other selection as a request to revise the plan.

### Step 4 — Execute approved commits

For each approved commit, stage only the relevant files and commit:

```bash
git add <file1> <file2> ...
git commit -s -m "type(scope): description"
```

Use a HEREDOC for multi-line messages when needed:

```bash
git commit -s -m "$(cat <<'EOF'
type(scope): short summary

- detail line 1
- detail line 2
EOF
)"
```

## Grouping Guidelines

| Change type | Separate commit? |
|---|---|
| New feature code | Yes |
| Bug fix | Yes |
| Refactor (no behaviour change) | Yes |
| Config / Kconfig changes | Yes (or with related feature) |
| Docs / README | Yes (unless tightly coupled to a feature) |
| Formatting / whitespace only | Yes |
| Generated / auto-updated files | Yes |
| Board overlay + matching prj.conf | Can group together |
| Sysbuild files | Group by logical purpose |

## NCS-Specific Notes

- `prj.conf`, `boards/*.conf`, `overlay-*.conf` — group with the feature they enable
- `CMakeLists.txt` changes — separate commit unless trivially part of a feature add
- `west.yml` changes — always a separate commit with a clear rationale
- `sysbuild/` files — group by logical purpose (e.g. mcuboot config changes together)
- Generated partition manager files (`pm/`, `*.map`) — separate `chore` commit or omit if not meaningful

## Splitting Changes Within a Single File

When one file contains two or more logically distinct changes that belong in separate commits (e.g. a bug fix + a new policy section), split them without interactive staging:

1. Temporarily strip the *later* section from the file (Python or StrReplace)
2. Stage and commit the *first* change
3. Restore the stripped section
4. Stage and commit the *second* change

```python
MARKER = "\n## Self-Update Policy\n"
text = path.read_text()
path.write_text(text[:text.find(MARKER)])  # strip later section
# → git add + commit "fix: ..."
path.write_text(text)                      # restore full content
# → git add + commit "feat: ..."
```

**Why**: `git log -p <file>` then shows each logical change in its own commit — easier to bisect, review, or revert independently.

**Pitfall**: Always verify the marker falls *before* the function/block closing lines. If the marker is the last statement in a function, the stripped file will be missing `return` and `}`, causing `expected declaration or statement at end of input` errors.

## Self-Update Policy

At the **end of each conversation**, review what was discovered and check whether any facts in this skill are new, corrected, or outdated (e.g. new grouping patterns, NCS-specific file conventions, commit message conventions).

If updates are warranted:
1. Collect all proposed changes with a brief rationale for each.
2. Present a summary to the user and ask for approval using `AskQuestion`.
3. Apply approved updates to this file immediately.

Do **not** modify this skill mid-conversation unless the user explicitly asks.
