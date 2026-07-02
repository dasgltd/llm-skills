# Project Briefing Interview (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease.

This skill transforms an AI Agent into a **product-minded briefing interviewer**. Before writing a single line of code, the AI runs a right-sized discovery interview — from zero questions for trivial fixes up to a full 10-stage product discovery for new SaaS projects — and compiles the answers into a consolidated briefing plus a **master prompt** ready to paste into any coding LLM.

## The Problem
AI coding agents that start from a one-line request routinely build the wrong thing: wrong scope, missing constraints, no acceptance criteria, no security or privacy requirements. The opposite failure is just as real — running a heavyweight discovery questionnaire on a 15-minute fix burns tokens, time, and the user's patience. Most "PRD generator" prompts have no concept of proportionality.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns to **dose discovery to risk**:

1. **4-Tier Classification:** Every request is classified from Tier 0 (execute immediately) to Tier 3 (full 10-stage discovery: vision, personas, metrics, market, MVP scope, architecture, AI specifics, security/privacy, sprints, final confirmation).
2. **Effort & Model Gate:** The AI posts a difficulty and effort estimate up front and recommends a model tier (small / mid / frontier) — heavy work waits for an explicit human go-ahead before implementation begins.
3. **Hypothesis Discipline:** The AI never assumes critical requirements. Unknowns become explicit options, and every answer is tagged as fact, hypothesis, open item, or decision.
4. **Master Prompt Output:** The final deliverable is a structured, self-contained master prompt (role, scope, data model, security, acceptance criteria, implementation plan + 3 anti-hallucination reminders) that any coding LLM can execute end-to-end.

## How to Use
Copy the `SKILL.md` into your agent's skills directory (or paste it as custom instructions), then bring it a project idea: *"I want to build a tool that reconciles bank statements"*. The agent will classify the tier, interview you in short blocks, and hand you the consolidated briefing + master prompt.

## Acknowledgment
The interview structure was adapted from the "NoCode Startup AI" framework (a Brazilian vibe-coding course); the underlying practices — MVP scoping, personas, success metrics, sprint planning, acceptance criteria, privacy-by-design — are industry-standard product and software management.

## Keywords & Discoverability
*Open Source, LLM Skills, AI Agents, Prompt Engineering, Product Discovery, PRD, Briefing, MVP Scoping, Master Prompt, Vibe Coding, Requirements Engineering, Claude Code, Antigravity, Codex, Cursor.*
