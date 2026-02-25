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
Instead of creating a blank file, copy the official template. Set `chmod 600` immediately to ensure only the root/deployment user can read the credentials.

```bash
cp .env.example .env
chmod 600 .env
sudo chown -R 1000:1000 ~/.openclaw

```



###  Create a secure token to a secure random string
```bash
openssl rand -hex 32
```

Open the `.env` file (`nano .env`). Uncomment and fill in your secure tokens, then append the infrastructure paths required by Docker Compose to the bottom of the file (replace `/home/youruser` with your actual absolute path, e.g., `/root`):

```bash
nano .env
```

Add the following settings:

```env
# --- Inside .env ---



OPENCLAW_GATEWAY_TOKEN=generate_a_long_secure_random_string

# 2. Uncomment and set your model provider keys
OPENAI_API_KEY=sk-proj-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here

# 3. Append these Docker Infrastructure Paths at the bottom of the file
OPENCLAW_IMAGE=openclaw:local
OPENCLAW_CONFIG_DIR=/home/youruser/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/youruser/.openclaw/workspace
OPENCLAW_GATEWAY_BIND=lan

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

This stack utilizes the official repository's environment variables but isolates the gateway entirely. Nginx acts as the only entry point. It also includes `init: true` to ensure the Node.js processes handle termination signals correctly.


#### Backup the existing `docker-compose.yml`

```bash
mv docker-compose.yml docker-compose.yml.bkp
```

#### Create a new `docker-compose.yml` file

```bash
nano docker-compose.yml
```

Add the following content to the file:


```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    container_name: openclaw-gateway
    env_file: .env
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    # Notice: No external ports mapped here. Traffic goes through Nginx.
    init: true
    restart: unless-stopped
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND:-lan}",
        "--port",
        "18789",
      ]

  openclaw-cli:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    container_name: openclaw-cli
    env_file: .env
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
      BROWSER: echo
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    stdin_open: true
    tty: true
    init: true
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

After the installation begins, follow this setup guide for the remaining steps:

[https://github.com/jahangir842/openclaw-installation/blob/main/installation-native-CLI.md](https://github.com/jahangir842/openclaw-installation/blob/main/installation-native-CLI.md)


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

*For unsafe/local testing without a domain, you can fallback to: `"dangerouslyAllowHostHeaderOriginFallback": true*`

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
