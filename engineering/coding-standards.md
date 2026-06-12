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

- Branch naming: `feature/short-description`, `fix/short-description`
- PRs require passing build before merge
- Squash merge to main
- Tag releases on main after deploy

## Testing

- /health endpoint required on all APIs
- End-to-end test any new form or payment flow before closing a story
- Unit tests for business logic; integration tests for DB access
