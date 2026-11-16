# Security Policy

## Supported Versions

| Version | Tag | Supported |
|---|---|---|
| `v1.0.6` | `v1` / `v1.0.6` | ✅ Active |
| `< v1.0.6` | `v1.0.5` | ⚠️ Upgrade to `v1.0.6` |

## Zero-Egress & Supply-Chain Security Model

1. **No Shell Interpolation of Workflow Inputs:** Every `inputs.*` parameter in `action.yml` is passed exclusively via step `env:` variables into `bash` arrays (`set -euo pipefail`), preventing GitHub Actions expression-injection (`${{ ... }}`) attacks.
2. **Cryptographic Binary Pinning:**
   - Self-hosted air-gapped runners with `/usr/local/bin/ciphermap` (`v1.0.6`) pre-baked make **zero** outbound network requests (`egress-policy: block` compatible).
   - Ephemeral `Linux x86_64` runners with `license-key` download the CLI over TLS 1.2+ from `https://www.ciphermap.io/api/dl/ci-cli` and verify the `64`-hex `X-CipherMap-SHA256` digest + exact `CipherMap Version v1.0.6` banner before execution.
3. **Offline Scan Execution:** All scans and CBOM builds execute locally inside your runner with `--offline --no-telemetry` — zero source code, keys, or findings leave your CI environment.

## Reporting a Vulnerability

Please report security issues privately to **security@ciphermap.io** (see `https://www.ciphermap.io/.well-known/security.txt`). Do not open public GitHub issues for undisclosed vulnerabilities. We acknowledge reports within 24 business hours.
