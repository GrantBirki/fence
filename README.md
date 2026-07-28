# Fence 🛡️

[![lint](https://github.com/openai/fence/actions/workflows/lint.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/lint.yml)
[![test](https://github.com/openai/fence/actions/workflows/test.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/test.yml)
[![build](https://github.com/openai/fence/actions/workflows/build.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/build.yml)
[![acceptance](https://github.com/openai/fence/actions/workflows/acceptance.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/acceptance.yml)
[![action acceptance](https://github.com/openai/fence/actions/workflows/action-acceptance.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/action-acceptance.yml)
[![released action / ubuntu-24.04 + ubuntu-latest](https://github.com/openai/fence/actions/workflows/action-drift-canary.yml/badge.svg?branch=main&event=schedule)](https://github.com/openai/fence/actions/workflows/action-drift-canary.yml?query=branch%3Amain+event%3Aschedule)
[![main / ubuntu-latest](https://github.com/openai/fence/actions/workflows/action-acceptance-ubuntu-latest.yml/badge.svg?branch=main&event=schedule)](https://github.com/openai/fence/actions/workflows/action-acceptance-ubuntu-latest.yml?query=branch%3Amain+event%3Aschedule)
[![integration](https://github.com/openai/fence/actions/workflows/integration.yml/badge.svg)](https://github.com/openai/fence/actions/workflows/integration.yml)

A GitHub Action for egress filtering and runner lockdown.

![Fence](./docs/assets/fence.png)

## Quick Start ⚡

Add Fence as the first step in a GitHub-hosted Linux job:

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: openai/fence@<commit-sha> # pin@vX.Y.Z
      - uses: actions/checkout@<checkout-commit-sha>
      - run: script/test
```

Use the full commit SHA and matching version comment from the [latest release](https://github.com/openai/fence/releases). Fence also supports `ubuntu-latest`.

By default, Fence:

- Blocks outbound network requests unless they are needed by the runner or allowed by your configuration.
- Turns off passwordless `sudo`.
- Turns off Docker and container access.
- Adds a network activity report to the GitHub job summary. (and to logs)

Check out the [getting started guide](docs/getting-started.md) for a full walkthrough.

## Allow A Destination 📝

Use `allowlist` to give your workflow access to the destinations it needs:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      api.example.com
      registry.example.com:8443
      udp://dns.example.com:53
      ip 192.0.2.10 tcp 443
      cidr 192.0.2.0/24 udp 123
```

Bare hostnames default to TCP port `443`. Fence also supports IPv6, custom ports, TCP, UDP, CIDR ranges, and one or two-level hostname wildcards. An allowlist can contain up to 64 unique entries.

See [allowlist syntax](docs/allowlist.md) for all supported formats.

## Try Audit Mode 🔍

Not sure what your workflow needs? Start with audit mode:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    mode: audit
```

Audit mode reports what Fence would block without changing network, `sudo`, or Docker access. Use the job summary to build your allowlist, then switch back to the default block mode.

## GitHub Artifacts And Pages 📦

GitHub artifact uploads, GitHub Pages, and caches can require additional storage access. Enable it when your workflow needs it:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allow_github_artifacts: true
```

> [!IMPORTANT]
> Artifact uploads can move data off the runner. This setting is disabled by default and slightly reduces Fence's security guarantees. Enable it only for jobs that need artifacts, Pages, or caches.

## Docker 🐳

Fence disables Docker by default. If your workflow needs containers, you can keep Docker available:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    container_policy: unsafe_preserve
```

> [!WARNING]
> Docker access weakens runner isolation. Use `unsafe_preserve` only when containers are required. This setting is explicitly telling you that it is "unsafe" and you will be "preserving" privileged Docker access on the runner.

## How It Works 🔧

Fence runs before the rest of your job and sets the rules for what the runner can reach.

1. Checks that the runner and the bundled Fence agent are supported.
2. Allows required GitHub and runner connections plus your `allowlist`.
3. Blocks other outbound requests and turns off passwordless `sudo` and Docker.
4. Monitors those protections until the job ends.
5. Reports network activity and fails the job if a protection is unexpectedly changed.

See [how Fence works](docs/how-it-works.md) for more detail.

## Network Reports 📋

Fence adds a network activity table to the job summary and post-job log. It also writes one machine-readable `FENCE_REPORT_JSON=` line so you can retrieve the report with the GitHub API:

```bash
gh api repos/OWNER/REPO/actions/runs/RUN_ID/jobs \
  --jq '.jobs[] | {id, name}'

gh api repos/OWNER/REPO/actions/jobs/JOB_ID/logs \
  | sed -n 's/^.*FENCE_REPORT_JSON=//p' \
  | jq .
```

The report includes allowed or blocked destinations, observed network activity, relevant warnings, and suggested allowlist entries in audit mode. Anyone with access to the job log can read it.

See [network reports](docs/how-it-works.md#network-reports) for a complete, readable example.

## Security Notes 🔒

Fence helps prevent later workflow steps from sending data to unexpected destinations or undoing the runner's network restrictions. It is not a full sandbox.

- **Supported runners:** GitHub-hosted x64 jobs using `ubuntu-24.04` or `ubuntu-latest`. Releases are validated against `ubuntu-24.04`, and `ubuntu-latest` is regularly tested.
- **Built-in connections:** GitHub Actions, job reporting, and the hosted runner still need a small set of GitHub and Azure platform connections. Those destinations remain reachable.
- **Azure platform:** Azure Instance Metadata Service remains reachable at `169.254.169.254:80`. Azure WireServer access is limited to root-owned host processes.
- **Artifacts:** `allow_github_artifacts: true` allows a limited GitHub storage channel and should only be used when needed.
- **Docker:** `unsafe_preserve` keeps containers available but reduces isolation.
- **Release pinning:** Use the full commit SHA from a published Fence release. Do not run the Action from `main`.

See the [security guide](docs/security.md) for more details.

## Further Reading 📚

- [Getting started](docs/getting-started.md)
- [Configuration examples](docs/examples.md)
- [Allowlist syntax](docs/allowlist.md)
- [How Fence works](docs/how-it-works.md)
- [Security guide](docs/security.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Release provenance](docs/release-provenance.md)
- [Local development](docs/development.md)
- [CLI reference](docs/cli.md)
- [Security policy](SECURITY.md)
- [Fence v0 specification](docs/v0.md)
- [Threat model](docs/threat-model.md)
- [Security review](docs/security-review.md)
- [Implementation history](docs/history.md)
- [Repository settings](docs/repository-settings.md)
- [Hermetic builds](https://software.birki.io/posts/hermetic-builds/)

## License ⚖️

Fence is released under the [MIT License](LICENSE).
