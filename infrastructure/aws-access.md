# AWS Access Guide

## Prerequisites

- AWS CLI configured with `personal` profile
- Session Manager Plugin installed (`brew install --cask session-manager-plugin`)
- TablePlus installed for database GUI

## Database Tunnel (Local → RDS)

RDS is in a private subnet and cannot be reached directly. Use the `dbtunnel` alias to open an SSM port-forwarding session:

```bash
dbtunnel
```

This forwards `localhost:5433` → RDS `5432` through the dev EC2 instance.

**TablePlus connection settings:**
- Host: `localhost`
- Port: `5433`
- User: `sgi_app_dev`
- Database: `sgidev`
- SSL: Required

Keep the tunnel running in a terminal tab while using TablePlus or running the API locally.

## EC2 SSH Access (SSM)

```bash
aws ssm start-session --target i-0d0b9e0848d5cf28e --profile personal
```

## Key AWS Resources

| Resource | Identifier |
|----------|-----------|
| Dev EC2 | `i-0d0b9e0848d5cf28e` |
| RDS endpoint | `sgi-postgres-prod.cq3w6wqcs0mx.us-east-1.rds.amazonaws.com` |
| SES domain | `sempergratusinc.com` (verified) |
| CloudFront | Serving sgi-website |
| Lambda | `sgi-contact-form` |

## SES Production Access

SES is currently in sandbox mode. To send to unverified addresses (real customers), request production access:

**AWS Console → SES → Account dashboard → Request production access**
