# Security Check Script Guide

## Purpose
`security-check.ps` scans the application workspace for dependencies impacted by Sonatype advisory `sonatype-2026-003429`, then generates an HTML report.

It checks:
- Direct dependencies from `package.json`
- Resolved/transitive dependencies from `package-lock.json` and `npm-shrinkwrap.json`
- Advisory impacted package+version list from Sonatype Guide

## Script Location
- `security-check.ps`

## Report Output
Default output file:
- `security-report-sonatype-2026-003429.html`

The report includes:
- Scan summary metrics
- Dependency match table
- Full advisory impacted component list (package + version)

## Prerequisites
- Windows PowerShell available (`powershell`)
- Network access to:
  - `https://guide.sonatype.com`

No extra installs are required.

## Recommended Run Commands

### Standard run (PowerShell terminal)
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\security-check.ps
```

### If `.ps1`/script execution is restricted by policy
Run in-memory from `.ps`:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((Get-Content -Raw .\security-check.ps)))"
```

### Run against a specific folder
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\security-check.ps -RootPath "C:\path\to\web-applications"
```

### Custom report output path
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\security-check.ps -ReportPath "security-report-custom.html"
```

## Parameters
- `-RootPath`
  - Root directory to scan.
  - Default: script directory (`$PSScriptRoot`), with fallback to current directory.
- `-AdvisoryId`
  - Advisory identifier to query.
  - Default: `sonatype-2026-003429`
- `-AdvisoryUrl`
  - Sonatype advisory URL for impacted components.
  - Default: `https://guide.sonatype.com/vulnerability/sonatype-2026-003429/components-impacted`
- `-ReportPath`
  - Output HTML filename/path.
  - Default: `security-report-sonatype-2026-003429.html`

## How To Read Results
- `Matches Found = 0`
  - No impacted dependencies were detected in manifests or scanned lockfiles.
- `Exact Version Matches`
  - Number of dependencies whose resolved version exactly matches an impacted advisory version.
- `Lockfiles Scanned` and `Resolved Dependencies Scanned`
  - Confirms transitive dependency analysis occurred.

## Troubleshooting

### `Could not read ... guide.sonatype.com`
Cause:
- Network/TLS/proxy restrictions.

Action:
- Verify endpoint access from your network.
- Re-run after connectivity is restored.

### `Lockfiles Scanned = 0`
Cause:
- No lockfiles present under `RootPath`, or wrong `RootPath`.

Action:
- Confirm `package-lock.json` / `npm-shrinkwrap.json` files exist in the scanned root.
- Re-run with explicit `-RootPath`.

### Script blocked by policy
Use the in-memory command:
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((Get-Content -Raw .\security-check.ps)))"
```

## Team Usage Notes
- Commit both script and report only if your team policy allows generated artifacts in git.
- Re-run after any dependency updates (package changes or lockfile refresh).
- For audit evidence, keep the generated report with timestamp and commit hash.
