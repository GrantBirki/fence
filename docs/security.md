# Security Guide 🔒

Fence limits where a GitHub Actions job can send network traffic and removes common ways to undo that restriction. It reduces the runner's attack surface; it does not turn the job into a sandbox.

## Supported Runners

Fence supports GitHub-hosted x64 jobs using `ubuntu-24.04` or `ubuntu-latest`.

- `ubuntu-24.04` is the fixed runner image used to validate releases.
- `ubuntu-latest` is regularly tested, but its image can change.
- Self-hosted runners, container jobs, other architectures, Windows, and macOS are not supported.

Fence checks the runner before applying its controls. If the runner does not match the supported configuration, setup fails instead of silently weakening protection.

Run Fence before checkout and any other steps you want it to protect.

## Protection Modes

- **Block:** Allows required platform connections and your allowlist, blocks other outbound requests, and disables passwordless `sudo` and Docker. This is the default.
- **Audit:** Reports what block mode would deny without blocking traffic or disabling `sudo` and Docker.
- **Preserved containers:** `container_policy: unsafe_preserve` keeps Docker and containerd available. Network filtering stays active, but runner isolation is weaker.

Fence does not automatically switch from block mode to audit or preserved containers.

## Built-In Network Access

GitHub-hosted jobs need some network access to run, report their status, and finish. Fence keeps a limited set of GitHub and Azure platform destinations available for those tasks.

The default policy includes GitHub web, API, release, and reporting services. Set `disable_broad_github_domains: true` to remove optional GitHub destinations when your job does not need them. Core Actions reporting remains available.

Fence also handles GitHub's job-reporting storage without allowing all Azure Blob Storage. GitHub artifacts, Pages, and caches require `allow_github_artifacts: true` when they need additional storage access.

> [!IMPORTANT]
> Anything the job is allowed to reach can also be used to send data. Keep allowlists narrow and enable artifact access only when needed.

## Azure Platform Services

GitHub-hosted runners depend on Azure platform services:

- **Azure Instance Metadata Service (IMDS):** `169.254.169.254:80` remains reachable from host and forwarded traffic, including later workflow steps.
- **Azure WireServer:** `168.63.129.16` on TCP ports `80` and `32526` is available only to root-owned host processes.

These are platform rules, not entries you need to add to `allowlist`. Fence does not claim to block IMDS.

## GitHub Artifacts

`allow_github_artifacts: true` lets approved runner-owned processes reach a small number of GitHub-shaped storage accounts over HTTPS. It is disabled by default.

Once a storage account has been allowed, later workflow steps can reach it too. Fence cannot inspect encrypted upload contents or prove that an account name belongs to GitHub. Treat artifact uploads as an intentional outbound data channel.

## Runtime Integrity

Fence verifies the bundled agent, runs it from a protected root-owned location, and continuously checks the runner's network and privilege controls.

If a required control changes, Fence records the failure and fails the job. It does not restore network, `sudo`, or Docker access; GitHub discards the hosted runner when the job ends.

## Reports And Privacy

Fence does not upload telemetry. The job summary and logs may include destination hostnames, IP addresses, process names, process IDs, control status, and warnings.

Reports do not include credentials, environment variables, full executable paths, command arguments, or packet contents. Anyone with permission to read a workflow job log can read its Fence report.

See [network reports](how-it-works.md#network-reports) for the report format and GitHub API example.

## Release Pinning

Pin the full `action_commit` SHA from a [published release](https://github.com/openai/fence/releases):

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
```

Do not reference `main`; it contains source code rather than the production Action bundle. See [release provenance](release-provenance.md) for the signed source, bundled artifact, and verification model.

For the complete security contract, see the [v0 specification](v0.md), [threat model](threat-model.md), and [security review](security-review.md). Report vulnerabilities using the [security policy](../SECURITY.md).
