# Local development of Agent Sandbox

This devcontainer is used for development of Agent Sandbox. It also acts as a bootstrap for continuously improving the Claude Code image we publish.

## Files

- `Dockerfile` - Debian bookworm base with dev tools, zsh, Claude Code, and firewall utilities
- `devcontainer.json` - VS Code devcontainer config with mounts and capabilities
- `init-firewall.sh` - Network lockdown script that runs at container start

## How It Works

On container start, `init-firewall.sh` runs with sudo and:

1. Flushes existing iptables rules (preserving Docker DNS)
2. Creates an ipset of allowed IP addresses
3. Fetches GitHub IP ranges dynamically from api.github.com/meta
4. Resolves other allowed domains (npm, Anthropic API, VS Code, etc.)
5. Sets default OUTPUT policy to DROP
6. Allows only traffic to the ipset destinations
7. Verifies the firewall blocks example.com and allows api.github.com

The container requires `CAP_NET_ADMIN` and `CAP_NET_RAW` capabilities for iptables manipulation.

## First-Time Setup

Claude Code needs OAuth authentication on first run. From your **host terminal**:

```bash
docker exec -it <container-name> zsh -i -c 'claude'
```

Follow the OAuth flow, then `/exit`. Credentials persist in the `.claude` volume.

## Adding Allowed Domains

Edit `init-firewall.sh` and add your domain to the `for domain in` loop:

```bash
for domain in \
    "registry.npmjs.org" \
    "api.anthropic.com" \
    "your-new-domain.com" \  # Add here
    ...
```

Then rebuild the container (Command Palette > Dev Containers: Rebuild Container).

## Troubleshooting

**Connection refused / timeouts**: The domain is probably not in the allowlist. Check `init-firewall.sh`.

**Firewall verification failed on start**: DNS resolution or GitHub API fetch failed. Check your host network connection.

**Claude auth issues**: Run auth from host terminal, not VS Code integrated terminal. See First-Time Setup above.
