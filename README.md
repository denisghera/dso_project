# Automated Vulnerability Discovery & Remediation

This repo contains a containerized WordPress lab for vulnerability discovery and remediation. It includes a GitHub Actions workflow that runs WPScan and stores scan output as artifacts.

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

Wait for the setup container to finish configuring the environment:

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

## GitHub Actions

Add a repository secret named `WPSCAN_API_TOKEN` before running the workflow.

Workflow path:

`.github/workflows/scan.yml`

## Notes

* Local secrets are stored in `.env` (ignored by git).
* Scan output artifacts are saved under `scans/` in CI.
