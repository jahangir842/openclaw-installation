# OpenClaw Installation Reference

This repository provides comprehensive installation guides and configurations for deploying **OpenClaw**, a powerful open-source AI agent framework.

## Table of Contents

- [Overview](#overview)
- [Installation Methods](#installation-methods)
  - [Native CLI Installation](#native-cli-installation)
  - [Docker-Based Installation](#docker-based-installation)
- [Quick Start](#quick-start)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Support & Documentation](#support--documentation)

---

## Overview

OpenClaw is a flexible and extensible AI agent framework that can be deployed in multiple ways depending on your infrastructure and use case. This repository contains installation guides and configuration files for both native and containerized deployments.

### Features

- **Flexible Deployment**: Choose between native CLI or Docker containerization
- **LLM Provider Support**: Works with Ollama, OpenAI, Anthropic, and other providers
- **Security-First**: Includes hardening guidelines and secure configuration practices
- **Production-Ready**: Includes reverse proxy (Nginx) and SSL/TLS support

---

## Installation Methods

### Native CLI Installation

**Best for:** Local development, single-machine deployments, and quick experimentation.

#### Quick Start

```bash
# Remove any previous installations
openclaw uninstall --all --yes --non-interactive
npm uninstall -g openclaw
npm cache clean --force

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```

#### Configuration

After installation, the interactive wizard will prompt you to configure your LLM provider. 

**Example: Using Ollama**

```bash
Model/auth provider: Skip for now
Filter models by provider: All providers
Default model: ollama/llama3.2
```

For more detailed native CLI instructions, see [`installation-native-CLI.md`](./installation-native-CLI.md).

---

### Docker-Based Installation

**Best for:** Production deployments, server environments, and isolated setups.

#### Key Benefits

- Consistent environment across different machines
- Easy updates and rollbacks
- Integrated with Nginx reverse proxy
- SSL/TLS support for secure HTTPS access

#### Phases

The Docker-based installation is organized into three phases:

1. **VPS Hardening**: Secure your Ubuntu host against automated attacks
   - SSH key-only authentication
   - Firewall configuration (UFW)
   
2. **Repository & Environment Setup**: Clone the repository and prepare configuration
   - Clone OpenClaw source
   - Create isolated volumes
   - Configure environment variables (`.env` file)

3. **Build & Deploy**: Build the Docker image and start services
   - Build the local Docker image
   - Deploy using Docker Compose

For detailed Docker installation instructions, see [`installation-docker-based.md`](./installation-docker-based.md).

---

## Quick Start

### For Native Installation

```bash
# 1. Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# 2. Run the configuration wizard
openclaw configure

# 3. Start OpenClaw
openclaw start
```

### For Docker Installation

```bash
# 1. Clone the OpenClaw repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 2. Prepare environment
mkdir -p ~/.openclaw/workspace
cp .env.example .env
chmod 600 .env

# 3. Configure .env file (add your API keys and paths)
nano .env

# 4. Build and start services
docker-compose up -d
```

---

## Prerequisites

### System Requirements

- **OS**: Ubuntu 20.04 LTS or later (other Linux distributions supported)
- **RAM**: Minimum 4GB (8GB+ recommended)
- **Disk Space**: 10GB+ available space
- **Network**: Internet access for downloading models and dependencies

### For Docker Installation

- Docker Engine 20.10+
- Docker Compose 2.0+
- Nginx (for reverse proxy and SSL)

### For Native Installation

- Node.js 18+ and npm
- Python 3.8+ (for certain model providers)
- curl or wget

---

## Configuration

### Environment Variables (.env)

When using Docker or native installation, you'll need to configure the `.env` file with:

```env
# Gateway authentication
OPENCLAW_GATEWAY_TOKEN=your_secure_random_string

# LLM Provider Keys (choose based on your provider)
OPENAI_API_KEY=sk-proj-your-key-here
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Docker paths (for Docker-based installation)
OPENCLAW_IMAGE=openclaw:local
OPENCLAW_CONFIG_DIR=/home/youruser/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/youruser/.openclaw/workspace
OPENCLAW_GATEWAY_BIND=lan
```

### Generating Secure Tokens

```bash
openssl rand -hex 32
```

### LLM Provider Setup

**Ollama** (Local)
```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://localhost:11434",
        "apiKey": "ollama-local",
        "api": "ollama"
      }
    }
  }
}
```

**OpenAI**
- Obtain API key from [OpenAI Platform](https://platform.openai.com)
- Set `OPENAI_API_KEY` in `.env`

**Anthropic**
- Obtain API key from [Anthropic Console](https://console.anthropic.com)
- Set `ANTHROPIC_API_KEY` in `.env`

---

## File Structure

```
openclaw-installation/
├── README.md                      # This file
├── installation-native-CLI.md     # Native CLI installation guide
├── installation-docker-based.md   # Docker-based installation guide
├── docker-compose.yml             # Docker Compose configuration
└── nginx.conf                     # Nginx reverse proxy configuration (if present)
```

---

## Common Tasks

### Starting OpenClaw

**Native Installation**
```bash
openclaw start
```

**Docker Installation**
```bash
docker-compose up -d
```

### Stopping OpenClaw

**Native Installation**
```bash
openclaw stop
```

**Docker Installation**
```bash
docker-compose down
```

### Viewing Logs

**Docker Installation**
```bash
docker-compose logs -f openclaw-gateway
docker-compose logs -f openclaw-cli
```

### Updating Configuration

**Native Installation**
```bash
nano ~/.openclaw/openclaw.json
openclaw restart
```

**Docker Installation**
```bash
nano .env
docker-compose restart
```

---

## Security Considerations

### For Production Deployments

1. **Use strong authentication tokens** - Generate with `openssl rand -hex 32`
2. **Enable firewall** - Only expose necessary ports (80, 443)
3. **Use SSH keys** - Disable password authentication
4. **Enable HTTPS** - Use Let's Encrypt with Nginx
5. **Secure sensitive files** - Set proper file permissions (`chmod 600` for `.env`)
6. **Regular updates** - Keep OpenClaw and dependencies up to date

### VPS Hardening (Docker)

The docker-based installation guide includes comprehensive VPS hardening steps:
- Enforce SSH key authentication
- Configure UFW firewall
- Disable root login
- Configure SSL/TLS certificates

---

## Troubleshooting

### Issue: Installation fails with permission denied

**Solution**: Ensure you have proper permissions or use `sudo` for system-wide installation.

### Issue: Cannot connect to LLM provider

**Solution**: Verify your API keys are correct and the provider is accessible from your machine.

### Issue: Docker containers fail to start

**Solution**: Check logs with `docker-compose logs` and verify `.env` file configuration.

---

## Support & Documentation

- **Official OpenClaw Repository**: https://github.com/openclaw/openclaw
- **Installation Script**: https://openclaw.ai/install.sh
- **Issue Tracker**: Report issues on the official GitHub repository

---

## Contributing

If you find issues or improvements for these installation guides, please contribute to the repository or report them to the main OpenClaw project.

---

## License

These installation guides and configurations are provided as-is. Please refer to the OpenClaw project license for usage terms.

---

## Version Information

- **Last Updated**: February 2026
- **Tested On**: Ubuntu 20.04 LTS, 22.04 LTS
- **OpenClaw Version**: 2026.2.x

---

**Happy installing! 🚀**
