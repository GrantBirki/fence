# CLI Reference 💻

Most users should run Fence as a GitHub Action. The Rust agent also provides a small CLI for inspecting its version, checking runner information, and previewing a policy.

```console
fence --version
fence check-support
fence render-plan --config policy.json
fence run --config /run/fence/example/config.json
```

## Print The Version

```console
fence --version
```

Prints the agent version. Source builds use `Cargo.toml`; published Action bundles record their version and provenance in `action/bundle-manifest.json`.

## Check Runner Information

```console
fence check-support
```

Shows the host operating system, architecture, available backend, and expected runner fingerprint. This command does not activate Fence or establish protection.

## Preview A Policy

```console
fence render-plan --config policy.json
```

Validates a JSON configuration and prints the resulting firewall plan without applying it.

## Run The Agent

```console
fence run --config /run/fence/example/config.json
```

Production execution must start through the Fence GitHub Action. Running this command directly fails with `trusted_launcher_required`; that check prevents an ordinary process from claiming it has activated runner protection.

See [how Fence works](how-it-works.md) and the [configuration contract](v0.md#configuration-interface) for more detail.
