# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Sandbox creates locked-down local sandboxes for running AI coding agents (Claude Code, Codex, etc.) with minimal filesystem access and restricted outbound network. It enforces network allowlisting via iptables/ipset rules at container startup.

## Development Environment

This project uses VS Code devcontainers. The container runs Debian bookworm with:
- Non-root `dev` user (uid/gid 500)
- Zsh with powerline10k theme
- Network lockdown via `init-firewall.sh` at container start

### Key Paths Inside Container
- `/workspace` - Your repo (bind mount)
- `/home/dev/.claude` - Claude Code state (named volume, persists per-project)
- `/commandhistory` - Bash/zsh history (named volume)

### Useful Aliases
- `claude-yolo` - Runs `claude --dangerously-skip-permissions` from /workspace

## Network Allowlist

The firewall (`init-firewall.sh`) blocks all outbound by default. Allowed destinations:
- GitHub (web, api, git) - fetched dynamically from api.github.com/meta
- registry.npmjs.org
- api.anthropic.com
- storage.googleapis.com
- sentry.io, statsig.anthropic.com, statsig.com
- VS Code marketplace and update servers

To add a domain: edit `init-firewall.sh`, add to the `for domain in` loop, rebuild container.

## Architecture

Three planned components (some still scaffolded):

1. **Images** (`images/`) - Base image + agent-specific images
2. **Runtime** (`runtime/`) - Docker Compose stack with proxy enforcement
3. **Devcontainer templates** (`devcontainer/templates/`) - `minimal` and `proxy-locked` variants

Current development focus is on the devcontainer in `.devcontainer/` which uses direct iptables rules rather than the proxy approach.

## Key Principles

- **Security-first**: Changes must maintain or improve security posture. Never bypass firewall restrictions without explicit user request.
- **Reproducibility**: Pin images by digest, not tag. Prefer explicit configs over defaults.
- **Agent-agnostic**: Core changes should support multiple agents. Agent-specific logic belongs in agent-specific images.
- **Policy-as-code**: Network policies and configs should be reviewed like source code.

## Testing Firewall Changes

The `init-firewall.sh` script includes verification at the end. After any network policy change:
1. Rebuild container
2. Script auto-verifies that example.com is blocked and api.github.com is reachable
3. Manually test your new allowed domain works

## Target Platform

Primary: Colima on Apple Silicon (macOS). Should work on any Docker-compatible runtime.
