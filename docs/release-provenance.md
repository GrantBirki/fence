# Release Provenance 🔏

Fence releases connect a reviewed source commit to the exact GitHub Action you run. A reviewed version bump authorizes the release; releases cannot be started manually.

## How A Release Is Built

1. A pull request updates the source version in `Cargo.toml` and `Cargo.lock`.
2. The pull request merges into protected `main` as signed source commit `M`.
3. Release automation builds and attests the Linux agent from `M`.
4. GitHub creates a signed distribution commit `D` with `M` as its only parent.
5. The Action acceptance tests and runner canary validate `D`.
6. The workflow publishes and verifies an immutable release.

The distribution commit adds exactly two generated files:

```text
action/bin/fence
action/bundle-manifest.json
```

Those files are not added to `main`.

## Pin The Published Action

Each release includes an `action-release.json` asset that identifies the reviewed source commit, distribution commit, binary checksum, manifest version, and signing workflow.

Use its full `action_commit` value in your workflow:

```yaml
- uses: openai/fence@<action-commit> # pin@vX.Y.Z
```

Use the version from the same release in the `# pin@vX.Y.Z` comment. The release notes contain a ready-to-copy example, and Dependabot can use the version comment when updating the pinned commit.

The release tag identifies the distribution commit, but workflows should use the full commit SHA. Do not run Fence from `main`; it does not contain the production Action bundle.

## Verify Release Artifacts

Fence verifies the binary, bundled manifest, artifact checksums, build attestations, and signed `D -> M` commit relationship before considering a release complete. The Action runs its verified bundled binary and never downloads an agent or policy at runtime.

Historical releases through `v0.6.3` keep their original tag behavior. Releases through `v0.8.3` retain their original `GrantBirki/fence` provenance.

See the [security policy](../SECURITY.md) for artifact verification commands and the [v0 specification](v0.md#ci-and-release-contract) for the full release contract.
