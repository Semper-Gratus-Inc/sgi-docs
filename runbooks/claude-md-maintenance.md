# CLAUDE.md Maintenance Runbook

## Purpose

This runbook documents the process for auditing, restructuring, and verifying a `CLAUDE.md` file in a Claude Code project. It covers extracting content into `@`-imported files, removing stale sections, and confirming the full context stack is clean.

Run this process any time the main `CLAUDE.md` feels bloated, has suspected stale content, or needs restructuring before a new phase of work.

---

## Trigger Conditions

Run this process when any of the following are true:

- `CLAUDE.md` has grown large enough that it's hard to scan at a glance
- You suspect sections are stale (old phases, completed PRs, outdated branch names)
- Starting a new phase and want the context stack clean before beginning
- Setting up a new machine and verifying the full context loads correctly
- After a long sprint where multiple memory entries accumulated without review

---

## Prerequisites

Before touching anything:

1. Confirm the repo has no uncommitted changes:
   ```bash
   git status
   ```
2. Confirm the current branch is pushed and clean — if there are uncommitted changes, commit or stash them first. Do not restructure `CLAUDE.md` on a dirty working tree.
3. Note which branch you are on. Run this process on `develop` or a dedicated housekeeping branch, not directly on `main`.

---

## Step 1 — Audit: Read, Analyze, Propose

Ask Claude Code to read the current `CLAUDE.md` and all imported files, identify what is stale or extractable, and propose a new structure. **No changes are made in this step.**

Prompt:
```
Read the current CLAUDE.md and all files it imports. Then:
1. Identify any content that is stale (completed work, old branch names, resolved issues)
2. Identify sections that belong in a separate @ imported file rather than the main file
3. Propose a restructured layout — what stays in CLAUDE.md, what moves where, what gets deleted
Do not make any changes yet. Show me the proposal first.
```

What to look for in the proposal:
- Stale phase references, closed PRs, or resolved bugs still described as active
- Large reference blocks (AWS IDs, GitHub repo list, server IPs) that bloat the main file and should live in `@`-imported files
- Load-bearing rules that must stay in the main file — branching rules, secrets policy, the graphify session-start requirement
- Anything that would break silently if moved (see Step 3)

---

## Step 2 — Verify @ Import Syntax Support

Before assuming `@`-import syntax works, verify it against the live Claude Code documentation. Do not rely on memory or assumptions — the feature may have changed.

Prompt:
```
Fetch https://docs.anthropic.com/en/docs/claude-code/memory and confirm whether @ import syntax is supported in CLAUDE.md. Show me the relevant section.
```

Confirm:
- The `@path/to/file.md` syntax is supported and actively maintained
- The paths are relative to the `CLAUDE.md` file location
- There are no known limitations (file size, nesting depth, etc.) that affect your use case

Do not proceed to implementation until this is confirmed from the live docs.

---

## Step 3 — Review the Proposal

Before approving, read the proposed structure carefully and check for:

**Valid @ paths**
- Every path in an `@` directive must be correct relative to where `CLAUDE.md` lives
- If a path is wrong, the import silently fails — Claude receives no error, it just doesn't load the file
- Verify each proposed path actually exists or will exist after the restructure

**Load-bearing rules staying in the main file**
- The graphify session-start block (`graphify update .`) must stay in `CLAUDE.md` — it is a session-start directive, not reference material
- Branching rules and secrets policy must stay in `CLAUDE.md` — they are behavioral rules Claude needs every session
- Any `@` import instructions for machine setup or infrastructure are reference material and safe to extract

**Silent failure risks**
- Graphify hooks are especially important: the runbook note in `CLAUDE.md` warns that without them, Claude's PreToolUse guardrails silently block Read and Bash operations. This warning must remain visible in the main file, not buried in an imported file.
- Test every proposed `@` import path before approving — a typo in a path produces no error

---

## Step 4 — Implement with Approval Gate

Once the proposal looks correct, ask Claude to implement it — but only after you have explicitly approved the structure.

Prompt:
```
The proposal looks good. Implement it now:
1. Create each new @ import file with the content we agreed on
2. Update CLAUDE.md to use @ imports and remove the extracted content
3. Show me each file change as you make it — do not batch silently
```

As Claude makes changes, verify each one before it moves to the next:
- Confirm the new file was created at the correct path
- Confirm the content matches what was proposed
- Confirm the `@` directive in `CLAUDE.md` points to the right relative path

If anything looks wrong, stop and correct it before continuing.

---

## Step 5 — Verify Session Load

After restructuring, start a fresh Claude Code session (or use `/clear`) and ask Claude to confirm that all imports resolved correctly.

Prompt:
```
What files were loaded into your context at session start? List every CLAUDE.md and imported file you can see, including the contents of any @ imports that were resolved.
```

Expected output:
- The main `CLAUDE.md`
- Each `@`-imported file listed by its full path with its contents visible
- No missing or empty imports

If any imported file is missing from the output, the `@` path is wrong. Fix it before proceeding.

---

## Step 6 — Context Health Check

After verifying the session load, run the context health check to audit all project memory files. This is the official verification step after any `CLAUDE.md` restructure — not optional cleanup.

Type in Claude Code:
```
audit my context
```

This invokes the `context-health-check` skill, which:
1. Reads all memory files for the current project
2. Cross-checks each entry against the current `CLAUDE.md` and its imports
3. Checks any PR or issue references via `gh pr view` / `gh issue view`
4. Produces a status table (CURRENT / STALE / MERGED / CLOSED / CONFLICTS / BROKEN)
5. Proposes specific fixes with evidence

**Do not consider the restructure complete until all flagged items are resolved.** Common findings after a restructure:
- Memory entries that reference the old `CLAUDE.md` structure (now superseded)
- Branch name discrepancies between memory and the updated `CLAUDE.md`
- Merged PRs still tracked as open in memory files

For each proposed fix the audit surfaces, review the evidence and apply or dismiss explicitly. Do not leave items in an unresolved state.

---

## Step 7 — Commit

Once the session load is verified and the context health check is clean, commit all changed files together.

```bash
git add CLAUDE.md .claude/
git status   # confirm only the expected files are staged
git commit -m "docs: restructure CLAUDE.md — extract reference sections to @ imports, remove stale content"
git push
```

Commit message conventions:
- `docs:` prefix for CLAUDE.md and runbook changes
- One sentence describing what moved and why
- Do not reference Claude or this runbook in the commit message — the commit is about the project, not the process

---

## Notes

- This entire process was first run on 2026-06-19 for the Semper Gratus `sgi-app` project
- The context health check in Step 6 was created as a direct result of that session — it replaced a manual audit that had to be done ad hoc
- The `@` import verification step (Step 2) exists because import syntax was confirmed live against docs rather than assumed from prior knowledge
