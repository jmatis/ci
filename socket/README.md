# Socket

**What it's for:** catches npm packages that are *deliberately* malicious or risky — malware, typosquats
(look-alike names), install scripts that run code, sudden ownership changes, obfuscated code. These have
no CVE, so Trivy, OSV-Scanner and Dependabot can't see them. Malicious packages are now the bigger npm risk.

**How it works** (`.github/workflows/socket.yml`):
1. Fetches the pushed commit with plain `git` (no third-party actions).
2. Builds the image from this folder: the `Dockerfile` pins the Node base image by version and digest,
   and `package.json` + `package-lock.json` pin the Socket CLI. Dependabot updates both (Node stays on
   major version 22, see `../.github/dependabot.yml`).
3. Runs `socket scan create --report` for the `nonamenoorg` Socket org. This uploads the **dependency
   manifests** (`package.json`, lockfiles) to Socket — not your source code — and waits for the verdict
   against your Socket security policy. Results also appear in the Socket dashboard.

**Fails when:** the scan breaks your Socket policy (`healthy: false`), or the token is missing.

**Needs:** a `SOCKET_CLI_API_TOKEN` secret in **each** repo that runs it, in two places: Settings →
Secrets and variables → **Actions**, and → **Dependabot** (runs started by Dependabot only see
Dependabot secrets). Secrets come from the calling repo, not from jmatis/ci.

**Other Socket org?** Pass `with: socket-org: <slug>` (the part after `/org/` in the dashboard URL).

**`osv-scanner.toml`** in this folder: an accepted, expiring exception for a zod 3.x advisory inside the
Socket CLI itself (no 3.x fix exists).

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
  socket:
    uses: jmatis/ci/.github/workflows/socket.yml@main
    secrets:
      SOCKET_CLI_API_TOKEN: ${{ secrets.SOCKET_CLI_API_TOKEN }}
```

## Rotating the token

GitHub secrets can't be read back, so to rotate (or when adding a new repo and the token isn't saved
anywhere):

1. Create a new token at
   <https://socket.dev/dashboard/org/nonamenoorg/settings/integrations/api-tokens>
   with permission to create full scans and read reports.
2. Run this in WSL and paste the token when asked. It isn't shown or saved; it stays in memory only and is
   written to the Actions and Dependabot secrets of every repo listed. Add new repos to the list.

   ```bash
   read -rsp "Socket token: " T; echo; for r in ci emdash-forum emdash-member-auth hcc-web; do for app in actions dependabot; do printf %s "$T" | gh secret set SOCKET_CLI_API_TOKEN -R jmatis/$r --app $app && echo "set $r ($app)"; done; done; unset T
   ```

3. Check you got a `set …` line for every repo and both places, then delete the old token in Socket.
4. Re-run a scan to confirm, for example `gh workflow run test.yml -R jmatis/ci`.
