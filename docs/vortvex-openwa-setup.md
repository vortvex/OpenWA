# OpenWA on the selected Mac mini

## Current state — 2026-10-11 America/Santiago

Source reviewed and installed in a temporary Linux validation environment at commit `8a45366a287750cf52c2658b0b783747cf51fad2` (OpenWA 0.24.0). `npm ci` and `npm run build:all` completed successfully. A local smoke check returned readiness, liveness and dashboard HTTP 200; unauthenticated session listing returned 401, authenticated listing returned 200 with an empty array. No WhatsApp account was paired.

These checks do not prove Docker/ARM64 startup on the Mac mini. That machine has not been accessed. Linking and continuous operation remain pending access to the target machine and phone-owner pairing.

## Installation on the target machine

Prerequisites: Git and an operational Docker engine with Docker Compose v2. Verify with `git --version`, `docker version` and `docker compose version`. Do not install another hosting service or expose the API publicly.

For a new checkout, run:

```sh
git clone https://github.com/vortvex/OpenWA.git
cd OpenWA
git checkout 8a45366a287750cf52c2658b0b783747cf51fad2
```

Copy `deploy/vortvex.env.example` from this setup branch into the checkout as `.env` only when no existing config is present; preserve an existing config and merge intentionally. Then:

```sh
chmod 600 .env
mkdir -p data
chmod 700 data
docker compose -f docker-compose.dev.yml config --quiet
docker compose -f docker-compose.dev.yml up -d --build
curl --fail http://127.0.0.1:2785/api/health/ready
```

The upstream single-container compose file is used intentionally: it mounts `./data` persistently and does not mount the Docker socket or launch the datastore orchestration proxy. Setting production and both database synchronization flags false uses database migrations. Baileys avoids Chromium and full-history download is disabled, although normal initial WhatsApp synchronization can still include recent chats and contacts. Phone notifications remain enabled. The bundled panel runs at `http://127.0.0.1:2785`. CSP upgrade is disabled only for this localhost HTTP use; use TLS before any nonlocal exposure.

The first boot creates an admin API key in `data/.api-key`. Keep it private; use it only for initial management. Do not paste the key, pairing code or WhatsApp session credentials into chat, logs, issues or Git. Runtime data and .env are ignored by the upstream repository.

## Pairing and agent access

Create one session in the dashboard and use the phone's Linked Devices flow to approve its QR. Confirm the dashboard reports connected before claiming successful pairing. Remote executors need authorized access to the Mac mini; localhost in a cloud workspace is not the Mac mini.

Create a separate VIEWER API key with the chosen session in `allowedSessions` and only explicitly selected conversations in `allowedChats`. An empty allowed-chats selection grants all chats: do not leave it empty for the agent. Do not use the admin key for routine agent work. MCP stays read-only; no sending, contacts writes, group changes or webhooks are configured in this setup.

The endpoint is `http://127.0.0.1:2785/mcp`. A running ChatGPT conversation cannot acquire a new MCP connector simply because the server is installed: client registration/reachability must be confirmed separately. Do not create a public tunnel to make it reachable.

## Operations

Keep the Mac mini and Docker running. Back up the private data directory to an approved private destination. Stop with `docker compose -f docker-compose.dev.yml stop`; this retains data. Disconnect through the phone's Linked Devices menu if access must be revoked. Do not delete data or re-pair as a troubleshooting shortcut.

No outgoing WhatsApp message is authorized by this installation request. Obtain an explicit recipient and purpose before sending.
