# GitHub Artifact Attestations & Build Provenance

## Overview

**GitHub Artifact Attestations** allow you to generate unfalsifiable build provenance and integrity guarantees for your software artifacts. By tying compiled binaries, containers, and packages directly back to the exact GitHub repository, workflow file, commit SHA, and triggering event that created them, artifact attestations protect against supply chain tampering and ensure software transparency.

Attestations adhere to the [SLSA (Supply-chain Levels for Software Artifacts)](https://slsa.dev) build provenance specification, offering cryptographically verifiable proof of an artifact's origin.

---

## Key Features

1. **Cryptographic Signatures without Key Management**
   - Uses GitHub's internal root signing authority (leveraging Sigstore public key infrastructure) to sign build artifacts automatically during workflow runs.
   - Eliminates the need to manage, store, or rotate external private keys or certificates.

2. **Detailed, Non-Tamperable Metadata**
   - Captures SLSA-compliant JSON metadata detailing:
     - Source repository and organization
     - Workflow path, run ID, and triggering event
     - Exact commit SHA and git reference
     - Runner environment parameters
   - Metadata is stored securely and linked to the GitHub repository release or run.

3. **Seamless Verification**
   - Verify artifacts easily using the GitHub CLI (`gh attestation verify`) or Sigstore tooling.
   - Ensures that binaries, container images, or packages originate genuinely from an authorized GitHub Actions workflow.

4. **Policy Enforcement**
   - Evaluates captured JSON provenance statements against security policies using engines such as Open Policy Agent (OPA) or Sigstore policy enforcement tools.
   - Enforces organizational security standards prior to deploying software into production environments.

---

## How to Generate Artifact Attestations

To generate artifact attestations in a GitHub Actions workflow, perform the following steps:

### 1. Set Workflow Permissions

Your GitHub Actions job requires specific OIDC token and attestation permissions:

```yaml
permissions:
  id-token: write      # Required for requesting the OIDC token used in signing
  attestations: write  # Required for writing artifact attestations
  contents: read       # Required for reading repo contents
```

### 2. Add the Attestation Step

Use the official `actions/attest-build-provenance` action after generating your build artifact:

```yaml
name: Build and Attest

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      attestations: write
      contents: read

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build artifact
        run: |
          mkdir -p dist
          echo "Hello World" > dist/my-app

      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v1
        with:
          subject-path: 'dist/my-app'
```

---

## How to Verify Artifact Provenance

You can verify the provenance of a built artifact using the GitHub CLI (`gh`).

### Verifying a File / Binary

```bash
gh attestation verify my-app --repo owner/repository-name
```

### Verifying a Container Image

```bash
gh attestation verify oci://ghcr.io/owner/image-name:tag --owner owner
```

### Example Verification Output

When verification succeeds, `gh attestation verify` displays details confirming:
- The signing authority and issuer (`https://token.actions.githubusercontent.com`)
- The source repository and workflow file
- The exact git commit SHA that produced the build artifact

---

## References & Documentation

- [GitHub Docs: Using artifact attestations to establish provenance for builds](https://docs.github.com/en/actions/security-guides/using-artifact-attestations-to-establish-provenance-for-builds)
- [GitHub Docs: Establishing provenance and integrity for your projects](https://docs.github.com/en/code-security/security-advisories/working-with-repository-security-advisories/establishing-provenance-and-integrity-for-your-projects)
- [SLSA Build Provenance Specification](https://slsa.dev/provenance)
