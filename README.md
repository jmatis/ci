# ci

Common continuous integration pipeline: shared GitHub Actions workflows for jmatis repos.
Each scanner runs from a digest-pinned image in `<scanner>/Dockerfile`; Dependabot keeps those current,
and `test.yml` runs every scanner on its pull requests before you merge. Each folder's README explains
what the scanner is for, how it works and how to handle false positives.

| Workflow | Finds | Fails on |
|---|---|---|
| [`semgrep.yml`](semgrep/README.md) | Bugs and insecure patterns in your code | any finding |
| [`trivy.yml`](trivy/README.md) | Vulnerable dependencies, secrets, misconfigurations | any finding (repo `.trivyignore` applies) |
| [`osv-scanner.yml`](osv-scanner/README.md) | Vulnerable dependencies (osv.dev database) | any finding (`osv-scanner.toml` next to the lockfile applies) |
| [`gitleaks.yml`](gitleaks/README.md) | Secrets anywhere in the git history | any finding (repo `.gitleaksignore` applies) |
| [`socket.yml`](socket/README.md) | Malicious or risky npm packages | your Socket policy; needs the `SOCKET_CLI_API_TOKEN` secret |

Not a scanner, so not part of `scan.yml` or `test.yml` (this repo has no docs of its own to check):

| Workflow | Checks | Fails on |
|---|---|---|
| [`docs.yml`](docs/README.md) | The committed API docs match the code | `npm run docs` changes anything in `docs/api` |

## Using all of them

Put this in the repo to scan as `.github/workflows/scan.yml`, and add the Socket token secret
(see [socket/README.md](socket/README.md)). Five jobs of about a minute each per push.

```yaml
# Security scanners from jmatis/ci. Each job fails (and GitHub emails you) on a finding.
name: scan

on:
  push:
  schedule:
    - cron: "17 6 * * 1" # Mondays, picks up new rules and newly published vulnerabilities
  workflow_dispatch:

jobs:
  semgrep:
    uses: jmatis/ci/.github/workflows/semgrep.yml@main
  trivy:
    uses: jmatis/ci/.github/workflows/trivy.yml@main
  osv-scanner:
    uses: jmatis/ci/.github/workflows/osv-scanner.yml@main
  gitleaks:
    uses: jmatis/ci/.github/workflows/gitleaks.yml@main
  socket:
    uses: jmatis/ci/.github/workflows/socket.yml@main
    secrets:
      SOCKET_CLI_API_TOKEN: ${{ secrets.SOCKET_CLI_API_TOKEN }}
```

The weekly schedule only runs from the repo's default branch.
