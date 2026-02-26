# Openclaw Installation on Linux (Ubuntu)

Here are the two primary ways to get OpenClaw running on Ubuntu.

### Installation: Native CLI Installer

1. **Remove the Old Versions**

Remove the openclas if it has been installed previously:

  ```bash
  openclaw uninstall --all --yes --non-interactive
  npm uninstall -g openclaw
  npm cache clean --force
  ```

2. **Install Openclaw One-Liner:**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

3. **Configure the Agent & LLM Provider:**
The interactive wizard will prompt you to select your LLM provider. Here is how to handle the configuration depending on your backend:

* **Ollama**:
```bash
Model/auth provider Skip for now
Filter models by provider All providers
Default model Enter model manually
Default model ollama/llama3.2
```

  
* **Model/auth provider**
Skip this if you're running the model locally, or use OpenAI or Anthropic if you are using the API.
```bash
Skip for now
```

* **Filter models by provider**
Select `All providers` if you're running the model locally, or use OpenAI or Anthropic's specific model names if you are using the API.
```bash
All providers
```

Create a backup of the existing configuration:

```bash
mv .openclaw/openclaw.json .openclaw/openclaw.json.bak
```

Then create a new **sample Ollama configuration file**:

Use one of the example configurations below as a reference:

```bash
nano ./.openclaw/openclaw.json
```

Use the following code:

```bash
{
  "meta": {
    "lastTouchedVersion": "2026.2.23",
    "lastTouchedAt": "2026-02-24T10:37:28.411Z"
  },
  "wizard": {
    "lastRunAt": "2026-02-24T10:37:28.394Z",
    "lastRunVersion": "2026.2.23",
    "lastRunCommand": "configure",
    "lastRunMode": "local"
  },
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://localhost:11434",
        "apiKey": "ollama-local",
        "api": "ollama",
        "models": [
          {
            "id": "llama3.2",
            "name": "llama3.2",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 32768,
            "maxTokens": 32768
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/llama3.2"
      },
      "models": {
        "ollama/llama3.2": {}
      },
      "workspace": "/home/user/.openclaw/workspace",
      "compaction": {
        "mode": "safeguard"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw"
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "password",
      "password": "pakistan"
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "nodes": {
      "denyCommands": [
        "camera.snap",
        "camera.clip",
        "screen.record",
        "calendar.add",
        "contacts.add",
        "reminders.add"
      ]
    }
  }
}

```

Restart the Openclaw Daemon

```bash
openclaw gateway restart
```

* **Local Models (vLLM):** Select the vLLM option. Input your vLLM base URL (e.g., `http://192.168.3.74:8080/v1`). When prompted for the vLLM API key, simply enter a placeholder like `sk-local`, `dummy`, or `1234`. OpenClaw requires a non-empty string to format the authorization header correctly, but your local vLLM instance will safely ignore it.

* **Cloud APIs (Claude / OpenAI):** If you switch to an external provider, select Anthropic or OpenAI and paste your active API key when prompted (e.g., `sk-ant-api03-...` for Claude or `sk-proj-...` for OpenAI).

* **Configure skills now**
```bash
Skip for now
```
* **Enable hooks**
```bash
Skip for now
```

Start TUI (Text User Interface)
```bash
Skip for now
```

The Installation has been completed.

If you want to reconfigure openclaw, you can run the following command:

```bash
openclaw configure
```

---

### Access the Web Dashboard:**

Configure Autherization:

The gateway requires auth. Add a token or password, 
openclaw dashboard --no-open 

nano ~/.openclaw/openclaw.json

and set the follwoing gateway auth section:

```bash
    "auth": {
      "mode": "password",
      "password": "pakistan"
```

The gateway UI will bind to port `18789`. If you are running this on a headless server, you can securely port-forward it via SSH to view it locally:
```bash
ssh -N -L 18789:127.0.0.1:18789 user@your_ubuntu_ip
```

Then visit the following link in your local browser to input your gateway token.

```bash
http://localhost:18789
```
Go to overview page, enter the password and click `Connect`.

---

### Generating a Token and Hooking this up to Telegram

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

Once paired, you will receive a success message in both the terminal and Telegram.

---

### Step 4: Configure Agent Skills

