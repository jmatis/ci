# ci

Common continuous integration pipeline: shared GitHub Actions workflows for jmatis repos.
Each scanner runs from a digest-pinned image in `<scanner>/Dockerfile`; Dependabot keeps those current,
and `test.yml` runs every scanner on its pull requests before you merge.

| Workflow | Finds | Fails on |
|---|---|---|
| `semgrep.yml` | Bugs and insecure patterns in your code | any finding |
| `trivy.yml` | Vulnerable dependencies, secrets, misconfigurations | any finding (repo `.trivyignore` applies) |
| `osv-scanner.yml` | Vulnerable dependencies (osv.dev database) | any finding (`osv-scanner.toml` next to the lockfile applies) |
| `gitleaks.yml` | Secrets anywhere in the git history | any finding (repo `.gitleaksignore` applies) |
| `socket.yml` | Malicious or risky npm packages | your Socket policy; needs the `SOCKET_CLI_API_TOKEN` secret |

Use one from another repo's `.github/workflows/<name>.yml`:

```yaml
on:
  push:
  schedule:
    - cron: "17 6 * * 1"
  workflow_dispatch:

jobs:
  semgrep:
    uses: jmatis/ci/.github/workflows/semgrep.yml@main
```

Socket also needs the token passed in:

```yaml
  socket:
    uses: jmatis/ci/.github/workflows/socket.yml@main
    secrets:
      SOCKET_CLI_API_TOKEN: ${{ secrets.SOCKET_CLI_API_TOKEN }}
```
