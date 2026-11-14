# CipherMap GitHub Action — Post-Quantum Cryptography, SARIF Code Scanning & Signed CycloneDX 1.7 CBOM

Run zero-egress post-quantum cryptography (PQC) discovery across 12 languages, `.jar`/`.whl` archives, `.jks`/`.p12` keystores, and TLS/IaC configs directly inside your own GitHub Actions runner — uploading SARIF 2.1.0 to the **GitHub Security → Code Scanning** tab and generating signed **CycloneDX 1.7 CBOM** artifacts with zero source code leaving your workflow.

[![Public GitHub Repo PQC Scanner](https://www.ciphermap.io/scan/badge/github/open-quantum-safe/liboqs-go)](https://www.ciphermap.io/scan?utm_source=action_readme&utm_medium=badge)

- **Instant Public Repo Scorecard (Zero Install):** [https://www.ciphermap.io/scan](https://www.ciphermap.io/scan)
- **Claim Free 7-Day Trial Key (`CIPHERMAP_LICENSE`):** [https://www.ciphermap.io/eval-pack#trial](https://www.ciphermap.io/eval-pack#trial)
- **GitHub Integration Guide:** [https://www.ciphermap.io/github](https://www.ciphermap.io/github)

## How to Scan Private Repositories in GitHub Actions

1. Claim your 7-day corporate work-email trial key (or paid Tier A/B key) at [`ciphermap.io/eval-pack#trial`](https://www.ciphermap.io/eval-pack#trial) and save it in **Settings → Secrets and variables → Actions** as `CIPHERMAP_LICENSE`.
2. Add `.github/workflows/ciphermap-pqc.yml`:

```yaml
name: CipherMap PQC & CNSA 2.0 Gate
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write   # Uploads SARIF 2.1.0 to GitHub Code Scanning

jobs:
  pqc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Ciphermap-io/ciphermap-action@v1
        with:
          fail-on: high
          target: cnsa-2.0
          cbom-file: ciphermap-cbom-1.7.json
          license-key: ${{ secrets.CIPHERMAP_LICENSE }}
```

When `license-key` (or `CIPHERMAP_LICENSE`) is supplied on `ubuntu-latest` (`Linux x86_64`), the Action authenticates against `https://www.ciphermap.io/api/dl/ci-cli`, verifies the `X-CipherMap-SHA256` digest + pinned `cli-version`, and runs `--offline --no-telemetry` inside your private runner.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `fail-on` | `high` | Severity that fails the build (`info`…`critical`, `none`) |
| `target` | — | Compliance profile (`cnsa-2.0`, `fips-140-3`, `nist-sp-800-56b`, `bsi-tr-02102`, `anssi-pqc`) |
| `license-key` | — | Signed `CIPHERMAP_LICENSE` secret (`CIPHERMAP-TEAM-...`, `CIPHERMAP-ENTERPRISE-...`, or 7-day trial key) |
| `sarif-file` | `ciphermap.sarif` | Output path for SARIF 2.1.0 report |
| `upload-sarif` | `true` | Upload SARIF to GitHub Security → Code Scanning (`if: always()`) |
| `cbom-file` | — | Emit deterministic CycloneDX 1.6 / 1.7 CBOM JSON |
| `cbom-spec-version` | `1.7` | CycloneDX CBOM specification version (`1.6` or `1.7`) |
| `include-archives` | `false` | Inspect `.jar`, `.war`, `.ear`, `.whl`, `.apk`, `.nupkg` archives |
| `args` | — | Extra whitespace-delimited CLI arguments |

## Security & Zero-Egress Guarantee

- **Zero Source Code Egress:** All AST, bytecode (`.jar`, `.whl`), keystore (`.jks`, `.p12`), and TLS/IaC analysis runs locally inside your GitHub Actions runner with `--offline --no-telemetry`.
- **Cryptographic Binary Verification:** Verifies `X-CipherMap-SHA256` and pinned CLI version (`v1.0.5`) prior to execution.
- **License:** MIT (Composite GitHub Action wrapper). Full documentation and Bitbucket / self-hosted runner templates at [`ciphermap.io/github`](https://www.ciphermap.io/github).
