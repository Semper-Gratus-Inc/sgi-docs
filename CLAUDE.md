# Semper Gratus Inc. — Claude Setup Guide

This file gives Claude full context for working on Semper Gratus Inc. projects.
Place this file at `~/SemperGratus/CLAUDE.md` on any new machine.

---

## Who I Am

- **Name:** Phillip Burt
- **Company:** Semper Gratus Inc. (software consulting startup)
- **Personal email:** phillip.a.burt@gmail.com
- **Business email:** phillip.burt@sempergratusinc.com
- **Contact/inquiry email:** hello@sempergratusinc.com (alias → phillip.burt@)
- **Domain:** sempergratusinc.com
- **Day job:** Works at AMP Corporate Electronics (use separate GitHub account `phillip-burt_ampce` for that)

---

## Machine Setup (New Mac)

### 1. Install Required Tools

```bash
# Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# AWS CLI
brew install awscli

# GitHub CLI
brew install gh

# Session Manager Plugin (for SSM tunnels to RDS)
brew install --cask session-manager-plugin

# .NET 10 SDK
brew install --cask dotnet-sdk

# Node.js (for Next.js frontend work)
brew install node
```

### 2. Configure AWS

```bash
aws configure --profile personal
```

When prompted:
- **AWS Access Key ID:** Create a new key for `phillip-cli` IAM user at console.aws.amazon.com → IAM → Users → phillip-cli → Security credentials → Create access key
- **AWS Secret Access Key:** From above step
- **Default region:** `us-east-1`
- **Default output format:** `json`

Verify it works:
```bash
aws sts get-caller-identity --profile personal
# Should return account 870140465651, user phillip-cli
```

### 3. Configure GitHub (Two Accounts)

```bash
# Add personal account first (for Semper Gratus work)
gh auth login --hostname github.com
# Sign in as: pburt-dev

# Add work account
gh auth login --hostname github.com
# Sign in as: phillip-burt_ampce

# Switch between them
gh auth switch --user pburt-dev          # Semper Gratus work
gh auth switch --user phillip-burt_ampce # AMP work
```

### 4. Working Directories

```bash
mkdir -p ~/SemperGratus
git clone git@github.com:Semper-Gratus-Inc/sgi-app.git ~/SemperGratus/sgi-app
git clone git@github.com:Semper-Gratus-Inc/sgi-website.git ~/SemperGratus/sgi-website
git clone git@github.com:Semper-Gratus-Inc/sgi-docs.git ~/SemperGratus/sgi-docs
```

### 5. DB Tunnel Alias (add to ~/.zshrc)

```bash
alias dbtunnel='aws ssm start-session \
  --target i-0d0b9e0848d5cf28e \
  --document-name AWS-StartPortForwardingSessionToRemoteHost \
  --parameters "{\"host\":[\"sgi-postgres-prod.cq3w6wqcs0mx.us-east-1.rds.amazonaws.com\"],\"portNumber\":[\"5432\"],\"localPortNumber\":[\"5433\"]}" \
  --profile personal'
```

Run `dbtunnel` before starting local API dev. Tunnels localhost:5433 → RDS:5432.

---

## AWS Infrastructure

**Account ID:** `870140465651`
**Region:** `us-east-1`
**Always use:** `--profile personal` on all AWS CLI commands

### Networking
| Resource | ID |
|---|---|
| VPC | `vpc-0b9ad97d7bd162715` (10.0.0.0/16) |
| Public Subnet 1a | `subnet-08d013e41537dde6e` (10.0.1.0/24) |
| Public Subnet 1b | `subnet-0c572eb1c64f9f6e8` (10.0.2.0/24) |
| Private Subnet 1a | `subnet-00165ae47ba260f73` (10.0.10.0/24) |
| Private Subnet 1b | `subnet-01b4a112c7ea1c31c` (10.0.11.0/24) |
| Internet Gateway | `igw-0367164014673d767` |
| Security Group - Web | `sg-048304f85f23c2602` (80/443 inbound) |
| Security Group - App | `sg-075d0d20e12177fb9` (.NET port 5000) |
| Security Group - DB | `sg-07abe3e27625a5a29` (PostgreSQL 5432) |

