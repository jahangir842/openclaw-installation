### Phase 1: Secure Server Configuration (VPS Hardening)

Before deploying any containers, the underlying Ubuntu host must be secured against automated attacks.

**1. Enforce SSH Key Authentication**
Disable password logins to prevent brute-force attacks.

```bash
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
sudo systemctl restart ssh

```

**2. Configure the Firewall (UFW)**
Drop all incoming traffic except for essential ports.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp   # Required for Let's Encrypt HTTP-01 challenge
sudo ufw allow 443/tcp  # Required for secure HTTPS UI access
sudo ufw enable

```

---

### Phase 2: Repository & Environment Setup

Clone the official repository to build the image locally and configure strict file permissions for sensitive data.

**1. Clone the Source**

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw

```

**2. Prepare Isolated Volumes**
Create the directories that will be mounted into the containers, ensuring the host user owns them securely.

```bash
mkdir -p ~/.openclaw/workspace
chmod 700 ~/.openclaw

```

**3. API Key Management (.env)**
Create the environment file. Setting `chmod 600` ensures only the owner can read these credentials.

```bash
touch .env
chmod 600 .env

```

Populate `.env` with your secure tokens:

```env
# .env
OPENCLAW_GATEWAY_TOKEN=generate_a_long_secure_random_string
OPENAI_API_KEY=sk-proj-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here

```

---

### Phase 3: Build the Local Image

Building the image directly from the repository gives you control over the dependencies and caches the layers for faster future updates.

Run this from the root of the cloned `openclaw` directory:

```bash
docker build -t openclaw:local -f Dockerfile .

```

---

### Phase 4: Reverse Proxy Configuration

We will use Nginx to terminate SSL and route traffic securely to the internal Docker network.

Create an `nginx.conf` file in your deployment directory:

```bash
nano nginx.conf

```

Add the following configuration, replacing `yourdomain.com` with the actual domain pointing to the VPS:

```nginx
server {
    listen 80;
    server_name openclaw.yourdomain.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name openclaw.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/openclaw.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://openclaw-gateway:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

```

---

### Phase 5: Docker Compose Architecture

This stack isolates the gateway entirely. It does not map port `18789` to the host machine; instead, Nginx acts as the only entry point.

Create your `docker-compose.yml`:

```yaml
version: '3.8'

services:
  openclaw-gateway:
    image: openclaw:local
    container_name: openclaw-gateway
    env_file: .env
    restart: unless-stopped
    volumes:
      - ~/.openclaw:/home/node/.openclaw
      - ~/.openclaw/workspace:/home/node/.openclaw/workspace
    # Notice: No external ports mapped here.
    command: ["node", "dist/index.js", "gateway", "--bind", "lan"]

  openclaw-cli:
    image: openclaw:local
    container_name: openclaw-cli
    env_file: .env
    volumes:
      - ~/.openclaw:/home/node/.openclaw
      - ~/.openclaw/workspace:/home/node/.openclaw/workspace
    stdin_open: true
    tty: true
    entrypoint: ["node", "dist/index.js"]

  nginx:
    image: nginx:alpine
    container_name: openclaw-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
    depends_on:
      - openclaw-gateway

```

---

### Phase 6: Bootstrapping & Operations

With the architecture defined, we initialize the configuration.

**1. Run the Onboarding Wizard**
Use the CLI container to configure your models without interrupting the main gateway daemon.

```bash
docker compose run --rm openclaw-cli onboard

```

**2. Secure the UI Origin**
Because OpenClaw is strict about web security, you must explicitly tell it your domain name is safe. Open `~/.openclaw/openclaw.json` and add your domain to the `controlUi` block:

```json
  "gateway": {
    "bind": "lan",
    "controlUi": {
      "allowedOrigins": ["https://openclaw.yourdomain.com"]
    }
  }

```

For unsafe and local access, you can add these:

```json
{
  "gateway": {
    "controlUi": {
      "dangerouslyAllowHostHeaderOriginFallback": true
    }
  },
  "agents": { ... },
  ... (the rest of your existing settings) ...
}
```

**3. Generate SSL Certificates**
Temporarily use Certbot to fetch the Let's Encrypt certificates before spinning up the full stack:

```bash
docker run -it --rm --name certbot \
  -v "$(pwd)/certbot/conf:/etc/letsencrypt" \
  -v "$(pwd)/certbot/www:/var/www/certbot" \
  certbot/certbot certonly --webroot -w /var/www/certbot -d openclaw.yourdomain.com

```

**4. Start the Production Stack**

```bash
docker compose up -d

```

**5. Device Pairing**
To log into the UI at `https://openclaw.yourdomain.com`, you will need a secure link and device approval:

```bash
# Get the login link
docker compose run --rm openclaw-cli dashboard --no-open

# Approve your browser session
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <Request_ID>

```

---

