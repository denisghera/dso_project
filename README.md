# Automated Vulnerability Discovery & Remediation Pipeline

This repository contains a complete DevSecOps pipeline for a containerized WordPress lab. It automates vulnerability discovery, applies basic security hardening, publishes a patched image, and verifies the remediation via WPScan.

## Architecture & Remediation

* **Vulnerable Baseline:** The initial environment utilized WordPress 6.0.3 and outdated plugins to simulate a legacy application.
* **Hardened Image:** The pipeline builds a custom `Dockerfile.hardened` which drops root privileges, removes unnecessary OS packages (e.g., `curl`, `wget`), and uses the latest WordPress core.
* **Automated Hardening:** The `wp-cli` container dynamically updates plugins, disables XML-RPC, and removes default informational files.
* **Registries:** The hardened image is automatically pushed to Docker Hub and GitHub Container Registry (GHCR).

## Quick start (local)

1) Create your local env file:

```bash
cp .env.example .env
```

2) Start the stack:

```bash
docker compose --env-file .env -f docker/docker-compose.yml down -v
docker compose --env-file .env -f docker/docker-compose.yml up -d --build
```

3) Check that WordPress and plugins are installed:

Wait for the setup container to finish configuring and hardening the environment:

```bash
docker logs -f wp-cli-setup
```

To list the installed plugins after setup is complete:

```bash
docker compose --env-file .env -f docker/docker-compose.yml run --rm wp-cli wp plugin list --allow-root
```

## Manual WPScan (local)

```bash
docker run -it --rm --network host wpscanteam/wpscan \
    --url http://localhost:8080 \
    --enumerate u,vp,vt,tt \
    --plugins-detection aggressive \
    --no-update --force
```

With API token:

```bash
docker run -it --rm --network host wpscanteam/wpscan \
    --url http://localhost:8080 \
    --enumerate u,vp,vt,tt \
    --plugins-detection aggressive \
    --api-token YOUR_TOKEN \
    --no-update --force
```

## GitHub Actions CI/CD

The automated workflow (`.github/workflows/scan.yml`) builds the hardened image, pushes it to the registries, deploys the stack, and runs an aggressive WPScan.

**Required Repository Secrets:**
Before running the workflow, you must configure the following secrets in your GitHub repository settings:

* `WPSCAN_API_TOKEN` - Token from wpscan.com
* `DOCKERHUB_USERNAME` - Your Docker Hub username
* `DOCKERHUB_TOKEN` - Your Docker Hub access token

## Notes

* Local secrets are stored in `.env` (ignored by git).
* Scan output artifacts (JSON format) are saved under `scans/` in CI.
* The workflow uses the default `GITHUB_TOKEN` to push to GHCR; ensure the workflow has "Read and write permissions" enabled under **Settings > Actions > General > Workflow permissions**.