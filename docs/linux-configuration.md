# Linux Server Administration & Nginx Reverse Proxy

This runbook targets Ubuntu 22.04/24.04. It is a reproducible configuration guide;
it is not evidence that a remote server has already been provisioned.

## 1. Create a non-root deployment account

```bash
sudo adduser deploy
sudo usermod -aG sudo deploy
```

Use a strong password for initial account setup, then administer with SSH keys.
Do not run application processes as root.

## 2. Configure SSH key authentication safely

On the client, create an Ed25519 key if one does not already exist:

```bash
ssh-keygen -t ed25519 -a 64
ssh-copy-id deploy@SERVER_IP
```

Open a second terminal and confirm key login works before changing SSH settings.
Keep the current SSH session open as a recovery path.

In `/etc/ssh/sshd_config.d/ hardening.conf` (create the file at the exact path
`/etc/ssh/sshd_config.d/50-hardening.conf`), set:

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
KbdInteractiveAuthentication no
```

Validate before reload:

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Do not close the existing session until a new key-authenticated session succeeds.
Cloud provider security-group rules must also allow SSH from your trusted IP only.

## 3. Configure UFW

First ensure SSH is allowed so you do not lock yourself out:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from YOUR_PUBLIC_IP to any port 22 proto tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

Replace `YOUR_PUBLIC_IP`. If the address changes, update the SSH rule before
removing the old one. Never expose the application port (8000) publicly.

## 4. Install and configure Nginx

```bash
sudo apt update
sudo apt install -y nginx
sudo mkdir -p /var/cache/nginx/task05
```

Copy `nginx/nginx.conf` into `/etc/nginx/conf.d/task05.conf`. This file is
included from Nginx's `http` context on standard Ubuntu packages. Ensure the
backend listens only on `127.0.0.1:8000` or an equivalent private interface.

```bash
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
sudo systemctl status nginx --no-pager
curl -i http://127.0.0.1/health
```

The configuration enables gzip, static-resource caching, per-client request
rate/connection limits, proxy headers, and non-cacheable health/API responses.
Adjust `server_name` and the upstream to match the deployment.

## 5. Enable HTTPS

Point a domain's DNS A/AAAA record to the server, allow TCP 80/443 in both UFW
and the cloud firewall, then install Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d YOUR_DOMAIN
sudo certbot renew --dry-run
```

Replace `YOUR_DOMAIN`. Certbot updates the Nginx virtual host with certificate
paths and HTTPS settings. Do not claim TLS is active until the domain resolves,
the certificate is issued, and an HTTPS request succeeds.

## 6. Verification and evidence

Capture real command output (redact public IPs and hostnames if needed):

```bash
lsb_release -a
sudo sshd -T | grep -E 'pubkeyauthentication|passwordauthentication|permitrootlogin'
sudo ufw status numbered
sudo nginx -t
sudo systemctl is-active nginx
curl -I http://YOUR_DOMAIN/health
curl -I https://YOUR_DOMAIN/health
sudo certbot certificates
```

Store genuine screenshots/logs in `docs/evidence/`. Do not paste fabricated
success output. Record the date, OS version, server provider, and test result.

## Rollback and safety

- Keep provider console access available before changing SSH or firewall rules.
- Validate SSH configuration with `sshd -t` before reload.
- Validate Nginx with `nginx -t` before reload.
- Keep a copy of the previous Nginx configuration before replacing it.
- Never commit private keys, TLS private keys, passwords, or cloud credentials.
