# .github

Shared GitHub configuration for the `brno01` repositories: reusable workflows and the default community files.

## What is here

| Path | What it is |
|---|---|
| `.github/workflows/security.yml` | Reusable security scan: secrets, vulnerable dependencies, static analysis, shell scripts and the workflows themselves. |
| `.github/workflows/claude.yml` | Reusable on-demand reviewer. Answers `@claude` comments from the repository owner and audits a repository on request. |
| `.github/workflows/release.yml` | Reusable release: creates the GitHub Release for a version tag with generated notes. |
| `templates/` | The small caller files to copy into a repository, plus the license texts. |
| `SECURITY.md`, `.github/PULL_REQUEST_TEMPLATE.md` | Defaults GitHub applies to every repository of the account that does not have its own. |

## Security scan

One job runs five checks, so a private repository spends about two billed minutes per run:

| Check | Tool | Per-repository configuration |
|---|---|---|
| Secrets | [gitleaks](https://github.com/gitleaks/gitleaks) | `.gitleaks.toml`, `.gitleaksignore` |
| Vulnerable dependencies | [osv-scanner](https://github.com/google/osv-scanner) | `osv-scanner.toml` |
| Static analysis | [semgrep](https://github.com/semgrep/semgrep) | `.semgrepignore`, `# nosemgrep` comments |
| Shell scripts | [shellcheck](https://github.com/koalaman/shellcheck) | `.shellcheckrc`, `# shellcheck disable=` comments |
| Workflows | [actionlint](https://github.com/rhysd/actionlint), [zizmor](https://github.com/zizmorcore/zizmor) | `.github/actionlint.yaml`, `zizmor.yml` |

Pull requests and pushes scan only the new commits for secrets; the weekly run and manual runs scan the whole history. Every check runs even when an earlier one fails, and the job summary shows the result of each.

When the scan fails on the default branch or in the weekly run, the workflow opens an issue labelled `security-scan`, or updates the one that is open, and closes it when the scan passes again. The issue names the failing checks and links the run; the findings stay in the run log.

Use it by copying `templates/security.yml` to `.github/workflows/security.yml`:

```yaml
jobs:
  security:
    permissions:
      contents: read
      issues: write
    uses: brno01/.github/.github/workflows/security.yml@v1
```

Inputs:

| Input | Default | Meaning |
|---|---|---|
| `semgrep-config` | `p/default` | Semgrep rule set. |
| `shellcheck-severity` | `warning` | Lowest shellcheck severity that fails the scan. |
| `report-only` | `false` | Show findings in the summary without failing the job. |
| `open-issue` | `true` | Open or update the `security-scan` issue. |

The scanners are downloaded at fixed versions and checked against their published SHA-256 sums. Third-party actions are pinned to a commit.

## On-demand reviewer

Copy `templates/claude.yml` and add the `CLAUDE_CODE_OAUTH_TOKEN` secret to the repository (create the token with `claude setup-token`).

- Write `@claude` in a comment on a pull request or an issue and Claude answers there.
- Run the workflow by hand and Claude audits the repository, opening at most five issues labelled `claude-audit` and skipping what is already reported.

Only the repository owner can trigger it, and it never runs by itself. It has read access to the code and can only write comments and issues.

## Release

Copy `templates/release.yml`. Pushing a tag such as `v1.2.3` creates the GitHub Release with generated notes. Use the `build-command` and `assets` inputs to build and attach files.

## Versioning

Callers reference `@v1`. The `v1` tag moves forward with compatible changes; a breaking change gets `v2`.

## License

[MIT](LICENSE)
