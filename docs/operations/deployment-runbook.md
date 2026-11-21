# FastAPI Production Lab — Deployment & Recovery Runbook

## 1. Purpose

This runbook documents the procedures used to deploy, monitor, troubleshoot, roll back, and recover the FastAPI Production Lab staging environment.

**Environment:** Render Staging  
**Application:** FastAPI  
**Database:** Render PostgreSQL  
**CI/CD:** GitHub Actions  
**Container Registry:** GHCR

> This runbook applies to the staging/demo environment. It is not a production disaster-recovery plan.

## 2. System Endpoints

| Service | URL |
|---|---|
| Application | https://fastapi-production-lab.onrender.com |
| Swagger Documentation | https://fastapi-production-lab.onrender.com/docs |
| Health Check | https://fastapi-production-lab.onrender.com/api/v1/health |
| Readiness Check | https://fastapi-production-lab.onrender.com/api/v1/ready |
| GitHub Actions | https://github.com/NamNguyen0903/fastapi-production-lab/actions |

## 3. Deployment Procedure

1. Create a feature branch from the latest `main`.
2. Implement and test changes.
3. Open a pull request.
4. Verify all six quality and security checks pass.
5. Review and merge the pull request.
6. Confirm the GitHub Actions workflow runs successfully on `main`.
7. Verify GHCR image publishing.
8. Confirm Render deployment is triggered.
9. Verify the Render deployment becomes `live`.
10. Verify the deployed commit SHA matches the expected commit.
11. Run health and readiness smoke tests.

**Deployment architecture note:** GHCR publishing and Render deployment are separate operations. Render currently builds its container from GitHub source at the selected commit SHA instead of pulling the GHCR image.

## 4. Post-Deployment Verification

Run:

```bash
BASE_URL="https://fastapi-production-lab.onrender.com"

curl -sS -o /dev/null -w "Health: %{http_code}\n" \
  "$BASE_URL/api/v1/health"

curl -sS -o /dev/null -w "Ready: %{http_code}\n" \
  "$BASE_URL/api/v1/ready"
```

Expected result:

```text
Health: 200
Ready: 200
```

Also confirm the deployment commit in Render matches the expected GitHub commit.

HTTP 200 responses alone do not establish that the correct version was deployed.

## 5. Incident: CI Pipeline Failed

**Symptoms**

- A required GitHub Actions check reports failure.
- The pull request cannot be merged.

**Procedure**

1. Open GitHub Actions.
2. Identify the failed job.
3. Read the relevant job logs.
4. Determine whether the failure comes from formatting, linting, tests, dependency scanning, secret scanning, or container scanning.
5. Fix the underlying issue on the feature branch.
6. Push the corrected changes.
7. Confirm all required checks pass.

Do not bypass branch protection or disable security checks simply to merge a failing change.

## 6. Incident: Render Deployment Failed

**Symptoms**

- Render reports a failed deployment status.
- GitHub Actions deployment verification fails.
- The expected commit is not Live.

**Procedure**

1. Inspect the GitHub Actions Deploy job.
2. Identify the Render deployment ID.
3. Open the corresponding deployment in the Render Dashboard.
4. Read build and runtime logs.
5. Verify the Docker build configuration and required environment variable names without exposing secret values.
6. Determine whether the failure occurred during build, startup, or runtime verification.
7. Correct the issue in a feature branch.
8. Run the required CI checks.
9. Deploy the corrected commit through the normal pipeline.

If a previously successful deployment remains Live and healthy, avoid unnecessary rollback.

## 7. Incident: Health Check Failed

**Symptoms**

- `/api/v1/health` returns an unexpected HTTP status.
- The endpoint is unreachable or times out.

**Procedure**

1. Confirm the Render Web Service status.
2. Inspect application logs.
3. Check whether the service is starting or restarting.
4. Verify the expected deployed commit.
5. Confirm the application is configured to listen on the expected port.
6. Recheck the endpoint after confirming service readiness.

If the latest deployment introduced the outage, assess rollback to a previously working deployment.

## 8. Incident: Database Readiness Failed

**Symptoms**

- `/api/v1/health` may succeed.
- `/api/v1/ready` fails.

**Procedure**

