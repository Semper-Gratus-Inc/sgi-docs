# Architecture Overview

## System Components

| Component | Tech | Hosting |
|-----------|------|---------|
| Public website | Next.js (static export) | S3 + CloudFront |
| API | ASP.NET Core (.NET 10) | EC2 (sgi-dev, Amazon Linux) |
| Database | PostgreSQL 16 | RDS (sgi-postgres-prod, private subnet) |
| Email | AWS SES | us-east-1 |
| Contact form | AWS Lambda + API Gateway | us-east-1 |
| Secrets | AWS Secrets Manager | us-east-1 |

## Infrastructure Notes

- RDS is in a private subnet — local access requires SSM port-forwarding tunnel (see [AWS Access Guide](../infrastructure/aws-access.md))
- EC2 IAM role grants `secretsmanager:GetSecretValue` — no hardcoded AWS credentials on the instance
- SES is in sandbox mode — must request production access before going live with real customer emails

## Databases

| Database | Purpose | Credentials Secret |
|----------|---------|-------------------|
| `sgidev` | Development / staging | `sgi/dev/db-credentials` |
| `sgi` | Production | `sgi/production/db-credentials` |

## Secrets Manager Keys

| Secret | Contents |
|--------|----------|
| `sgi/master/db-credentials` | RDS master (admin) credentials |
| `sgi/dev/db-credentials` | App dev user credentials |
| `sgi/production/db-credentials` | App prod user credentials |

## Deployment

- Website: GitHub Actions on push to main → build → S3 sync → CloudFront invalidation
- API: Manual deploy to EC2 for now (CI/CD planned in later phase)
