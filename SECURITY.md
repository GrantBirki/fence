# Security Policy

## Supported Versions

Security fixes are applied to `main` and the latest stable Fence release.

Fence supports GitHub-hosted x64 runners using `ubuntu-24.04` or `ubuntu-latest`. Each release is validated on `ubuntu-24.04`, while `ubuntu-latest` is regularly tested against the same runner security checks. Self-hosted runners, container jobs, and other operating systems or architectures are outside the supported protection boundary.

Audit mode observes network activity without blocking it. `container_policy: unsafe_preserve` keeps Docker and containerd available and has weaker isolation than default block mode.

See the [security guide](docs/security.md), [v0 specification](docs/v0.md), and [threat model](docs/threat-model.md) for the full security and support boundaries.

## Reporting A Vulnerability

Use [GitHub private vulnerability reporting](https://github.com/openai/fence/security/advisories/new) when it is available. Otherwise, contact the repository maintainer directly.

> [!IMPORTANT]
> Do not open a public issue containing exploit details for an unresolved vulnerability.

## Dependency Policy

- Pin direct Cargo dependencies to exact versions.
- Commit `Cargo.lock` and the vendored dependencies in `vendor/cache`.
- Use `script/update` for dependency updates.
- Use `script/vendor-rust` for Rust toolchain updates.
- Use `script/vendor-update-tools`, `script/vendor-release-tools`, and `script/vendor-test-tools` for their respective pinned tools.
- Keep routine builds and tests offline after the Rust toolchain is prepared.

## Offline Development

Prepare Rust explicitly, then run the standard offline project commands:

```console
script/prepare-rust
script/bootstrap
script/test
script/lint
script/build
```

`script/prepare-rust` verifies the pinned Rust distribution before installing it. The remaining commands use checked-in dependencies and do not download tools.

GitHub-hosted jobs are not completely air-gapped. Checkout, runner preparation, artifact uploads, release publication, and attestation verification still use GitHub network services.

## Verify Release Artifacts

Use the checksums and attestations attached to a published release:

```console
shasum -a 256 -c checksums.txt

gh attestation verify <artifact> \
  --repo openai/fence \
  --signer-workflow openai/fence/.github/workflows/release.yml
```

On Linux, use `sha256sum -c checksums.txt` if `shasum` is unavailable.

Releases through `v0.8.3` were published before the repository transfer. Verify those artifacts against the repository and signer recorded in their original attestations.

See [release provenance](docs/release-provenance.md) for the source commit, distribution commit, and published Action pin.
