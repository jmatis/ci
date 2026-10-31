# Gitleaks

**What it's for:** finds secrets (API keys, tokens, passwords) anywhere in the repo's **git history**,
including ones that were committed and later deleted. A deleted secret is still readable by anyone with
access to the history, so it must be rotated, not just removed. Trivy and Semgrep only see current files.

**How it works** (`.github/workflows/gitleaks.yml`):
1. Fetches the pushed commit **with its full history** using plain `git` (no third-party actions).
2. Builds the image from this folder's `Dockerfile`, which pins `ghcr.io/gitleaks/gitleaks` by version
   and digest. Dependabot opens a pull request when a newer release is at least 7 days old.
3. Runs `gitleaks git --redact` over every commit. `--redact` keeps the secret values out of the log.
   (The `safe.directory` setting lets git inside the container read a checkout owned by the runner user.)

**Fails when:** any commit contains something that looks like a secret.

**If it finds a real secret:** rotate it first (make a new one, revoke the old one). Removing it from
the code doesn't help, because it stays in the history.

**False positive?** Add its fingerprint (printed in the log, e.g. `<commit>:<file>:<rule>:<line>`) to a
`.gitleaksignore` at the scanned repo's root, with a comment saying why — see hcc-web's
`.gitleaksignore` for the Cloudflare Access audience tags.

**Typical run:** ~10 seconds (1 billed minute).

## Example

```yaml
# .github/workflows/scan.yml in the repo to scan
on:
  push:
  schedule:
    - cron: "17 6 * * 1"
  workflow_dispatch:

jobs:
  gitleaks:
    uses: jmatis/ci/.github/workflows/gitleaks.yml@main
```
