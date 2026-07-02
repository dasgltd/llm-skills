# WordPress Deploy & SEO Optimizer (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into an autonomous **WordPress DevOps & SEO Expert**. It provides strict procedures for deploying theme changes, ensuring maximum PageSpeed frontend performance, and injecting Yoast SEO metadata directly into the WordPress database via WP-CLI.

## The Problem
When AI agents edit WordPress files, they typically lack the context to push those changes to production seamlessly. Furthermore, if the user asks an AI to "optimize a post's SEO", the AI might edit HTML directly, bypassing WordPress database plugins like Yoast SEO. This leads to broken Yoast scores and manual copy-pasting for the user.

## The Solution (This Skill)
By equipping your AI Agent with this skill, it learns the **Automated WordPress Workflow**:
1. **Frontend Optimization**: It enforces modern web practices (`loading="lazy"`, WebP, preloading) whenever modifying PHP theme files.
2. **WP-CLI Database Injection**: The agent uses `wp-cli` to update Yoast SEO metadata (`_yoast_wpseo_title`, `_yoast_wpseo_metadesc`, focus keywords) directly in the database without requiring the user to open the WordPress admin dashboard.
3. **Automated Rsync Deployment**: It creates temporary `expect` scripts to automate `rsync` deployments over SSH, pushing local theme changes to production securely and immediately.
4. **Parallel Subagents**: For bulk SEO updates across multiple pages, the skill instructs the agent to spawn concurrent subagents to process and inject data simultaneously, drastically speeding up the workflow.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, WordPress, WP-CLI, SEO, Yoast SEO, PageSpeed, Rsync Deployment, DevOps.*

## License & Copyright
Copyright (c) 2026 DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
