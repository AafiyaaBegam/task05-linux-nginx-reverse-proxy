# Linux Server Configuration

## 1. User Permissions
Create a dedicated non-root user for server administration.

```bash
sudo adduser deploy
sudo usermod -aG sudo deploy
```

## 2. SSH Key Authentication

Generate an SSH key on the client machine and copy the public key to the server.

```bash
ssh-keygen -t ed25519
ssh-copy-id deploy@SERVER_IP
```

After verifying key-based login, disable password authentication in the SSH server configuration:

```text
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH after validating the configuration.

## 3. UFW Firewall

Allow only the required service ports:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

## 4. Nginx Reverse Proxy

Nginx listens on HTTP port 80 and forwards requests to the backend application at `127.0.0.1:8000`.

Configuration: `nginx/nginx.conf`

## 5. SSL/TLS

For a public domain, Certbot can obtain and configure a Let's Encrypt certificate:

```bash
sudo certbot --nginx -d example.com
```

Replace `example.com` with your own domain.

## 6. Verification

```bash
sudo nginx -t
sudo systemctl status nginx
sudo ufw status
```

**Note:** These are configuration instructions, not proof of a deployed Linux server. Execute and verify them in a Linux environment before claiming production deployment.
