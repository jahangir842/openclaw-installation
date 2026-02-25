Openclaw Installation:

* github repo: https://github.com/openclaw/openclaw

### Step 1: Clone the Repository & Prepare the Environment

You need to build the images directly from the OpenClaw repository root to ensure all dependencies are cached correctly.

```bash
# Clone the repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Create your isolated host directories
mkdir -p ~/.openclaw/workspace
chmod 700 ~/.openclaw

```

Now, create your `.env` file. This directly addresses the Upwork requirement for "Proper API key management (environment variables, no hardcoding)."

```bash
touch .env
chmod 600 .env

```

Open the `.env` file and add a secure gateway token:

```env
OPENCLAW_GATEWAY_TOKEN=your_secure_random_token_here

```

### Step 2: Build the Core Images

Instead of pulling a pre-built image, building it locally allows you to control the layers and dependencies (like adding specific apt packages if a client requests them later).

Run the manual build command from the repo root:

```bash
docker build -t openclaw:local -f Dockerfile .

```

**The Security Bonus (Sandboxing):** The client emphasized security. OpenClaw has a powerful feature where the gateway runs on the host, but the agent's actual tool execution (running code, modifying files) happens in a *separate, throwaway Docker container*.

Build the dedicated sandbox image so we can enable this security feature later:

```bash
./scripts/sandbox-setup.sh

```

*Portfolio Note: Be sure to mention "Per-session Agent Sandboxing" in your Upwork interviews. It shows you understand how to prevent an AI agent from accidentally wiping the host system.*

### Step 3: Run the Onboarding Wizard via CLI

In a production environment, you never want your main gateway daemon blocked by an interactive terminal prompt. OpenClaw handles this by using a separate CLI execution for onboarding.

Run the wizard to set up your LLM providers (Anthropic, OpenAI, etc.):

```bash
docker compose run --rm openclaw-cli onboard

```

Follow the interactive prompts. The configurations you set here will be saved to your `~/.openclaw` volume.

### Step 4: Start the Gateway Service

Once onboarding is complete, you can launch the continuous gateway daemon in detached mode.

```bash
docker compose up -d openclaw-gateway

```

You can verify it is running healthily by checking the logs:

```bash
docker compose logs -f openclaw-gateway

```

### Step 5: Access the Control Dashboard

To securely access the web interface, you need to generate a paired session link. Run the dashboard command via the CLI tool:

```bash
docker compose run --rm openclaw-cli dashboard --no-open

```

This will output a URL like `http://127.0.0.1:18789/?token=...`.

1. Open this link in your browser.
2. If you see an "unauthorized" or "pairing required" message, it means your browser device needs approval.
3. List pending devices: `docker compose run --rm openclaw-cli devices list`
4. Approve your browser: `docker compose run --rm openclaw-cli devices approve <requestId>`

### Step 6: Enable the Advanced Sandbox (Optional but Recommended)

To truly harden the setup, edit your OpenClaw configuration file (located in `~/.openclaw/config/` after onboarding) and enable the sandbox feature we built in Step 2:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "agent",
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "network": "none"
        }
      }
    }
  }
}

```

*This configuration ensures that any code the agent executes runs in an isolated `bookworm-slim` container with absolutely no network egress (`network: "none"`).*

---

### How to pitch this on Upwork

Once you have this running locally, you have everything you need to win the job. You can tell the client:

> *"I have set up the manual Docker Compose flow for OpenClaw. I separated the `openclaw-gateway` daemon from the interactive `openclaw-cli` to keep the runtime clean. I utilized strict `.env` injection for API keys to prevent hardcoding. Furthermore, I built the dedicated `openclaw-sandbox:bookworm-slim` image and configured the agent to execute all tools inside a secondary, throwaway container with `network: none` enabled, ensuring complete host isolation."*

Would you like me to write a quick `README.md` based on this flow that you can send to the Upwork client as an example of the "Basic documentation" they requested?