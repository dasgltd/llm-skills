# Git Workflow (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into an **Autonomous Git Operator**. It dictates exactly how the agent should perform local version control and remote synchronization (GitHub) to minimize user intervention, avoid endless permission prompts, and guarantee clean commit histories.

## The Problem
By default, when asked to "save code to Git", AI agents will often prompt the user for permission on every single command: `git add`, `git commit`, and `git push`. They might also write poor, generic commit messages like "update files" or get completely stuck if there's a merge conflict or untracked upstream branch.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **Silent & Efficient Git Workflow**:
1. **Chained Execution**: The agent learns to chain git commands (`git add . && git commit -m "..." && git push`) to execute the entire operation with a single approval click from the user.
2. **Semantic Commits**: It enforces the use of conventional commits (e.g., `feat:`, `fix:`, `docs:`) providing clear, professional git logs.
3. **Conflict Resolution**: It includes fallback strategies for handling upstream branch conflicts safely, without deleting user data.
4. **Credential Awareness**: It teaches the agent how to handle GitHub authentication gracefully via local keychain or SSH, preventing failed pushes.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, Git, GitHub, Version Control, CI/CD, DevOps.*

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
