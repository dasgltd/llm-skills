# LLM Skills Repository

This repository is a collection of **Skills** (custom capabilities, system prompts, and detailed instructions) designed to guide autonomous Artificial Intelligence Agents and AI coding assistants.

It is built and structured for the major **AI Agents** and LLMs on the market, including:
* **Google Antigravity**, **Claude Code**, **GitHub Copilot**, **OpenAI Codex**, **Cursor**, **Aider**, **Devin**, **AutoGPT**, **ChatGPT**, **Gemini**, among other agentic frameworks.

These skills provide in-depth context, business rules, and step-by-step workflows that teach AI how to execute complex, end-to-end tasks safely and professionally without hallucinatory mistakes.

## How to use

Each folder inside the `skills/` directory contains an independent skill with its own `SKILL.md` (the actual prompt/instructions read by the AI) and a detailed `README.md` explaining what it accomplishes for the human user.

If you are using Google Antigravity, you can configure your local skills directory to pull from this repository, or simply clone this repository into your environment's plugins directory.

## Available Skills

| Skill | Description |
|-------|-----------|
| [wordpress-deploy-optimizer](./skills/wordpress-deploy-optimizer/) | Transforms your AI into a WordPress DevOps engineer. It automates theme deployments via `rsync` and injects SEO metadata (Yoast) and PageSpeed performance directly into the database using `wp-cli`, bypassing the need for WP Admin dashboard intervention. |
| [n8n-workflow-updater](./skills/n8n-workflow-updater/) | Provides a complete and safe lifecycle for an AI to edit and deploy n8n workflows. It bypasses strict API errors by sanitizing JSON payloads, enforces proactive local backups, and applies visual best practices to node organization. |
| [firebase-spark-deploy-workaround](./skills/firebase-spark-deploy-workaround/) | Teaches the AI how to deploy modern web applications (Next.js, React, Angular, Vite) to Firebase Hosting on the free Spark Plan, circumventing Cloud Functions limitations by forcing static builds via automated GitHub Actions CI/CD. |
| [apps-script-workflow](./skills/apps-script-workflow/) | Transforms the AI into a proactive Google Apps Script developer. Defines rigorous methods for local deployment via clasp, automatic UI menus, Toasts for user feedback, and defensive Google Sheets programming. |
| [code-documentation-expert](./skills/code-documentation-expert/) | Enforces strict technical writing standards for AI Agents, guaranteeing robust docstrings (JSDoc, PEP), architecture explanations over obvious syntax repeating, and automatic insertion of version control headers and copyrights. |
| [easypanel-vps-deployer](./skills/easypanel-vps-deployer/) | Transforms your AI into a VPS DevOps engineer. Outlines exact patterns for deploying Docker containers, managing databases, and injecting `.env` securely inside EasyPanel environments using generic SSH keys. |
| [git-workflow](./skills/git-workflow/) | Configures an autonomous Git operator, training the AI to chain standard version control commands silently and efficiently while implementing Semantic Commits and protecting upstream branches. |
| [open-source-publisher](./skills/open-source-publisher/) | A strict DevSecOps firewall skill. Forbids the AI from pushing code to public repositories without a mandatory sanitization phase to prevent data leaks (API Keys, secrets), while automatically handling Open Source SEO. |
| [mcp-builder](./skills/mcp-builder/) | Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Licensed under Apache 2.0 (Copyright 2026 Anthropic, PBC), modified by DASG. |

---
*This is an open-source repository in constant evolution. New LLM Skills for automation, productivity, and web development will be added over time.*