### Servers
| Resource | ID | IP | Environment |
|---|---|---|---|
| Dev EC2 | `i-0d0b9e0848d5cf28e` | `32.197.179.242` | dev |
| Prod EC2 | `i-07fd4f896e1173ed8` | `44.206.234.1` | prod |
| RDS PostgreSQL | `sgi-postgres-prod` | Private subnet only | both |

**RDS endpoint:** `sgi-postgres-prod.cq3w6wqcs0mx.us-east-1.rds.amazonaws.com`

**Connect to servers (no SSH — use SSM Session Manager):**
```bash
aws ssm start-session --target i-0d0b9e0848d5cf28e --profile personal  # Dev
aws ssm start-session --target i-07fd4f896e1173ed8 --profile personal  # Prod
```

**Run commands on servers remotely:**
```bash
aws ssm send-command \
  --instance-ids i-0d0b9e0848d5cf28e \
  --document-name AWS-RunShellScript \
  --parameters 'commands=["your command here"]' \
  --profile personal
```

### Database
- **Engine:** PostgreSQL 16
- **Instance:** `sgi-postgres-prod` (db.t3.micro)
- **Credentials:** Stored in AWS Secrets Manager — never hardcoded
  ```bash
  aws secretsmanager get-secret-value --secret-id sgi/production/db-credentials --profile personal
  aws secretsmanager get-secret-value --secret-id sgi/dev/db-credentials --profile personal
  ```

**Databases:**
| DB Name | Purpose | User |
|---|---|---|
| `sgidev` | Development / local | `sgi_app_dev` |
| `sgi` | Production | `sgi_app` |

**Local dev connection string** (appsettings.Development.json — gitignored):
```
Host=localhost;Port=5433;Database=sgidev;Username=sgi_app_dev;Password=<from Secrets Manager>
```

### DNS & SSL
| Resource | Value |
|---|---|
| Route 53 Hosted Zone | `Z04060091XG9KWVLMXJSK` |
| ACM Certificate ARN | `arn:aws:acm:us-east-1:870140465651:certificate/f492b414-7d82-461b-a7d2-da5941c50e5d` |
| Nameservers | ns-243.awsdns-30.com, ns-1153.awsdns-16.org, ns-1580.awsdns-05.co.uk, ns-739.awsdns-28.net |

**Subdomains:**
- `sempergratusinc.com` → CloudFront (company website)
- `www.sempergratusinc.com` → CloudFront
- `dev.sempergratusinc.com` → Dev EC2 (32.197.179.242)
- `app.sempergratusinc.com` → Prod EC2 (44.206.234.1)

### CloudFront & S3
| Resource | Value |
|---|---|
| CloudFront Distribution ID | `E3SG546WOCI8JM` |
| CloudFront Domain | `d1ppav9ndlk83l.cloudfront.net` |
| Website S3 Bucket | `sgi-website-sempergratusinc` |
| Deploy Artifacts Bucket | `sgi-deploy-870140465651` |
| CloudTrail Logs Bucket | `sgi-cloudtrail-logs-870140465651` |
| VPC Flow Logs Bucket | `sgi-vpc-flowlogs-870140465651` |

**Deploy website:**
```bash
aws s3 sync ./out s3://sgi-website-sempergratusinc --profile personal
aws cloudfront create-invalidation --distribution-id E3SG546WOCI8JM --paths "/*" --profile personal
```

### Lambda & SES
| Resource | Value |
|---|---|
| Contact Form Lambda | `sgi-contact-form` |
| SES Domain | `sempergratusinc.com` (verified) |
| SES Status | **Sandbox** — request production access before real customer emails |
| Contact email | `hello@sempergratusinc.com` (alias → phillip.burt@sempergratusinc.com) |

**SES production access:** AWS Console → SES → Account dashboard → Request production access

