# Troubleshooting 🔍

Start with the Fence job summary. It shows network activity, blocked destinations, warnings, and the final state of the runner's security controls.

## Fence Fails To Start

Check that the job uses a GitHub-hosted x64 runner with `ubuntu-24.04` or `ubuntu-latest`. Fence does not support self-hosted runners, container jobs, or other architectures.

Make Fence the first step. Checkout, setup actions, and other commands can change the runner before Fence has verified it.

If you need more detail, set the standard `ACTIONS_STEP_DEBUG` repository secret to `true` and rerun the job.

## A Network Request Is Blocked

Temporarily switch to audit mode:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    mode: audit
```

Check the job summary, add the required hostname or IP address to your allowlist, and return to block mode. Use the narrowest destination, protocol, and port that works.

If an allowed hostname still fails, check whether the service uses redirects, a CDN, a separate storage domain, or a non-default port.

## Artifact Uploads Or Pages Fail

Artifact uploads, GitHub Pages, and caches can need extra GitHub storage access:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allow_github_artifacts: true
```

Enable this only for jobs that need it. Do not allowlist all Azure Blob Storage or add changing GitHub storage domains by hand.

## Docker Does Not Work

Fence disables Docker and containerd by default. Keep them available only when your workflow requires containers:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    container_policy: unsafe_preserve
```

This weakens runner isolation. Image pulls may also need registry, authentication, CDN, or storage destinations in your allowlist.

## The Job Reports Critical Drift

Critical drift means a required protection changed or a monitored Fence component stopped working. Fence fails the job because it can no longer verify the original security guarantees.

Check the job summary and debug logs for the affected control. Do not work around the failure by adding a broader allowlist.

## The Agent Cannot Run Directly

`fence check-support` and `fence render-plan` are inspection commands. Running `fence run` directly fails with `trusted_launcher_required` because production protection must start through the GitHub Action.

See the [CLI reference](cli.md), [security guide](security.md), and [allowlist guide](allowlist.md) for more detail.
