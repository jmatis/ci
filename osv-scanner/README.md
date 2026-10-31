# OSV-Scanner

**What it's for:** checks every lockfile in the repo (`package-lock.json` and others) against Google's
[osv.dev](https://osv.dev) vulnerability database. It's a second opinion next to Trivy: it caught the
zod advisory (GHSA-mrhx-c7xc-m3w5) in hcc-web that Trivy missed.

**How it works** (`.github/workflows/osv-scanner.yml`):
1. Fetches the pushed commit with plain `git` (no third-party actions).
2. Builds the image from this folder's `Dockerfile`, which pins `ghcr.io/google/osv-scanner` by version
   and digest. Dependabot opens a pull request when a newer release is at least 7 days old.
3. Runs `osv-scanner scan source -r` over the whole repo. Vulnerability data is queried live, so new
   advisories are caught without an image bump.

**Fails when:** any lockfile has a package with a known vulnerability.

**Accepted risk?** Put an `osv-scanner.toml` next to the lockfile, with a reason and an expiry date so
the exception can't be forgotten — see `../socket/osv-scanner.toml`:

```toml
[[IgnoredVulns]]
id = "GHSA-xxxx-xxxx-xxxx"
ignoreUntil = 2027-01-31
reason = "why this is acceptable"
```

**Typical run:** ~15 seconds (1 billed minute).

## Example

```yaml
# .github/workflows/scan.yml in the repo to scan
on:
  push:
  schedule:
    - cron: "17 6 * * 1"
  workflow_dispatch:

jobs:
  osv-scanner:
    uses: jmatis/ci/.github/workflows/osv-scanner.yml@main
```
