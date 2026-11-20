
# Security Risk Register

## Project
fastapi-production-lab

## Scan Baseline
- Date: 2026-11-20
- Scanner: Trivy
- Operating System: Debian 13.7
- Image: fastapi-production-lab:patched
- Remaining findings: 47 HIGH, 0 CRITICAL
- Unique remaining CVEs: 11

## Remediation Completed
- CVE-2026-85091
- Package: zlib1g
- Action: Upgrade Debian package
- Verification: CVE no longer detected in patched image

## Outstanding Vulnerabilities

| CVE | Package group | Findings | Scanner status | Risk decision |
|---|---|---:|---|---|
| CVE-2026-76642 | util-linux | 9 | affected | Pending |
| CVE-2026-78408 | util-linux | 9 | affected | Pending |
| CVE-2026-78409 | util-linux | 9 | affected | Pending |
| CVE-2026-78410 | util-linux | 9 | affected | Pending |
| CVE-2025-69720 | ncurses | 4 | affected | Pending |
| CVE-2026-16742 | systemd | 2 | affected | Pending |
| CVE-2026-54369 | libacl1 | 1 | affected | Pending |
| CVE-2026-119373 | sqlite3 | 1 | affected | Pending |
| CVE-2026-119374 | sqlite3 | 1 | affected | Pending |
| CVE-2026-119379 | sqlite3 | 1 | affected | Pending |
| CVE-2026-9538 | perl-base | 1 | fix_deferred | Pending |

## Risk Management Policy

- Full scans must report HIGH and CRITICAL findings.
- Fixable HIGH and CRITICAL findings block CI.
- Unfixed findings require risk assessment.
- No vulnerability is automatically accepted.
- Security exceptions must be documented and reviewed.
- Rescan when upstream fixes become available.

## Known Scanning Limitation

The pip vendored SBOM file is excluded from Trivy
scanning due to an investigated setuptools metadata
finding.

This exclusion may reduce visibility into packages
represented only by that SBOM and requires periodic
review.

## Pending Actions

- Assess exploitability and application exposure.
- Assign a risk owner.
- Record mitigation measures.
- Verify vulnerability status with Debian advisories.
- Obtain a documented risk decision before release.
