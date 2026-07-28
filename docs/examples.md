# Configuration Examples 🧪

Use the full commit SHA and matching version from the [latest Fence release](https://github.com/openai/fence/releases). Run Fence before checkout and the rest of your job.

## Default Block Mode

No extra configuration is needed:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
```

Fence blocks unexpected outbound traffic and disables passwordless `sudo` and Docker.

## Audit A Workflow

See what your workflow needs without blocking it:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    mode: audit
```

Review the job summary, add the recommended hostnames or IP addresses to your allowlist, and switch back to block mode.

## Allow HTTPS Destinations

Bare hostnames default to TCP port `443`:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      api.example.com
      artifacts.example.com
```

## Use Custom Ports And Protocols

Mix hostnames, ports, IP addresses, and CIDR networks:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      registry.example.com:8443
      tcp://cache.example.com:9443
      udp://dns.example.com:53
      ip 192.0.2.10 tcp 443
      ip 2001:db8::10 udp 53
      cidr 192.0.2.0/24 udp 123
      cidr 2001:db8::/64 tcp 443
```

See the [allowlist guide](allowlist.md) for entry limits and validation rules.

## Upload GitHub Artifacts

Enable GitHub storage access when a job needs artifacts, GitHub Pages, or caches:

```yaml
jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: openai/fence@<commit-sha> # pin@vX.Y.Z
        with:
          allow_github_artifacts: true

      - uses: actions/checkout@<checkout-commit-sha>

      - run: script/build

      - uses: actions/upload-artifact@<upload-artifact-commit-sha>
        with:
          name: build-output
          path: dist/
```

> [!IMPORTANT]
> Artifact uploads can move data off the runner. Enable this option only when the job needs it.

## Use Docker

Keep container access available when your workflow requires Docker or containerd:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    container_policy: unsafe_preserve
    allowlist: |
      auth.docker.io
      registry-1.docker.io
```

Image pulls can require additional registry, layer, or storage domains. Start with audit mode if you need help identifying them.

> [!WARNING]
> Preserving Docker access weakens runner isolation.

## Use A Hostname Wildcard

Allow exactly one or two hostname levels:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      *.docker.io
      *.*.example.com
```

`*.docker.io` matches `auth.docker.io`, but not `docker.io` or `one.two.docker.io`. All wildcard entries share a limit of eight matched hostnames.

## Restrict Optional GitHub Domains

Remove optional GitHub web, API, release, and application destinations:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    disable_broad_github_domains: true
```

Core GitHub Actions reporting still works. Enable this setting only when later steps do not need the excluded GitHub services.

## Set The Platform Profile

Fence selects the supported platform profile automatically. If you need to set it explicitly:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    platform_profile: github_hosted_workflow_bootstrap_v5
```

Other profile values are rejected.

## Use Raw JSON

The `config` input is intended for advanced use:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    config: >-
      {"schema_version":1,"mode":"block","invocation_id":"my-job-1","allowlist":[]}
```

Do not combine `config` with native inputs such as `allowlist` or `mode`. Most workflows should use native inputs and let Fence create its own invocation ID.

For the complete configuration contract, see the [v0 specification](v0.md#configuration-interface).