### Security Controls
| Control | Status |
|---|---|
| CloudTrail | Active — `sgi-audit-trail` |
| GuardDuty | Active — detector `9ecf4f4145a1a8eda8d64d93fdc8a1d2` |
| VPC Flow Logs | Active — `fl-079f833e420824b2f` |
| S3 Public Access | Blocked account-wide |
| SSH | Disabled — use SSM Session Manager only |
| IMDSv2 | Enforced on all EC2 instances |
| MFA | Enabled on root + phillip-cli IAM user |

---

## GitHub

- **Personal account:** `pburt-dev`
- **Work account:** `phillip-burt_ampce` (AMP Corporate — keep separate)
- **Semper Gratus org:** `Semper-Gratus-Inc`

### Repos
| Repo | Purpose | Clone |
|---|---|---|
| `sgi-app` | .NET 10 Web API + EF Core | `git@github.com:Semper-Gratus-Inc/sgi-app.git` |
| `sgi-website` | Next.js static site → S3/CloudFront | `git@github.com:Semper-Gratus-Inc/sgi-website.git` |
| `sgi-docs` | Internal docs, business rules, runbooks | `git@github.com:Semper-Gratus-Inc/sgi-docs.git` |

### Branching Rules — MUST FOLLOW EVERY SESSION

```
main                          ← production, NEVER commit directly
develop                       ← CI/CD deploys to dev EC2 on push
feature/phase-N-name          ← one per phase, branched off develop
feature/phase-N-story-N-desc  ← one per story, branched off its phase branch
fix/short-description         ← hotfixes off main only
```

**Before starting any story:**
1. Check out the current phase feature branch
2. Branch off it: `git checkout -b feature/phase-N-story-N-description`
3. Do all work on that branch — zero commits to main or develop directly

**When a story is complete:**
1. Run code review (`/superpowers:requesting-code-review`)
2. Fix all Critical and Important issues
3. Open PR: story branch → phase feature branch
4. Tag Phillip for approval — do NOT merge without it

**When a phase is complete:**
1. Open PR: phase feature branch → `main`
2. Tag Phillip for approval — do NOT merge without it

**Current active branches:**
| Branch | Phase | Status |
|---|---|---|
| `feature/phase-1-foundation` | Phase 1 — Foundation | Active |

### CI/CD (GitHub Actions)
Workflow files are in `sgi-app/.github/workflows/`:
- `deploy-dev.yml` — triggers on push to `develop` branch → deploys to dev EC2
- `deploy-prod.yml` — triggers on push to `main` branch → requires manual approval → deploys to prod EC2

**GitHub Secrets required on sgi-app repo:**
- `AWS_ACCESS_KEY_ID` — from IAM user `sgi-github-actions-deployer`
- `AWS_SECRET_ACCESS_KEY` — from same user

To create deploy keys:
```bash
aws iam create-access-key --user-name sgi-github-actions-deployer --profile personal
```

---

## Tech Stack

- **Backend:** ASP.NET Core (.NET 10), minimal API style
- **Frontend:** Next.js (static export — `output: 'export'`)
- **Database:** PostgreSQL 16 (AWS RDS), EF Core + Npgsql
- **Web server:** NGINX (reverse proxy to .NET on port 5000)
- **Hosting:** AWS EC2 (t3.small) for API; S3/CloudFront for website
- **CI/CD:** GitHub Actions
- **Secrets:** AWS Secrets Manager (prod), appsettings.Development.json (local, gitignored)

---

## First Client

**Gym owner** — needs:
- Professional website
- Online member sign-up
- Digital contracts + e-signatures
- Recurring billing (Stripe)
- Member self-service portal
- Owner dashboard

---

## Monthly AWS Cost Estimate

~$60–70/month:
- 2x EC2 t3.small: ~$30
- RDS db.t3.micro: ~$15
- GuardDuty: ~$4
- CloudFront/S3/misc: ~$5–10

---

## Still TODO

- [ ] Add `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as GitHub repo secrets (sgi-app)
- [ ] Install SSL certs on NGINX via Let's Encrypt / certbot on each EC2
- [ ] Request SES production access (currently sandbox)
- [ ] Set up Stripe account for payment processing
- [ ] Choose e-signature provider (DocuSign or HelloSign)
- [ ] Build gym client project (in progress — Story #2 next)
