# Task 05 — Linux Server Administration & Nginx Reverse Proxy

A practical Ubuntu server-hardening and reverse-proxy configuration project.

## Implemented repository deliverables

- `nginx/nginx.conf` — reverse proxy, gzip compression, static cache, API no-cache headers, request rate limiting, connection limits, proxy headers, and security headers.
- `nginx/tls-server.conf.example` — HTTPS virtual-host reference with TLS 1.2/1.3 and HTTP-to-HTTPS redirect.
- `docs/linux-configuration.md` — non-root administration, SSH key authentication, safe SSH hardening, UFW firewall, Nginx setup, Certbot TLS, verification, and rollback instructions.

## Security controls

- SSH key authentication; disable password/root login only after confirming a second key-authenticated session.
- UFW defaults to deny inbound traffic; allow SSH from a trusted IP and HTTP/HTTPS.
- Keep the application upstream private; do not expose port 8000 directly.
- Rate-limit requests and connections at Nginx.
- Do not cache health checks or dynamic API responses.
- Never commit private keys, TLS private keys, passwords, or cloud credentials.

## Configure and test

See [Linux configuration runbook](docs/linux-configuration.md) and [Nginx configuration](nginx/nginx.conf).

On an Ubuntu server, validate with:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo ufw status verbose
curl -i http://YOUR_DOMAIN/health
```

For a real TLS deployment, point DNS to the server and use Certbot as documented in the runbook.

## Evidence and deployment status

The repository contains reusable configuration and a step-by-step runbook. Actual server provisioning, SSH/UFW execution, DNS, certificate issuance, and live HTTPS checks must be performed on a real Linux host and supported by genuine evidence before claiming a live deployment.

Evidence location: `docs/evidence/`.
