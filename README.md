# VPN Stack Infrastructure

This stack provides a WireGuard VPN service with a web UI (wg-easy) and secure HTTPS access via Caddy reverse proxy. The setup is managed with Docker Compose and supports both internal and Cloudflare DNS-based TLS.

---

## Contents
- [Overview](#overview)
- [Directory Structure](#directory-structure)
- [Environment Variables](#environment-variables)
- [Build & Run Instructions](#build--run-instructions)
- [Caddy Configuration](#caddy-configuration)
- [Docker Compose Services](#docker-compose-services)
- [Notes](#notes)

---

## Overview
- **wg-easy**: Provides a web UI for managing WireGuard VPN users and configuration.
- **Caddy**: Handles HTTPS, reverse proxy, and optional Cloudflare DNS-based TLS.
- **Docker Compose**: Orchestrates containers and networks.

---

## Directory Structure
- `run.sh` – Main entry script to configure and run the stack.
- `docker-compose.yml` – Defines the wg-easy service and networks.
- `docker-compose.root-caddy.yml` – Defines the Caddy service and volumes.
- `.env.example` – Example environment variables for configuration.
- `.gitignore` – Files and directories to ignore in git.
- `caddy_global/Caddyfile` – Caddy reverse proxy configuration.
- `caddy_global/Dockerfile` – Dockerfile for building Caddy with Cloudflare DNS plugin.

---

## Environment Variables
Copy `.env.example` to `.env` and adjust as needed:

- `ADMIN_USERNAME`, `ADMIN_PASSWORD`: Web UI admin credentials
- `MULTIPLE_CADDY_GLOBAL`: Set to `true` for multiple global Caddy services
- `VPN_SERVER_ADDRESS`: IP address for the VPN server
- `VPN_SERVER_PORT`: UDP port for WireGuard (default: 42815)
- `VPN_ADMIN_SERVER_ADDRESS`: Domain for Caddy HTTPS access
- `CF_API_TOKEN`: Cloudflare API token for DNS-based TLS
- `HTTP_PORT`, `HTTPS_PORT`: Ports for Caddy (default: 80/443)

---

## Build & Run Instructions

1. Copy and edit environment variables:
   ```sh
   cp .env.example .env
   # Edit .env as needed
   ```
2. Start the stack:
   ```sh
   ./run.sh up -d
   ```
   - The script ensures required Docker networks exist and configures Compose files and TLS based on your environment.
3. To stop the stack:
   ```sh
   ./run.sh down
   ```

---

## Caddy Configuration
- The Caddyfile (`caddy_global/Caddyfile`) sets up a reverse proxy for the wg-easy web UI.
- TLS is configured based on whether the admin server address is an IP (internal TLS) or domain (Cloudflare DNS-based TLS).
- The Caddy Docker image is built with the Cloudflare DNS plugin (see `caddy_global/Dockerfile`).

---

## Docker Compose Services
### docker-compose.yml
- **wg-easy**: WireGuard VPN with web UI
  - Image: `ghcr.io/wg-easy/wg-easy:15`
  - Networks: `wg`, `caddy_net`
  - Volumes: Persistent WireGuard config, host modules
  - Ports: UDP port for VPN
  - Capabilities: `NET_ADMIN`, `SYS_MODULE`
  - Sysctls: IP forwarding, disable IPv6

### docker-compose.root-caddy.yml
- **caddy**: Reverse proxy
  - Build context: `./caddy_global`
  - Ports: 80 (HTTP), 443 (HTTPS)
  - Volumes: Caddyfile, data, config
  - Networks: `caddy_net`

---

## Notes
- `.gitignore` excludes `.env` and `caddy_certs`.
- For production, ensure secrets are set securely and not committed to version control.
- For more details, see comments in `run.sh` and the Compose files.

---

## License
See upstream wg-easy and Caddy repositories for license details.
