# PQC Vulnerability Scanner — GitHub Action

Detect quantum-vulnerable cryptography (RSA, ECDSA, ECDH, DES, SHA-1, etc.) in your repository and automatically migrate to NIST-approved post-quantum algorithms.

## Tiers

| | Free | Paid |
|---|---|---|
| Scan for vulnerabilities | ✅ | ✅ |
| PR annotations | ✅ | ✅ |
| Job summary table | ✅ | ✅ |
| SARIF / Security tab | ✅ | ✅ |
| Auto-fix PR | — | ✅ |
| File limit | 200 files | Unlimited |
| Setup required | None | `PQC_API_KEY` secret |

## Free tier — scan only

No sign-up required. Just add the action to any workflow:

```yaml
# .github/workflows/pqc-scan.yml
name: PQC Security Scan
on: [push, pull_request]

jobs:
  pqc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: quantumtoolkit/pqc-scanner-action@v1
        with:
          fail_on_vulnerable: true
          severity_threshold: high
```

The action will:
- Annotate PRs with inline vulnerability markers
- Write a findings table to the job summary
- Exit with code 1 if vulnerabilities are found (when `fail_on_vulnerable: true`)

## Free tier — with SARIF (Security tab integration)

Findings appear in the **GitHub Security tab** as code scanning alerts, surfaced inline on PR diffs.

```yaml
# .github/workflows/pqc-scan.yml
name: PQC Security Scan
on: [push, pull_request]

permissions:
  security-events: write   # required for upload-sarif

jobs:
  pqc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: quantumtoolkit/pqc-scanner-action@v1
        with:
          fail_on_vulnerable: false
          report_format: sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: pqc-results.sarif
```

## Paid tier — auto-fix PR

Set `PQC_API_KEY` as a repository secret, then the action will automatically open a pull request with post-quantum migrations applied.

```yaml
      - uses: quantumtoolkit/pqc-scanner-action@v1
        with:
          fail_on_vulnerable: false   # let the fix PR handle it
          pqc_api_key: ${{ secrets.PQC_API_KEY }}
          migration_mode: pqc_only
```

When vulnerabilities are found, a PR is automatically opened:

> **[PQC] Auto-migrate 3 file(s) to post-quantum algorithms**
> *RSA → ML-KEM, ECDSA → ML-DSA • 7 findings fixed*

Get an API key at [quantumtoolkit.com](https://quantumtoolkit.com).

## Inputs

| Input | Default | Description |
|---|---|---|
| `paths` | `**/*.py **/*.java` | Glob patterns to scan (space-separated) |
| `fail_on_vulnerable` | `true` | Exit 1 if vulnerabilities found above threshold |
| `severity_threshold` | `medium` | Minimum severity to report: `critical`, `high`, `medium`, `low` |
| `report_format` | `annotations` | Output format: `annotations`, `sarif`, `json` |
| `pqc_api_key` | — | API key for paid auto-fix tier; also enables scan results to appear in the platform dashboard |
| `migration_mode` | `pqc_only` | Migration strategy: `pqc_only`, `hybrid`, `migration` |
| `scan_scope` | `changed` | Files to scan: `changed` (only files in this push/PR) or `all` (entire repo) |
| `scan_id` | — | Platform scan ID (set automatically by platform integrations) |

## Detected algorithms

| Algorithm | Quantum Risk |
|---|---|
| RSA (all key sizes) | Critical |
| ECDSA / ECDH | Critical |
| DSA | Critical |
| DES / 3DES | High |
| RC4 | High |
| MD5 | Medium |
| SHA-1 | Medium |

## Languages supported

- Python (`.py`)
- Java (`.java`)

Go, C, Rust support coming in v2.

## IP protection

The action runs as a compiled native binary (via [Nuitka](https://nuitka.net/)) inside a Docker container. The binary cannot be reverse-engineered to recover Python source. Transformation logic runs entirely on QuantumToolkit's cloud — it never enters your runner.
