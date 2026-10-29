# StrixGuard Agent

The **StrixGuard Agent** (`strixguard-agent`) is a small, outbound-only program
that connects [StrixGuard](https://strixguard.org) to a self-managed GitLab
inside your network, without opening any inbound port.

- **Broker mode** (default): the agent relays read-only GitLab requests for
  StrixGuard cloud, which clones and scans. Only the agent is installed.
- **Local scan**: the agent clones and scans inside your network and uploads
  findings; normal scans may also upload allowlisted raw scanner artifacts,
  logs and source-derived excerpts.
- In the default agent-local custody mode, the GitLab token stays on the agent
  host. Sealed custody sends the plaintext token to the backend once for
  sealing. GitLab hosts and namespaces are pinned in the agent's own
  configuration; StrixGuard can narrow, never widen.
- AI labeled **local** uses an operator-configured endpoint; it is not
  guaranteed to run on the agent host unless explicitly constrained.

This repository publishes the agent **releases** (Linux `amd64` and `arm64`).
The source is maintained by StrixGuard; see [Security](SECURITY.md) for
reporting issues.

## Download

From the [latest release](https://github.com/StrixGuard/strixguard-agent/releases/latest):

```bash
ARCH=amd64   # or arm64
curl -fLO https://github.com/StrixGuard/strixguard-agent/releases/latest/download/strixguard-agent_linux_${ARCH}.tar.gz
curl -fLO https://github.com/StrixGuard/strixguard-agent/releases/latest/download/SHA256SUMS
sha256sum --ignore-missing -c SHA256SUMS
tar -xzf strixguard-agent_linux_${ARCH}.tar.gz
sudo ./strixguard-agent/install.sh
```

`latest` points to the newest production release. For a pre-release, use its
tag instead, for example
`https://github.com/StrixGuard/strixguard-agent/releases/download/pre-sha-41abb15e355d/strixguard-agent_linux_amd64.tar.gz`
(see [Releases](https://github.com/StrixGuard/strixguard-agent/releases)).

Your StrixGuard installation serves the same files, matching its version:
`https://<your StrixGuard API host>/v1/agent/releases/latest`
(also shown under **Settings → Agents** in the app).

## Install and register

1. In StrixGuard open **Settings → Agents → Create agent** and copy the one-time
   registration token (valid for one hour).
2. Install as above. `install.sh` creates the `strixguard-agent` system user,
   `/etc/strixguard-agent` and `/var/lib/strixguard-agent`, installs the binary
   and a hardened systemd unit, and never handles secrets.
3. Edit `/etc/strixguard-agent/agent.yaml`: `server.url` is your StrixGuard API
   address; add your GitLab under `scm` (and `ca_file` for a private CA).
4. Store the GitLab token (`read_api` + `read_repository` only) and a key
   passphrase as root-only credential files, register with the token, run
   `strixguard-agent doctor`, then `sudo systemctl enable --now strixguard-agent`.

Full guide: https://docs.strixguard.org/agent/install ·
Configuration: https://docs.strixguard.org/agent/configuration

## Verify a release

Every release has a `SHA256SUMS` file. Check the archive before installing:

```bash
sha256sum --ignore-missing -c SHA256SUMS
```

Each archive contains `strixguard-agent/` with the static binary, `install.sh`,
`strixguard-agent.service`, `agent.yaml.example`, `README.txt` and `VERSION`.

## Versions

Releases follow the StrixGuard server version they ship with. Pre-releases
(`pre-…`) match the StrixGuard pre-production environment; use the latest
non-pre-release in production.
