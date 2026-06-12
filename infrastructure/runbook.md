# Infrastructure Runbook

## Deploy Website

Automatic on push to `main` in `sgi-website` repo. GitHub Actions handles build, S3 sync, and CloudFront invalidation.

To manually trigger:
1. Push any commit to main, or
2. Go to GitHub Actions → sgi-website → Run workflow

## Restart API on EC2

```bash
aws ssm start-session --target i-0d0b9e0848d5cf28e --profile personal
sudo systemctl restart sgi-api   # or whatever service name is configured
```

## Run EF Core Migration (Local)

1. Start the DB tunnel: `dbtunnel`
2. From `sgi-app/src/Api/`:
```bash
dotnet ef database update
```

## Check API Health

```
GET https://<api-domain>/health
```

Expected response:
```json
{ "status": "healthy", "database": "connected" }
```

## Rotate DB Password

1. Generate new password
2. Update in AWS Secrets Manager (`sgi/dev/db-credentials` or `sgi/production/db-credentials`)
3. Update the RDS user password via SSM:
```bash
psql -h <rds-endpoint> -U sgi_admin -d postgres -c "ALTER USER sgi_app_dev PASSWORD 'newpassword';"
```
4. Restart the API to pick up the new secret

## Common Issues

| Issue | Fix |
|-------|-----|
| `dbtunnel` fails — plugin not found | `brew install --cask session-manager-plugin` |
| API can't connect to DB | Check tunnel is running, verify Secrets Manager secret is current |
| Contact form not sending | Check Lambda logs in CloudWatch, verify SES domain is verified |
