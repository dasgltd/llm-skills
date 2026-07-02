# Open Source Publisher (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into an **Open Source Security & Marketing Auditor**. It acts as a mandatory firewall whenever you ask your AI to publish local code to a public repository.

## The Problem
When users ask an AI to "make this code public" or "push to GitHub", the AI typically just executes the `git push` blindly. This often leads to disastrous data leaks: API keys, private IPs, Webhook URLs, and internal company names get exposed to the world. Furthermore, the repository is usually uploaded without proper READMEs or SEO tags, making it invisible to the community.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **Strict Open Source Protocol**:
1. **Sanitization First**: The AI is forbidden from publishing until it actively searches for and anonymizes API keys, SSH keys, Telegram/Slack IDs, and internal company mentions.
2. **Data Leak Containment**: If a leak accidentally occurs, the AI is pre-trained to immediately use the GitHub API to turn the repository Private, destroy the local `.git` history, scrub the data, and recreate the repository cleanly.
3. **Repository SEO**: The AI automatically generates an English `README.md` optimized with relevant keywords and uses the GitHub API to apply discovery Topics (Tags) to the repository so other developers can find it.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, Security, Data Leak Prevention, DevSecOps, GitHub.*

## License & Copyright
Copyright (c) 2026 DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
