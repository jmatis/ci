# Trivy

**What it's for:** a broad first pass over the repo — vulnerable npm packages (from `package-lock.json`),
secrets left in current files, and misconfigurations in Dockerfiles and other infrastructure files.

**How it works** (`.github/workflows/trivy.yml`):
1. Fetches the pushed commit with plain `git` (no third-party actions).
2. Builds the image from this folder's `Dockerfile`, which pins `ghcr.io/aquasecurity/trivy` by version
   and digest. Dependabot opens a pull request when a newer release is at least 7 days old.
   (Pinning matters here: Trivy's own releases were hijacked in March 2026.)
3. Runs `trivy fs --scanners vuln,secret,misconfig .` from the repo root. Trivy downloads its
   vulnerability database on each run, so newly published CVEs are caught without an image bump.

**Fails when:** any vulnerability, secret or misconfiguration is found (`--exit-code 1`), at any severity.

**False positive or accepted risk?** Add the ID to a `.trivyignore` at the scanned repo's root, with a
comment saying why — see this repo's own `.trivyignore`.

**Overlap:** OSV-Scanner and Dependabot alerts also check dependencies, from different databases.
Each has caught things the others missed.

**Typical run:** ~20–35 seconds (1 billed minute).

## Example

```yaml
# .github/workflows/scan.yml in the repo to scan
on:
  push:
  schedule:
    - cron: "17 6 * * 1"
  workflow_dispatch:

jobs:
  trivy:
    uses: jmatis/ci/.github/workflows/trivy.yml@main
```
