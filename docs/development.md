# Local Development 🛠️

Fence keeps its Rust dependencies vendored so normal development commands can run without network access after the toolchain is prepared.

## Get Started

```console
script/prepare-rust
script/bootstrap
script/test
script/lint
script/build
```

`script/prepare-rust` is the only networked step in the normal setup. It downloads the pinned Rust toolchain and verifies its checksums before installation.

After preparation, `script/bootstrap`, `script/test`, `script/lint`, and `script/build` run offline using the checked-in toolchain locks and vendored Cargo dependencies.

## Update Dependencies

Use the repository's update scripts when you intentionally need network access:

- `script/update` updates Rust dependencies.
- `script/vendor-rust` updates the pinned Rust toolchain.
- `script/vendor-update-tools` updates dependency-audit tools.
- `script/vendor-release-tools` updates retained release tools.
- `script/vendor-test-tools` updates retained test tools.

Do not install or download tools as a side effect of a build or test.

## Build The GitHub Action

`script/assemble-action-bundle` builds a production-shaped Action bundle from an existing agent artifact. It does not download an agent or policy.

The `main` branch intentionally does not contain generated Action binaries. Release automation adds the agent and its manifest to a separate, signed distribution commit.

## Temporary Files

Local builds and tool preparation use ignored paths under `target/tmp` unless `TMPDIR` or `RUNNER_TEMP` is already set.

> [!WARNING]
> Do not run destructive hosted-runner or lockdown tests on a developer machine or reusable runner. Those scripts are intended for disposable GitHub-hosted CI jobs.

For the complete build and test contracts, see [AGENTS.md](../AGENTS.md) and the [v0 specification](v0.md#hermetic-repository-contract).
