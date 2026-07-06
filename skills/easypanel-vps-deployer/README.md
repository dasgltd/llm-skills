# EasyPanel VPS Deployer (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into a **DevOps Engineer** capable of deploying and managing Docker containers, databases, and environments directly on your Virtual Private Server (VPS) via the **EasyPanel** control panel.

## The Problem
When users ask AI agents to "deploy this to my server", agents often hallucinate complex bare-metal Nginx configurations, attempt to install raw Docker engines, or provide generic Unix commands that conflict with modern VPS management panels like EasyPanel, Coolify, or CapRover. Furthermore, agents often struggle with authentication, asking for root passwords in plain text.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **EasyPanel Ecosystem Workflow**:
1. **Containerized Thinking**: It understands that apps must be deployed as Docker services, strictly avoiding manual Nginx or PM2 setups on the host machine.
2. **Environment Variable Management**: It knows how to structure and inject `.env` files safely into remote containers.
3. **Database Links**: It automatically provisions attached databases (PostgreSQL, MySQL, Redis) using the EasyPanel service architecture.
4. **Remote Execution (Passwordless)**: It provides the exact SSH/CLI commands required to interact with the VPS without ever asking you for a password, relying securely on SSH Keys.

## Setting up SSH Keys for this Skill
For this skill to work autonomously, your AI needs passwordless access to your VPS. You must set up an SSH Key.

### 1. Generate an SSH Key
Open your terminal (Mac/Linux) or PowerShell (Windows) and run:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
Press Enter to accept the default file location. When prompted for a passphrase, you can leave it empty for full AI automation, or set one if you prefer manual unlocks.

### 2. Copy the Key to your VPS
You need to send the public part of your key to your VPS so it recognizes your computer.

**On Mac / Linux:**
```bash
ssh-copy-id root@<YOUR_VPS_IP>
```

**On Windows (PowerShell):**
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@<YOUR_VPS_IP> "cat >> .ssh/authorized_keys"
```

Once this is done, the AI Agent will be able to `rsync` files and run deployment commands automatically in the background without getting stuck on password prompts!

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, EasyPanel, VPS, Server Deployment, Docker, DevOps, CI/CD, Containerization, SSH.*

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
