---
name: create-changeset
description: Create a changeset markdown file for a monorepo using @changesets/cli. Use when the user asks to create a changeset, add a changeset, run pnpm changeset, or mentions "체인지셋"/"changeset 생성". Detects affected packages from git diff and writes the file directly so the user does not have to drive the interactive CLI.
---

# Create Changeset

## Quick start

1. Detect changed packages (git diff against `main` + uncommitted)
2. Read `.changeset/config.json` for fixed groups, ignore list, baseBranch
3. Peek at 2 recent `.changeset/*.md` files to mirror style (language, bump level convention)
4. Write `.changeset/<random-3-word-slug>.md` directly via the Write tool
5. Print the file path; do NOT auto-commit (config typically has `commit: false`)

## File format

```
---
"package-a": minor
"package-b": patch
---

one short sentence describing the change (match repo language)
```

Bump levels: `major` (breaking), `minor` (feature/visible change), `patch` (small fix). If recent changesets in the repo use `minor` for fixes, follow that convention rather than picking by semver theory.

## Workflow

1. **Find changed files**
   - `git diff <baseBranch>...HEAD --name-only`
   - `git diff --name-only` (uncommitted)
   - `git diff --name-only --cached` (staged)
2. **Map files → packages**
   - Walk up to nearest `package.json` and read its `name` field
   - Or pattern-match `apps/<name>/` and `packages/<name>/` against the workspace
3. **Apply config rules** from `.changeset/config.json`
   - `ignore`: drop these packages
   - `fixed`: if any member of a fixed group is changed, list ALL members of that group with the same bump level
   - `linked`: same bump level across linked packages
   - `privatePackages.version === false`: drop private packages
4. **Mirror existing style** — read 2 recent changesets (sorted by mtime); match their language and tone
5. **Generate slug** — 3 random words joined by hyphens (e.g., `silver-dolphins-walk`); ensure no collision in `.changeset/`
6. **Write the file** with `Write` tool. Do not run `pnpm changeset` (interactive prompt cannot be driven from here)
7. **Report** the path back to the user; let them stage/commit themselves

## Edge cases

- Only ignored/private packages changed → tell the user no changeset is needed
- No package changes (e.g., docs-only at repo root) → ask whether to add a changeset for the root or skip
- Multiple unrelated changes in one branch → suggest splitting into multiple changeset files (one per logical change), each with its own description
- Fresh branch with no upstream — fall back to `git diff main...HEAD` or whatever `baseBranch` says

## Reference

- Official docs: https://github.com/changesets/changesets/tree/main/docs
- Adding a changeset: https://github.com/changesets/changesets/blob/main/docs/adding-a-changeset.md
- Common questions: https://github.com/changesets/changesets/blob/main/docs/common-questions.md