1. Check the Render PostgreSQL service status.
2. Inspect application logs for database connectivity errors.
3. Verify that `DATABASE_URL` is configured correctly without displaying its value.
4. Check database availability and network connectivity.
5. Confirm the schema revision is compatible with the deployed application.
6. Review recent database migrations.

**Important:** Do not reset, delete, recreate, or downgrade the database as a first troubleshooting step.

Database changes require a separate review of data integrity, backup availability, and migration compatibility.

## 9. Manual Rollback Procedure

Use rollback when a newly deployed application version causes a significant regression and a previously successful version is known to be compatible with the current database.

**Before rollback**

- Identify the last known good deployment.
- Review database migration compatibility.
- Check whether the incident involves application code or database state.
- Record the current deployment commit SHA.

**Rollback steps**

1. Open the Render Dashboard.
2. Select the FastAPI staging Web Service.
3. Open deployment history.
4. Identify the last known good deployment.
5. Select the Render rollback action.
6. Confirm the rollback.
7. Wait until the deployment reports `Live`.
8. Verify the deployed commit SHA.
9. Run both smoke tests.
10. Record the outcome.

**Database warning:** Rolling back an application does not roll back Alembic migrations or database data.

If an incompatible database migration has already been applied, application rollback alone may not restore service.

## 10. Recovery to the Latest Approved Version

Once the problem has been corrected:

1. Confirm the corrected commit passed CI.
2. Deploy it through GitHub Actions or an approved recovery procedure.
3. Verify the deployment becomes Live.
4. Confirm the expected commit SHA.
5. Run health and readiness smoke tests.
6. Review application logs.
7. Record the recovery result.

## 11. Database Migration Safety

Alembic manages database schema changes.

Useful commands:

```bash
alembic heads
alembic current
alembic upgrade head
```

Run these commands only after verifying the target `DATABASE_URL`.

The staging environment currently uses manually executed migrations.

Before applying migrations:

- Review generated migration code.
- Confirm database compatibility.
- Assess whether the migration is reversible.
- Identify the backup and recovery plan.
- Avoid destructive changes without explicit review.

A successful application deployment does not automatically prove migration safety.

## 12. Security Incident Handling

If Gitleaks detects a possible credential:

1. Stop the affected release.
2. Identify which credential may have been exposed.
3. Revoke or rotate the affected credential.
4. Investigate where the secret was introduced.
5. Remove the secret from the code and configuration.
6. Assess whether Git history remediation is required.
7. Rerun security checks.
8. Review the incident before approving deployment.

Deleting a secret from the latest commit does not invalidate a credential already exposed in Git history.

If dependency or container scanning reports vulnerabilities, investigate the affected packages, available fixes, and actual exposure.

Do not treat a successful configured gate as proof that all vulnerabilities have been resolved.

## 13. Known Operational Risks

- The recorded assessment baseline includes 47 unresolved HIGH container findings.
- Render Free resources have availability and performance limitations.
- Automated database backup/restore verification is not implemented.
- Database migrations are currently manual.
- Rollback is performed manually.
- Dedicated monitoring and alerting are not implemented.
- Render builds from GitHub source rather than deploying the exact GHCR artifact.

See [Security Risk Register](../security/risk-register.md).

## 14. Incident Report Template

For each significant incident, record:

| Field | Description |
|---|---|
| Incident ID | Unique identifier |
| Date and Time | When the incident was detected |
| Environment | Staging |
| Affected Component | API, database, deployment, CI/CD |
| Symptoms | Observed failure |
| Commit SHA | Relevant deployment version |
| Root Cause | Confirmed cause or pending investigation |
| Actions Taken | Troubleshooting and recovery steps |
| Verification | Evidence of service recovery |
| Follow-up Actions | Changes needed to prevent recurrence |

## 15. Recovery Checklist

- [ ] Correct deployment identified
- [ ] Required CI checks passed
- [ ] Render deployment status is Live
- [ ] Expected commit SHA verified
- [ ] Health endpoint returns HTTP 200
- [ ] Readiness endpoint returns HTTP 200
- [ ] Relevant application logs reviewed
- [ ] Database compatibility assessed
- [ ] Incident outcome documented

A recovery is considered verified only when the relevant technical checks and incident-specific acceptance criteria have passed.