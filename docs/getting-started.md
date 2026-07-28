# Getting Started 🚀

Fence is a GitHub Action that limits outbound network access and locks down your runner. Run it before checkout and any other steps you want to protect.

## Add Fence To Your Workflow

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: openai/fence@<commit-sha> # pin@vX.Y.Z

      - uses: actions/checkout@<checkout-commit-sha>

      - name: Run tests
        run: script/test
```

Replace `<commit-sha>` and `vX.Y.Z` with the full commit and version from a [published release](https://github.com/openai/fence/releases). The release notes include a ready-to-copy pin. Do not reference `main`; it does not include the bundled Action.

Fence supports GitHub-hosted x64 jobs using `ubuntu-24.04` or `ubuntu-latest`. `ubuntu-24.04` is the fixed image used for release validation. `ubuntu-latest` is regularly tested, but its image can change.

## What Happens By Default?

With no extra configuration, Fence:

- Allows the GitHub and runner connections needed to finish the job.
- Blocks other outbound network requests.
- Turns off passwordless `sudo`.
- Turns off Docker and container access.
- Reports network activity in the job summary.

You do not need to list GitHub's job-reporting or storage domains yourself.

## Allow A Domain

Add the destinations your workflow actually needs:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      api.example.com
      registry.example.com:8443
```

Bare hostnames default to TCP port `443`. See the [allowlist guide](allowlist.md) for other ports, IP addresses, CIDR ranges, and wildcards.

## Start With Audit Mode

If you do not know which destinations to allow, use audit mode first:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    mode: audit
```

Audit mode shows what Fence would block without blocking traffic or disabling `sudo` and Docker. Review the job summary, add the required destinations to your allowlist, and then remove `mode: audit` to enable blocking.

## Upload Artifacts Or Deploy Pages

Artifact uploads, GitHub Pages, and caches sometimes need extra access to GitHub storage:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allow_github_artifacts: true
```

> [!IMPORTANT]
> This setting allows a limited storage channel that later workflow steps can also use. Leave it off unless your job needs artifacts, Pages, or caches.

## Use Docker

If your workflow requires Docker or containerd, preserve container access explicitly:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    container_policy: unsafe_preserve
```

> [!WARNING]
> Preserving container access weakens Fence's runner isolation.

For more examples, see [configuration examples](examples.md), [allowlist syntax](allowlist.md), and [troubleshooting](troubleshooting.md).
