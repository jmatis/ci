# Docs

**What it's for:** keeps a library's API docs (generated from its code comments) in step with the code.
The docs are Markdown committed in the repo, so they show on GitHub even for private repos, where
GitHub Pages isn't available on the free plan.

**How it works** (`.github/workflows/docs.yml`):
1. Fetches the pushed commit using plain `git` (no third-party actions).
2. Builds the image from this folder's `Dockerfile`, which pins the Node image by version and digest.
   Dependabot opens a pull request when a newer release is at least 7 days old.
3. Runs `npm ci --ignore-scripts` and the repo's own `npm run docs`, so the doc generator's version comes
   from the repo's lockfile.
4. Checks that nothing in `docs/api` (or the `path` input) changed, was added or was deleted.

**Fails when:** the code's exports or comments changed but the docs weren't regenerated.
The fix is always the same: run `npm run docs` and commit the result.

**Setting up a repo** (TypeDoc, as in emdash-forum and emdash-member-auth):
1. `npm install -D typedoc typedoc-plugin-markdown`
2. Add `"docs": "typedoc"` to the `scripts` in `package.json`.
3. Add a `typedoc.json` listing the entry points the package exports. Keep `"disableSources": true`:
   source links carry line numbers, so every unrelated edit would make the docs out of date.
4. Run `npm run docs`, commit `docs/api`, and add the workflow below.

**Typical run:** ~30 seconds (1 billed minute).

## Example

```yaml
# .github/workflows/docs.yml in the repo to check
name: docs

on:
  push:
  workflow_dispatch:

jobs:
  docs:
    uses: jmatis/ci/.github/workflows/docs.yml@main
```
