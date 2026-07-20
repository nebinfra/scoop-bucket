# scoop-bucket

**Official [Scoop](https://scoop.sh) bucket for NebInfra command-line tools on Windows.**

This repository ships Scoop manifests for the NebInfra CLI suite. Add the bucket once and install any tool with `scoop install`. Every manifest pins the upstream release URL and SHA-256 sum, so Scoop's built-in checksum verification catches any tampering at install time.

> **In scope.** Manifests for tools published from the `nebinfra/*-dist` release-asset mirrors. Windows binaries only.
> **Out of scope.** Source code, non-Windows binaries (use Homebrew on macOS / Linux), pre-release builds.

---

## Table of contents

- [Add the bucket](#add-the-bucket)
- [Available manifests](#available-manifests)
- [Verify signatures (defense-in-depth)](#verify-signatures-defense-in-depth)
- [Pin a specific version](#pin-a-specific-version)
- [Update and upgrade](#update-and-upgrade)
- [Uninstall](#uninstall)
- [Reporting concerns](#reporting-concerns)

---

## Add the bucket

```powershell
scoop bucket add nebinfra https://github.com/nebinfra/scoop-bucket
```

Once added, install any tool with the bare name:

```powershell
scoop install nebcli
scoop install nebguard
```

---

## Available manifests

| Manifest | Description | Upstream release mirror |
| --- | --- | --- |
| `nebcli` | NebInfra CLI (cluster bootstrap, Helm validation, kubectl / argocd / aws proxying) | [`nebinfra/nebcli-dist`](https://github.com/nebinfra/nebcli-dist) |
| `nebguard` | NebGuard, the AI guardrails engine for Claude Code | [`nebinfra/nebguard-dist`](https://github.com/nebinfra/nebguard-dist) |

Each manifest resolves the platform-specific release asset (`windows_x86_64` or `windows_arm64`) and verifies its SHA-256 sum against the upstream-published value.

---

## Verify signatures (defense-in-depth)

Scoop's built-in SHA-256 verification protects against tampering after the manifest is committed. For an additional cryptographic check against the NebInfra signing key, run cosign after installing.

Install cosign:

```powershell
scoop install cosign
```

Verify a tool:

```powershell
$app = 'nebcli'    # or 'nebguard'
$version = (& $app --version).Split()[-1]
$arch = if ([Environment]::Is64BitOperatingSystem) { 'x86_64' } else { 'i386' }
$asset = "${app}_${version.TrimStart('v')}_Windows_${arch}.zip"
$base = "https://github.com/nebinfra/${app}-dist/releases/download/${version}"

Invoke-WebRequest "$base/$asset"        -OutFile "$env:TEMP\$asset"
Invoke-WebRequest "$base/$asset.bundle" -OutFile "$env:TEMP\$asset.bundle"

cosign verify-blob `
  --bundle "$env:TEMP\$asset.bundle" `
  --key "https://raw.githubusercontent.com/nebinfra/trust/main/cosign-keys/${app}-prod.pub" `
  --rekor-url https://rekor.sigstore.dev `
  "$env:TEMP\$asset"
```

A successful run prints `Verified OK`. The full trust model (cosign + AWS KMS + Rekor transparency) is documented at [`nebinfra/trust`](https://github.com/nebinfra/trust).

---

## Pin a specific version

To install a specific historical version (every release is immutable):

```powershell
scoop install nebcli@1.2.3
```

To hold an installed package at its current version and skip auto-updates:

```powershell
scoop hold nebcli
```

---

## Update and upgrade

```powershell
scoop update                    # refresh bucket + scoop itself
scoop update nebcli nebguard    # upgrade installed NebInfra tools
```

Or upgrade everything across all buckets:

```powershell
scoop update *
```

---

## Uninstall

```powershell
scoop uninstall nebcli
scoop uninstall nebguard
scoop bucket rm nebinfra        # optional, drops the bucket entirely
```

For `nebguard`, also run `nebguard uninstall` before removal to restore `~/.claude/settings.json` and clean `~/.nebcore/nebguard/`.

---

## Reporting concerns

For manifest bugs (wrong checksum, broken auto-update, missing architecture), open an issue at <https://github.com/nebinfra/scoop-bucket/issues>.

For supply-chain concerns about the underlying binaries themselves, please open a **security advisory** in the relevant `-dist` repository (`nebinfra/nebcli-dist` or `nebinfra/nebguard-dist`). The full trust + signature design lives at [`nebinfra/trust`](https://github.com/nebinfra/trust).
