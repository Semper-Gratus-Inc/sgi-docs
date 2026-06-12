# Engineering Standards

## General

- All code is reviewed before merging to main (via superpowers code review)
- No direct commits to main — all work on feature branches
- Commit messages describe WHY, not what
- No dead code, commented-out blocks, or TODO comments left in merged PRs

## C# / .NET API

- Target .NET 10, minimal API style (Program.cs top-level statements)
- EF Core for all database access — no raw SQL except for migrations
- Secrets never in source code or appsettings.json — use Secrets Manager (prod) or appsettings.Development.json (local, gitignored)
- Use `NpgsqlConnectionStringBuilder` for connection strings — never string interpolation
- Environment split: `IsDevelopment()` for local, Secrets Manager for everything else
- Controllers for all API endpoints — no inline route handlers beyond /health
- Async all the way down — no `.Result` or `.Wait()`

## Next.js / Frontend

- CSS variables for all colors and spacing — no hardcoded Tailwind color classes
- Mobile responsive required on all pages
- No inline styles except where CSS variables are needed via `style={{}}`
- ScrollReveal on all above-the-fold sections
- Static export only (`output: 'export'`) — no server-side rendering

## Git Workflow

### Branch Structure

```
main                          ← production, protected — PRs only, Phillip approval required
develop                       ← CI/CD deploys to dev EC2 on every push
feature/phase-N-name          ← one per phase, branched off develop
feature/phase-N-story-N-desc  ← one per story, branched off its phase branch
fix/short-description         ← hotfixes, branched off main
```

### Rules

- **Never commit directly to `main` or `develop`** — all changes via PR
- Every story gets its own branch off the current phase feature branch
- Story branch → PR → phase feature branch (Phillip approval required)
- Phase feature branch → PR → `main` when all stories are complete (Phillip approval required)
- `main` → triggers prod deploy with manual approval gate in GitHub
- `develop` → triggered by merging phase feature branch in for dev testing before merging to main

### Branch Naming

Branch names map directly to GitHub issue titles. Use the `SD-` prefix + GitHub issue number + the issue title (kebab-cased).

| Type | Pattern | Example |
|---|---|---|
| Phase feature | `feature/phase-N-name` | `feature/phase-1-foundation` |
| Story | `story/SD-[#]-[issue-title]` | `story/SD-4-member-portal` |
| Bug | `bug/SD-[#]-[issue-title]` | `bug/SD-17-invoice-decimal-mismatch` |
| Hotfix (prod) | `fix/SD-[#]-[issue-title]` | `fix/SD-22-payment-webhook-timeout` |

**Rules:**
- `SD` stands for Semper Dev — use it on every branch, no exceptions
- `[#]` is the GitHub issue number
- `[issue-title]` is the GitHub issue title, lowercased and hyphenated — do not rename or paraphrase it
- Branch names should be recognizable from the issue list at a glance

### PR Process

1. **Start a story:** branch off the current phase branch
   ```bash
   git checkout feature/phase-1-foundation
   git checkout -b feature/phase-1-story-3-member-portal
   ```
2. **Finish a story:** open PR to phase feature branch, request Phillip's approval
3. **Finish a phase:** open PR from phase feature branch → `main`, request Phillip's approval
4. PRs require a passing build (GitHub Actions) before approval is requested
5. Code review (`/superpowers:requesting-code-review`) before opening any PR

### Current Phase Branches

| Branch | Phase | Status |
|---|---|---|
| `feature/phase-1-foundation` | Phase 1 — Foundation | Active |

## Testing

- /health endpoint required on all APIs
- End-to-end test any new form or payment flow before closing a story
- Unit tests for business logic; integration tests for DB access
