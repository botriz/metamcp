# MetaMCP deployment security preflight

This repository is an upstream-derived MCP gateway. Review upstream changes and this fork's differences before upgrading the production image.

## Required secrets

Create a private `.env` from `example.env`. Generate independent secrets:

```bash
openssl rand -hex 32
openssl rand -hex 32
```

Use the first value for `POSTGRES_PASSWORD` and the second for `BETTER_AUTH_SECRET`. Never reuse a password, commit `.env`, or paste production secrets into GitHub issues or pull requests.

Set `APP_URL` and `NEXT_PUBLIC_APP_URL` to the actual HTTPS hostname before starting. The Compose file intentionally refuses to start when required values are missing.

## Network boundary

- The application binds to loopback by default: `127.0.0.1:12008`.
- PostgreSQL is reachable only from the internal Compose network; no host port is published.
- Put a maintained TLS reverse proxy in front of the application.
- Do not expose the Docker socket or PostgreSQL port to the public internet.
- Use a firewall to allow only SSH and HTTPS from the intended sources.

## Startup

```bash
cp example.env .env
# Edit .env: set both generated secrets and the real HTTPS APP_URL.
docker compose config --quiet
docker compose up -d
docker compose ps
```

Keep the `postgres_data` volume and test backups/restores. Pin the container image to a reviewed immutable digest for production; the upstream `latest` tag is mutable.

This Compose file is a deployment template, not a completed production approval. Confirm authentication bootstrap, OAuth redirect URIs, MCP endpoint policies, provider credentials, and backup restoration before exposing the gateway.
