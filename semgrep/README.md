# Semgrep

**What it's for:** finds bugs and insecure patterns in *your own code* — things like `eval` on user input,
unescaped HTML output, injection, weak crypto, hard-coded secrets and unsafe GitHub Actions scripts.
The other scanners here look at dependencies or history; this is the only one that reads your code.

**How it works** (`.github/workflows/semgrep.yml`):
1. Fetches the pushed commit with plain `git` (no third-party actions).
2. Builds the image from this folder's `Dockerfile`, which pins the open-source
   `semgrep/semgrep` image by version and digest. Dependabot opens a pull request when a newer
   release is at least 7 days old.
3. Runs `semgrep scan` with the `p/default`, `p/typescript`, `p/nodejs` and `p/secrets` rule sets,
   downloaded fresh from the Semgrep registry on every run (so new rules arrive without an image bump).
   Usage metrics are off; your code is not uploaded.

**Fails when:** any rule matches (`--error`). GitHub emails you about the failed run.

**Known gap:** Semgrep can't parse `.astro` files, so only its secrets rules run on them.
Check new `set:html` uses by hand.

**False positive?** Add `// nosemgrep: <rule-id>` on the line, or list paths in a `.semgrepignore` in the scanned repo.

**Typical run:** ~20–30 seconds (1 billed minute).

## Example

```yaml
# .github/workflows/scan.yml in the repo to scan
on:
  push:
  schedule:
    - cron: "17 6 * * 1"
  workflow_dispatch:

jobs:
  semgrep:
    uses: jmatis/ci/.github/workflows/semgrep.yml@main
```
