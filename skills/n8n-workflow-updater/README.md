# n8n Workflow Updater (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into a proactive **n8n Automation Master**. It teaches the AI how to safely fetch, modify, organize, and publish complex n8n workflows through the n8n API using robust Node.js scripts instead of unreliable one-size-fits-all tools.

## The Problem
When AI agents try to use standard HTTP tools or MCPs to update n8n workflows (using `PUT /api/v1/workflows/<id>`), they frequently encounter `400 Bad Request` errors. This happens because the n8n API strictly rejects payloads containing read-only fields like `meta`, `tags`, `pinData`, or certain nested `settings`. Additionally, AI agents often overwrite user changes if they rely on outdated local JSON backups.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **Safe n8n Execution Lifecycle**:
1. **Always Fetch First:** The agent will explicitly pull the live workflow state before making any modifications, preventing data loss.
2. **Payload Sanitization:** It writes custom Node.js scripts to strip rejected fields before deploying to the n8n API.
3. **Design Best Practices:** The agent automatically renames generic nodes (e.g., `HTTP Request1`) to descriptive ones and adds `n8n-nodes-base.stickyNote` elements to organize large workflows visually.
4. **Credential Handoff:** The AI handles the logic but gracefully informs the user which nodes need their manual credential selection in the UI.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, n8n, n8n Workflows, Node.js, API Integration, Safe Deployment, Low Code.*
