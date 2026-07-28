# Allowlist Guide 📝

Use the `allowlist` input to give your workflow access to specific network destinations:

```yaml
- uses: openai/fence@<commit-sha> # pin@vX.Y.Z
  with:
    allowlist: |
      api.example.com
      registry.example.com:8443
      udp://dns.example.com:53
```

Each line adds one destination. Blank lines and comments starting with `#` are ignored.

## Supported Formats

```text
# HTTPS hostname
example.com

# Hostname with a custom TCP port
example.com:8443
tcp://example.com:443

# Hostname with a UDP port
udp://dns.example.com:53

# Explicit hostname
hostname example.com tcp 443

# Hostname wildcards
*.example.com
*.*.example.com

# IPv4 and IPv6 addresses
ip 192.0.2.10 tcp 443
ip 2001:db8::10 udp 53

# IPv4 and IPv6 networks
cidr 192.0.2.0/24 udp 123
cidr 2001:db8::/64 tcp 443
```

A hostname without a port uses TCP port `443`. Use the `ip` or `cidr` format for IP addresses and networks, especially IPv6.

## Entry Limit

Fence accepts up to 64 unique, normalized entries. Repeating a hostname, changing its capitalization, or writing the same IP address another way does not create an extra entry. Different ports, protocols, or destination types do count separately.

Fence rejects the 65th unique entry before changing the runner.

> [!NOTE]
> The advanced JSON `config` input has its own 64-entry validation. Do not combine `config` with native Action inputs.

## Wildcards

Wildcards match exactly one hostname level per `*`:

- `*.example.com` matches `api.example.com`.
- `*.example.com` does not match `example.com` or `one.two.example.com`.
- `*.*.example.com` matches `one.two.example.com`.
- `*.*.example.com` does not match `api.example.com` or `three.one.two.example.com`.

All wildcard entries share a limit of eight matched hostnames for the job. Use exact hostnames when you can; a broad wildcard gives a workflow more places to send data.

## CIDR Networks

CIDR entries must use a real network address:

```text
# Valid
cidr 192.0.2.0/24 udp 123

# Invalid: host bits are set
cidr 192.0.2.1/24 udp 123
```

Fence rejects invalid entries before applying any runner changes.

## GitHub Storage

You do not need to add GitHub's job-reporting storage accounts to your allowlist. Fence handles the small set of accounts required by the GitHub runner.

If your job needs GitHub artifacts, Pages, or caches, use `allow_github_artifacts: true` instead of allowlisting storage domains. This setting is disabled by default because it gives later workflow steps a limited storage channel.

See the [configuration examples](examples.md) for complete workflows or the [v0 specification](v0.md#effective-policy) for the full policy.
