## Openclaw Installation on Linux (Ubuntu)

Here are the two primary ways to get OpenClaw running on Ubuntu.



### Option 1: Native CLI Installer

If you prefer running the framework directly on the host OS, the official bash script handles the heavy lifting, including fetching and installing Node.js (v22+) if your system doesn't have it yet.

1. **Run the One-Liner:**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash

```


2. **Launch the Background Daemon:**
To ensure the agent stays alive continuously in the background, run the onboarding command with the daemon flag:
```bash
openclaw onboard --install-daemon

```

---

### Option 2: Docker Compose (Recommended)

This approach provides the granular control and container isolation standard in robust infrastructure deployments. It also makes it straightforward to securely expose the web dashboard behind a Cloudflare tunnel or reverse proxy later on.

1. **Clone the Repository:**
```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw

```


2. **Execute the Setup Wrapper:**
This script builds the gateway image, handles the initial configuration, and spins up the containers.
```bash
./docker-setup.sh

```


3. **Configure the Agent:**
The interactive wizard will prompt you to select your LLM provider (like Anthropic, OpenAI, or Google Gemini) and paste your API keys.
4. **Access the Web Dashboard:**
The gateway UI will bind to port `18789`. If you are running this on a headless server, you can securely port-forward it via SSH to view it locally:
```bash
ssh -N -L 18789:127.0.0.1:18789 user@your_ubuntu_ip

```

Then visit `http://localhost:18789` in your local browser to input your gateway token.

---



Both paths will ultimately ask you to pair the agent with a chat interface so you can start issuing commands from your phone or desktop.

---

### Generating a Token and Hooking this up to a Telegram

Here are the steps to connect OpenClaw to Telegram. The strict HTTPS requirements Telegram enforces for webhooks are automatically handled by your Cloudflare DNS, making the routing to your Hostinger VPS incredibly smooth.

I've outlined the standard Telegram setup below, but if you prefer to keep your daily workflow inside Element, just let me know and we can pivot to the Matrix integration instead.

### Step 1: Generate the Bot Token

1. Open Telegram and search for the **@BotFather** account (ensure it has the official verified blue checkmark).
2. Send the message `/newbot` and follow the prompts. You will be asked to choose a display name and a unique username (the username must end in "bot").
3. BotFather will return an HTTP API Token (a long string of characters). Copy this to your clipboard.

### Step 2: Configure OpenClaw

1. SSH into your server.
2. If you are running the interactive Terminal UI (TUI) setup wizard, select **Telegram (Bot API)** as your messaging channel and paste the token when prompted.
3. If you are configuring it manually via your environment file, add the token like this:
```env
TELEGRAM_BOT_TOKEN="your_token_here"

```


Then, restart the OpenClaw gateway. *(Note: By default, OpenClaw uses long-polling for Telegram. This means your bot actively reaches out to Telegram, so you don't even need to open inbound ports or configure complex firewall rules for webhooks).*

### Step 3: Approve the Connection

1. Switch back to Telegram, search for your newly created bot's username, and send `/start` or a simple "hi".
2. The bot will respond with a secure **pairing code**.
3. Go back to your server's terminal and approve the device by running:
```bash
openclaw pairing approve telegram <code>

```


*(Replace `<code>` with the actual code your bot sent you).*

Once paired, you will receive a success message in both the terminal and Telegram, and you can start chatting with your AI assistant directly from your phone.