An AI agent without skills is just a standard chatbot. To allow your OpenClaw agent to execute real tasks, you need to install specific modules.

1. **Access the Skill Registry:** From your terminal, run `openclaw skills list` to view the catalog of available capabilities.
2. **Install Core Capabilities:** Use the CLI to equip the agent with the tools it needs. For standard operations, you will likely want:
```bash
openclaw skills install web-browser  # Enables headless web scraping and page interaction
openclaw skills install terminal     # Grants secure shell command execution in the sandbox
openclaw skills install file-system  # Permits reading and writing local files

```


3. **Verify:** Open your newly connected Telegram chat and ask the bot, "What skills do you currently have enabled?" to confirm the modules are active and recognized by the LLM.

---

## Enable Hooks 

In OpenClaw, **Hooks** are an extensible, event-driven system made up of small scripts that automatically run in response to specific agent commands or lifecycle events.

When the setup wizard asks if you want to **Enable hooks?**, it is asking whether you want to activate this background automation layer.

### What Hooks Do

If enabled, OpenClaw can automatically trigger specific actions when certain events occur inside the gateway (such as issuing a `/new`, `/reset`, or `/stop` command). Some common uses for bundled hooks include:

* **Session Memory:** Automatically saving a snapshot of your conversation context to the agent's memory file whenever you reset the chat with the `/new` command.
* **Command Logging:** Keeping a centralized audit trail of all the commands you issue to the agent, which is highly useful for troubleshooting.
* **Custom Automations:** Triggering follow-up actions, injecting extra workspace files, or calling external webhooks when the agent starts or ends a session.

### What Should You Choose?

If you are just getting started and want to keep the initial setup simple, you can safely select **"Skip for now"**. You can always enable and configure hooks later via the CLI (using commands like `openclaw hooks list` or `openclaw hooks enable`) once you are more familiar with the framework.

---

## Verifying Your OpenClaw Installation

Once the installation is complete, you need to reload your shell configuration so your terminal recognizes the newly installed command.

1. **Reload your bash profile:**
```bash
source ~/.bashrc

```


2. **Test the OpenClaw CLI:**
Run the base command to ensure the framework is active and responding.
```bash
openclaw

```



**Expected Output:**

> 🦞 OpenClaw 2026.2.22-2 (45febec) — I'll do the boring stuff while you dramatically stare at the logs like it's cinema.

---

## Installing External Skills via ClawHub

While core capabilities are built directly into OpenClaw, third-party integrations must be downloaded using **ClawHub**, the framework's official package manager.

1. **Install ClawHub Globally:**
Since OpenClaw runs on Node.js, use `npm` to install the ClawHub CLI so it is accessible anywhere on your system.
```bash
npm install -g clawhub

```


2. **Download the GitHub Skill:**
Use the newly installed ClawHub tool to fetch the GitHub integration into your workspace.
```bash
clawhub install github

```


**Expected Output:**
> `✔ OK. Installed github -> /home/open/.openclaw/workspace/skills/github`



---

## Authenticating the GitHub Skill

By default, the GitHub skill handles repositories, pull requests, and commits using either a managed OAuth flow or a Personal Access Token (PAT). For a headless daemon running on an Ubuntu server, passing a PAT via your environment variables is the most reliable method.

**Step 1: Generate a Personal Access Token**

1. Log in to GitHub in your browser and go to **Settings > Developer Settings > Personal Access Tokens (Classic)**.
2. Click **Generate new token**.
3. Give it a descriptive name (e.g., "OpenClaw Daemon") and check the `repo` scope (to read/write code and issues) and the `project` scope.
4. Generate the token and copy it to your clipboard.

**Step 2: Add the Token to OpenClaw**
OpenClaw reads environment variables from a hidden `.env` file in its home directory.

1. Open or create this file on your server:
```bash
nano ~/.openclaw/.env

```


2. Add your GitHub token to the file:
```env
GITHUB_TOKEN="ghp_your_long_token_string_here"

```


3. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

**Step 3: Restart the Daemon**
For the background service to pick up the new environment variable, restart the OpenClaw process:

```bash
openclaw gateway restart

```

