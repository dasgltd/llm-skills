# Firebase Spark Deploy Workaround (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill teaches AI Agents how to successfully deploy modern web applications (Next.js, Vite, React, Angular) to **Firebase Hosting** entirely on the **Free Tier (Spark Plan)**. 

## The Problem
Firebase heavily promotes *App Hosting* and *Web Frameworks*, which are amazing features, but they quietly provision **Cloud Functions** under the hood for Server-Side Rendering (SSR) and optimizations. Cloud Functions require a credit card and the **Blaze (Pay-as-you-go) Plan**. If a user on the Spark Plan tries to deploy using modern CLI defaults, the deployment will fail. 

## The Solution (This Skill)
By giving this skill to your AI Agent, it will learn a highly specific workaround to circumvent this limitation:
1. **Force Static Output**: It will modify the framework's config (e.g. `output: 'export'` for Next.js) to generate a purely static site.
2. **Classic Hosting**: It will initialize traditional Firebase Hosting instead of App Hosting.
3. **CI/CD via GitHub Actions**: It will configure a GitHub Actions workflow to run the build step on GitHub's servers and deploy the static result to Firebase.

This approach guarantees a professional, automated CI/CD pipeline without ever touching Cloud Functions, keeping the project 100% free on the Spark Plan.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, Firebase, Firebase Hosting, Spark Plan, Free Tier, CI/CD, GitHub Actions, Next.js, React, Angular, Vite, Deployment Workaround.*

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
