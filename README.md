# ci

Common continuous integration pipeline: shared GitHub Actions workflows for jmatis repos.

- `.github/workflows/semgrep.yml`: open-source Semgrep scan; the run fails when it finds anything.
  Use it from another repo's `.github/workflows/semgrep.yml`:

  ```yaml
  jobs:
    semgrep:
      uses: jmatis/ci/.github/workflows/semgrep.yml@main
  ```