With authentication complete, the agent can streamline how you track tasks. Instead of manually writing them out, you can just ask OpenClaw to review your recent Nuxt.js logs for the `gts-frontend` and automatically draft a detailed GitHub issue containing the stack traces and proposed fixes.

---

## SOUL.md

The `SOUL.md` file is the brain of your workspace. While OpenClaw has a base personality, dropping a `SOUL.md` file into a project directory overrides its default behavior and gives the agent strict, context-aware instructions on how to behave, code, and manage infrastructure specifically for that repository.

Here is how to set it up and tailor it to your engineering workflow.

### Step 1: Create the File

Navigate to the root directory of your project (for example, your `gts-frontend` folder or your AI backend) and create the file:

```bash
cd /path/to/your/project
nano SOUL.md

```

### Step 2: Define the Rules

The file is written in standard Markdown. You want to define the agent's identity, its strict technical stack, and how it should handle your task management.

Paste the following template into the file and adjust it as needed:

```markdown
# Identity
You are an expert DevOps engineer, System Administrator, and full-stack developer assisting with project architecture, deployment, and debugging.

# Coding & Infrastructure Standards
- **Backend & APIs:** Default to Python and FastAPI. Ensure all endpoints are asynchronous where appropriate.
- **Machine Learning:** Strictly use PyTorch over TensorFlow for all model architecture and training scripts.
- **Frontend:** Follow Nuxt.js conventions and best practices for component structure.
- **Infrastructure:** All services must be containerized using Docker. Write configurations with Kubernetes deployments in mind.

# Workflow & Task Management
- **Issue Tracking:** Strictly use GitHub issues to track tasks, bugs, and infrastructure changes. 
- **Bug Reporting:** When asked to review logs, automatically draft a GitHub issue containing the formatted stack trace, root cause analysis, and a proposed fix.
- **Communication:** Be direct, concise, and prioritize technical accuracy over conversational filler.

```

### Step 3: Save and Apply

Save the file (`Ctrl+O`, `Enter`, `Ctrl+X`).

Whenever OpenClaw operates within this directory, it will read this `SOUL.md` file first. It now inherently knows not to suggest TensorFlow when you ask for a neural network snippet, and it knows exactly how you prefer to track your bugs.

### Testing It Out

To see this in action, you can test it directly from your CLI or your connected chat. Navigate to the directory containing the `SOUL.md` and run a prompt through the OpenClaw message command:

```bash
openclaw message send "Review the recent Nuxt.js build logs in this directory. If there are any Docker image build errors, create a new GitHub issue detailing the problem."

```

---

### How OpenClaw Handles Memory

OpenClaw separates its memory system into two distinct layers: the active context window (short-term) and the persistent local database (long-term). This ensures the agent does not lose track of complex projects over time, while keeping your local vLLM from being overwhelmed by massive token counts.

**1. Short-Term Memory (The Active Session)**

* **How it works:** This is your immediate back-and-forth conversation. It holds the recent shell commands, the stack traces you just pasted, and the current logic of the file you are editing.
* **Best Practice:** When you finish troubleshooting one bug and move to a completely different task, type `/new` or `/reset` in your chat interface. This clears the short-term context. It prevents token bloat and keeps the model from confusing old error logs with new ones.

**2. Long-Term Memory (Persistent Context)**

* **How it works:** When you reset a session, OpenClaw automatically distills the important takeaways from that conversation (such as architectural decisions, successfully resolved bugs, or specific server configurations) and writes them to its local storage.
* **Manual Storage:** You can explicitly force the agent to store a critical fact by telling it directly: *"Remember that the production database requires a VPN connection."*
* **Retrieval:** In future sessions, the agent runs a background search against this database to pull relevant context before replying. You can also query it yourself by asking, *"What do you remember about the database configuration?"*

### Managing the Memory Files

Because OpenClaw stores all memory locally on your Ubuntu server, your data remains entirely private, and you maintain complete control over what the agent knows.

* **Viewing the Data:** You can visualize the agent's memory map and read the exact nodes of information it has saved by opening the Web UI dashboard.
* **Pruning Context:** If the agent starts stubbornly referencing outdated infrastructure or old code, you can delete those specific memory nodes via the dashboard, or simply instruct the agent in chat: *"Forget the previous instructions about..."*

---

