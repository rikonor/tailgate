# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tailgate is an ingress gateway automation system for home lab services. It provisions a Vultr VPS running Caddy as a reverse proxy, connected to a Tailscale mesh network. This allows public HTTPS traffic to route through the VPS to backend services that only exist on the private tailnet.

**Architecture:** Public HTTPS → Caddy (Vultr VPS) → Tailscale → Backend services (hidden)

## Commands

All automation is via `mise` tasks defined in `mise.toml`:

```bash
mise install                # Install tools (vultr-cli, mermaid-cli)
mise run provision          # Create Vultr VPS
mise run wait-ready         # Poll until instance is active
mise run configure          # SSH setup: apt update, BBR, Tailscale, Caddy, UFW
mise run tailscale-up       # Authenticate VPS to tailnet (requires TS_AUTHKEY)
mise run setup-caddy        # Deploy Caddyfile
mise run verify             # Health checks (SSH, Tailscale, Caddy, HTTP)
mise run ssh                # Interactive SSH to VPS
mise run dns-check          # Dry-run DNS changes
mise run dns-update         # Apply DNS via Porkbun API
mise run mermaid:check      # Validate Mermaid diagrams
mise run mermaid:render     # Render .mmd to .svg
mise run destroy            # Delete VPS (destructive)

# Tailnet diagnostics (file-based tasks in .mise/tasks/)
mise run tailnet:status     # List all tailnet devices
mise run tailnet:routes     # Show subnet route advertisers
mise run tailnet:ping-lan   # Ping LAN devices via subnet routing
mise run nas:status         # Check Tailscale on Synology NAS
mise run nas:reauth         # Re-authenticate NAS Tailscale (interactive)
```

## Configuration

Key environment variables in `mise.toml`:
- `VPS_ID` / `VPS_IP`: Set after provisioning
- `TS_AUTHKEY`: Required for `tailscale-up` (from https://login.tailscale.com/admin/settings/keys)
- `PORKBUN_API_KEY` / `PORKBUN_SECRET_KEY`: Required for DNS tasks

For tailnet diagnostics:
- `JUMPBOX_IP`: Media server Tailscale IP (for SSH tunneling to NAS)
- `LAN_HOSTS`: Comma-separated LAN IPs to ping
- `NAS_LAN_IP` / `NAS_USER` / `NAS_SUBNET`: NAS configuration
- `SYNOLOGY_PASS`: Set in environment (not in file) for NAS SSH access

Backend services use Tailscale MagicDNS hostnames (e.g., `synapse`, `ricon-family-website`).

## Architecture Notes

- VPS: Ubuntu 24.04 LTS on Vultr (Atlanta, vhf-1c-1gb plan)
- Reverse proxy: Caddy with automatic HTTPS
- Network: TCP BBR congestion control enabled
- Firewall: UFW with ports 22, 80, 443, 8448 (Matrix federation)
- Diagrams: Source of truth is `docs/*.mmd`, rendered to SVG

The Caddyfile is embedded in the `setup-caddy` task and includes Matrix well-known endpoints for federation support.
