# CipherMap GitHub Action — Post-Quantum Cryptography, SARIF Code Scanning & Signed CycloneDX 1.7 CBOM

Run zero-egress post-quantum cryptography (PQC) discovery across 12 languages, `.jar`/`.whl` archives, `.jks`/`.p12` keystores, and TLS/IaC configs directly inside your own GitHub Actions runner — uploading SARIF 2.1.0 to the **GitHub Security → Code Scanning** tab and generating signed **CycloneDX 1.7 CBOM** artifacts with zero source code leaving your workflow.

[![Public GitHub Repo PQC Scanner](https://www.ciphermap.io/scan/badge/github/open-quantum-safe/liboqs-go)](https://www.ciphermap.io/scan?utm_source=action_readme&utm_medium=badge)

- **Instant Public Repo Scorecard (Zero Install):** [https://www.ciphermap.io/scan](https://www.ciphermap.io/scan)
- **Claim Free 7-Day Corporate Trial Key (`CIPHERMAP_LICENSE`):** [https://www.ciphermap.io/eval-pack#trial](https://www.ciphermap.io/eval-pack#trial)
- **GitHub Integration & Air-Gapped Runner Guide:** [https://www.ciphermap.io/github](https://www.ciphermap.io/github)

## 1. Enterprise Supply-Chain Hardened Workflow (`v1.0.6`)

Following industry-wide GitHub Enterprise SHA-pinning and StepSecurity `harden-runner` policies (`CVE-2025-30066` mitigation), we recommend pinning actions to full 40-character commit SHAs and restricting runner egress:

```yaml
name: CipherMap PQC, CNSA 2.0 & Signed CBOM Gate
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
      - name: Harden GitHub Runner (Optional — StepSecurity)
        uses: step-security/harden-runner@ec9f2d5744a09debf3a187a3f4f675c53b671911 # v2.13.0
        with:
          egress-policy: block
          allowed-endpoints: >
            github.com:443
            api.github.com:443
            www.ciphermap.io:443

      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Run CipherMap Zero-Egress PQC & CBOM Gate
        uses: Ciphermap-io/ciphermap-action@v1 # Pin to the v1.0.6 40-char commit SHA in regulated orgs
        with:
          fail-on: high
          target: cnsa-2.0
          include-archives: "true"
          cbom-file: ciphermap-cbom-1.7.json
          cbom-spec-version: "1.7"
          license-key: ${{ secrets.CIPHERMAP_LICENSE }}
```

### Zero-Egress Air-Gapped Self-Hosted Runners (`0` Network Calls)
If `/usr/local/bin/ciphermap` (or `~/.local/bin/ciphermap`) matching `cli-version` (`v1.0.6`) is pre-installed on a self-hosted GitHub Enterprise Server runner image, `ciphermap-action` automatically detects the local binary and executes **zero** outbound `curl` calls — running 100% under `egress-policy: block` without even needing `www.ciphermap.io:443` allowlisted.

When running on ephemeral `ubuntu-latest` (`Linux x86_64`) without a pre-baked binary, supplying `license-key` authenticates against `https://www.ciphermap.io/api/dl/ci-cli`, verifies the `X-CipherMap-SHA256` digest + pinned `CipherMap Version v1.0.6` string, and executes `scan` / `cbom` with `--offline --no-telemetry`.

## 2. Inputs

| Input | Default | Meaning |
|---|---|---|
| `fail-on` | `high` | Severity that fails the build (`info`, `low`, `medium`, `high`, `critical`, `none`) |
| `target` | — | Compliance profile (`cnsa-2.0`, `fips-140-3`, `nist-sp-800-56b`, `bsi-tr-02102`, `anssi-pqc`) |
| `license-key` | — | Signed `CIPHERMAP_LICENSE` secret (`CIPHERMAP-TEAM-...`, `CIPHERMAP-ENTERPRISE-...`, or 7-day corporate trial key) |
| `cli-version` | `v1.0.6` | Exact CipherMap CLI version verified before scan execution |
| `sarif-file` | `ciphermap.sarif` | Output path for SARIF 2.1.0 report |
| `upload-sarif` | `true` | Upload SARIF to GitHub Security → Code Scanning (`if: always()`) |
| `cbom-file` | — | Emit deterministic CycloneDX 1.6 / 1.7 CBOM JSON |
| `cbom-spec-version` | `1.7` | CycloneDX CBOM specification version (`1.6` or `1.7`) |
| `cbom-html-file` | — | Emit standalone zero-backend Interactive HTML CBOM Explorer |
| `attest-slsa` | `false` | Emit Hybrid PQC (`Ed25519 + ML-DSA-65`) SLSA v1.1 DSSE provenance |
| `compliance-framework` | — | Emit regulatory working paper (`cnsa-2.0`, `pci-dss-4`, `fedramp`, `cmmc`, `eu-dora`, `oscal`, `all`, etc.) |
| `wrapper-target` | — | Generate zero-code PQC edge wrapper (`envoy`, `nginx`, `haproxy`, `k8s-sidecar`, `ipsec`, `cert-manager`) |
| `include-archives` | `false` | Inspect `.jar`, `.war`, `.ear`, `.whl`, `.apk`, `.aab`, `.nupkg`, `.zip` archives |
| `baseline-file` | — | Gate PRs on continuous PQC drift against a prior baseline JSON |
| `vendor-sla` | `false` | Emit Third-Party Vendor PQC SLA procurement scorecard |
| `args` | — | Extra whitespace-delimited CLI arguments |

## 3. OWASP Dependency-Track (`v4.12–4.14` & `v5.x Hyades`) CBOM Ingestion

CipherMap supports **both** native OWASP Dependency-Track `v5.x` (`Hyades` `cryptographic-asset` ingestion) **and** Dependency-Track `v4.12.x–4.14.x` (automatically projecting `cryptographic-asset` components into lossless `cdx:cbom:*` file carriers with async token polling and readback verification so `4.x` never silently drops cryptographic assets):

```yaml
      - name: Push Signed CBOM to OWASP Dependency-Track (v4 or v5 Auto-Detect)
        env:
          CIPHERMAP_LICENSE: ${{ secrets.CIPHERMAP_LICENSE }}
          CIPHERMAP_DEPENDENCY_TRACK_API_KEY: ${{ secrets.DTRACK_API_KEY }}
          DTRACK_URL: ${{ secrets.DTRACK_URL }}
          DTRACK_PROJECT_UUID: ${{ secrets.DTRACK_PROJECT_UUID }}
        run: |
          if [ -n "${CIPHERMAP_DEPENDENCY_TRACK_API_KEY:-}" ]; then
            "${CIPHERMAP_EXECUTABLE:-ciphermap}" cbom . \
              --offline \
              --include-archives \
              --spec-version 1.6 \
              --dt-compat auto \
              --push-url "${DTRACK_URL}" \
              --project-id "${DTRACK_PROJECT_UUID}"
          fi
```

## 4. Shift-Left Companion: Local `stdio` MCP Referee (`Claude Code`, `Cursor`, `Windsurf`)

Pair this CI gate with [`Ciphermap-io/ciphermap-mcp`](https://github.com/Ciphermap-io/ciphermap-mcp) (`io.github.Ciphermap-io/ciphermap-mcp` on the Official MCP Registry) and `ciphermap immunity emit-agent-rules` so AI coding assistants never introduce Round-3 `kyber768` drafts, `RSA-2048`/`SHA-1` regressions, or Trojan Unicode `.cursorrules` / `CLAUDE.md` configs (`IMM-AICFG-001`) before opening a PR.