Great catch! You've found the exact panel needed to bridge the gap between your personal PC and the OpenClaw gateway.

Based on your latest screenshot, here is your finalized, step-by-step guide for establishing a secure, authenticated connection to your DevOps dashboard.

---

## Guide: Accessing OpenClaw Dashboard on a Local Network

To resolve the **"Secure Context"** and **"Unauthorized"** errors seen in your terminal and browser, follow these steps.

### 1. Establish a Secure Context (SSH Tunnel)

Browsers block gateway features on plain IP addresses (like `192.168.3.77`). You must use an SSH tunnel to make the browser believe the dashboard is local.

* **Run this on your personal PC:**
```bash
ssh -N -L 18789:127.0.0.1:18789 open@192.168.3.77

```


* **Open your browser to:** `http://localhost:18789`
* *This resolves the red "control ui requires device identity" error.*

### 2. Authenticate the Dashboard (The "Overview" Tab)

Once the page loads, you will see an "Unauthorized" error. You must provide the password found in your `openclaw.json` file.

* **Navigate to:** `Control` > `Overview` in the left sidebar.
* **Locate "Gateway Access":** Find the **Password (not stored)** field.
* **Enter Password:** Type `pakistan`.
* **Action:** Click **Connect**.
* *The status will turn green, and your "Config" and "Debug" snapshots will automatically populate with your server data.*

### 3. Verify Connection

* Check the top right of the dashboard; it should no longer say "Health Offline."
* Go to the **Config** tab; the "Raw JSON5" block should now display your full `openclaw.json` contents.

---

## 3. Remote Login

1. **Get Token:**
   ```bash
   source ~/.bashrc
   openclaw gateway token`
   ```
4. **Access:** On your personal PC, go to `http://<ubuntu-ip>:18789`

---

## Cloudflare Access:

Exposing your dashboard directly to the internet by opening ports on your Hostinger VPS is a major security risk, especially for an AI agent with access to your file system and terminal.

A **Cloudflare Tunnel** solves this by creating a secure, outbound-only connection from your server to Cloudflare's edge network. It gives you a beautiful, secure HTTPS URL (like `openclaw.yourdomain.com`) without opening any inbound firewall ports.

Here is how to set it up on your Ubuntu server.

### Step 1: Install the Cloudflare Daemon (`cloudflared`)

First, download and install the official Cloudflare package.

```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

```

### Step 2: Authenticate with Your Cloudflare Account

Run the login command. It will generate a URL in your terminal.

```bash
cloudflared tunnel login

```

Copy that URL, paste it into your local web browser, log in to your Cloudflare account, and select the domain you want to use for this tunnel.

### Step 3: Create the Tunnel

Once authenticated, go back to your server terminal and create a new tunnel named `openclaw-ui`:

```bash
cloudflared tunnel create openclaw-ui

```

*Note: This command will output a UUID (a long string of letters and numbers) and the path to a `.json` credentials file. You will need that UUID for the next step.*

### Step 4: Configure the Routing

You need to tell Cloudflare to route traffic from your new tunnel to OpenClaw's local port (`18789`).

1. Create and open a configuration file:
```bash
nano ~/.cloudflared/config.yml

```


2. Paste the following configuration, replacing `<your-tunnel-uuid>` with the ID generated in Step 3, and changing the hostname to your desired web address:
```yaml
tunnel: <your-tunnel-uuid>
credentials-file: /home/open/.cloudflared/<your-tunnel-uuid>.json

ingress:
  - hostname: openclaw.yourdomain.com
    service: http://127.0.0.1:18789
  - service: http_status:404

```


3. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

### Step 5: Route the DNS and Start the Service

Now, map the tunnel to your actual DNS records and install it as a background service so it survives server reboots.

1. Create the DNS record automatically:
```bash
cloudflared tunnel route dns openclaw-ui openclaw.yourdomain.com

```


2. Install and start the service:
```bash
sudo cloudflared service install
sudo systemctl start cloudflared
sudo systemctl enable cloudflared

```



That's it! You can now open your browser on any device and navigate to `https://openclaw.yourdomain.com`. You will be greeted by the OpenClaw login screen where you can paste your Gateway token, and your traffic will be fully encrypted end-to-end.

---
