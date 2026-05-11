# test-subnet Validator Installer

This directory contains scripts to install and maintain a test-subnet validator node.

## Quick Installation

```bash
curl -s https://raw.githubusercontent.com/kacper-wolkiewicz-reef/test-subnet/refs/heads/deploy-config-prod/installer/install.sh | bash
```

This will:
1. Create a working directory at `~/test-subnet-validator/` (default).
2. Prompt you for configuration values if `.env` does not exist.
3. Fetch and run `update_compose.sh` once to set up `docker-compose.yml` and start the stack.
4. Install a cron job that re-runs `update_compose.sh` every 15 minutes so the validator stays up to date.

## Custom Installation

```bash
curl -s https://raw.githubusercontent.com/kacper-wolkiewicz-reef/test-subnet/refs/heads/deploy-config-prod/installer/install.sh | bash -s -- [ENV_NAME] [WORKING_DIRECTORY]
```

- `ENV_NAME`: branch suffix for `deploy-config-<ENV_NAME>` (defaults to `prod`).
- `WORKING_DIRECTORY`: where to install (defaults to `~/test-subnet-validator/`).

Example:

```bash
curl -s https://raw.githubusercontent.com/kacper-wolkiewicz-reef/test-subnet/refs/heads/deploy-config-prod/installer/install.sh | bash -s -- prod /opt/test-subnet-validator
```

## Prerequisites

- Docker and Docker Compose installed.
- `cron` running.
- `curl`, `bash`, and `openssl`.
- Validator wallet and hotkey on disk, funded and registered as needed for the
  target network and netuid (default location: `~/.bittensor/wallets`).
- Internet access to GitHub and the configured container registry.

## Container Registry Credentials

The `Build validator image` workflow pushes the validator image to
`ghcr.io/kacper-wolkiewicz-reef/`. It authenticates using:

- `username: ${{ github.actor }}`
- `password: ${{ secrets.GITHUB_TOKEN }}`

`GITHUB_TOKEN` is provided automatically by GitHub Actions; the workflow already
declares `packages: write`, so no extra secrets are required for the default
GitHub Container Registry setup.

## Environment Variables

Written to `<WORKING_DIRECTORY>/.env` on first run:

- `NETUID` — subnet netuid (default: `1`).
- `BITTENSOR_NETWORK` — Bittensor network alias (e.g. `finney`, `test`, `local`).
- `BITTENSOR_WALLET_NAME` / `BITTENSOR_WALLET_HOTKEY_NAME` — wallet identifiers consumed by pylon.
- `HOST_WALLET_DIR` — host-side path to the Bittensor wallets directory (mounted read-only into pylon).
- `ENVIRONMENT` — deploy environment suffix used in the validator image name (default: `prod`).
- `VALIDATOR_PYLON_OPEN_ACCESS_TOKEN` — auto-generated 32-byte hex token shared between validator and pylon.

## Manual Update

```bash
curl -s https://raw.githubusercontent.com/kacper-wolkiewicz-reef/test-subnet/refs/heads/deploy-config-prod/installer/update_compose.sh | bash -s -- [ENV_NAME] [WORKING_DIRECTORY]
```

## Image Rolling

The validator image name is environment-scoped:
`ghcr.io/kacper-wolkiewicz-reef/test-subnet-validator-${ENVIRONMENT}`.
The initial `docker-compose.yml` can use the `v0-latest` tag as a bootstrap
placeholder for the first operator rollout. After bootstrap, promote releases
only by pinning an immutable image digest
(`ghcr.io/kacper-wolkiewicz-reef/test-subnet-validator-${ENVIRONMENT}@sha256:...`)
in `deploy-config-prod`; the cron job on every operator host will pick it up
within 15 minutes.
