# Security Policy

## Supported Versions

Security fixes land on `main` and in the latest stable release.

Fence supports GitHub-hosted x64 runners using `ubuntu-24.04` or `ubuntu-latest`. Releases are validated on `ubuntu-24.04`, and `ubuntu-latest` is tested regularly. Fence does not support self-hosted runners, container jobs, or other operating systems and architectures.

Audit mode observes network activity without blocking it. `container_policy: unsafe_preserve` leaves Docker and containerd available, which weakens runner isolation.

See the [security guide](docs/security.md), [v0 specification](docs/v0.md), and [threat model](docs/threat-model.md) for the full security model.

## Reporting A Vulnerability

Report vulnerabilities through [GitHub private vulnerability reporting](https://github.com/openai/fence/security/advisories/new). If it is unavailable, contact the repository maintainer directly.

> [!IMPORTANT]
> Do not open a public issue containing exploit details for an unresolved vulnerability.

## Dependency Policy

- Pin direct Cargo dependencies to exact versions.
- Commit `Cargo.lock` and the vendored dependencies in `vendor/cache`.
- Use `script/update` for dependency updates.
- Use `script/vendor-rust` for Rust toolchain updates.
- Use `script/vendor-update-tools`, `script/vendor-release-tools`, and `script/vendor-test-tools` to update their pinned tools.
- Keep routine builds and tests offline after the Rust toolchain is prepared.

## Offline Development

Prepare the Rust toolchain, then run the normal project commands:

```console
script/prepare-rust
script/bootstrap
script/test
script/lint
script/build
```

`script/prepare-rust` downloads and verifies the pinned Rust toolchain. The remaining commands run offline using checked-in dependencies.

GitHub-hosted jobs are not completely air-gapped. Checkout, runner setup, artifact uploads, releases, and attestation verification still use GitHub's network services.

## Verify Release Artifacts

Verify a published release with its checksums and attestations:

```console
shasum -a 256 -c checksums.txt

gh attestation verify <artifact> \
  --repo openai/fence \
  --signer-workflow openai/fence/.github/workflows/release.yml
```

On Linux, use `sha256sum -c checksums.txt` if `shasum` is unavailable.

Releases through `v0.8.3` predate the repository transfer. Verify them against the repository and signer in their original attestations.

See [release provenance](docs/release-provenance.md) for the source commit, distribution commit, and published Action pin.
