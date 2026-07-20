# Security Policy

## Reporting a vulnerability

Report security issues privately through GitHub: open the [Security tab](https://github.com/nebinfra/scoop-bucket/security) of this repository and choose **Report a vulnerability**, or email <security@nebinfra.com>.

Please do not open public issues or pull requests for security reports.

We aim to acknowledge new reports within 3 business days and to keep you informed while we investigate. We follow coordinated disclosure: we ask that you give us a reasonable window to ship a fix before publishing details. There is no paid bounty program at this time; we are happy to credit reporters in the release notes on request.

## Scope

This repository holds the Scoop manifests that install NebInfra tools on Windows. In scope here:

- Manifest contents: download URLs, hash pins, install and update logic, and anything that could mislead users into an unsafe state.
- A manifest that installs an artifact differing from the one published on the corresponding releases page.

Vulnerabilities in the installed binaries themselves belong in the product's release repository: [nebinfra/nebcli-dist](https://github.com/nebinfra/nebcli-dist) or [nebinfra/nebguard-dist](https://github.com/nebinfra/nebguard-dist). Issues with the published verification keys belong in [nebinfra/trust](https://github.com/nebinfra/trust).

## How this bucket stays trustworthy

Manifests pin the exact hash of each release artifact, and Scoop verifies that hash at install time. The artifacts are additionally cosign-signed; you can verify them independently against the public keys in [nebinfra/trust](https://github.com/nebinfra/trust) using the instructions in the product's release repository. A mismatch between what the manifest installs and what the release page serves is a security report: please tell us immediately.

## Supported versions

Manifests track the latest release of each tool. Security fixes ship forward in new releases; there are no backported fixes to older manifest revisions.
