# MCP Builder (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into an expert **Model Context Protocol (MCP) Architect**. It provides the exact design patterns needed to build, expose, and integrate external services into an LLM using MCP servers.

## The Problem
When asked to build an MCP server, AI agents often write monolithic files without proper routing, hallucinate non-existent standard libraries, or fail to define precise JSON Schemas for the tools they expose. This results in brittle servers that other LLMs cannot understand or invoke correctly.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **MCP Best Practices**:
1. **Schema Perfection**: It understands that the `description` and `parameters` of a tool are not just for humans; they are prompts for other LLMs. It writes verbose, exact JSON Schemas.
2. **Framework Mastery**: It uses `FastMCP` for Python projects or `@modelcontextprotocol/sdk` for Node.js, setting up standard structures immediately.
3. **Resource & Prompt Handling**: It goes beyond just exposing "tools", teaching the AI how to expose dynamic Resources (like database tables) and Prompts natively.
4. **Error Boundaries**: It implements strict try-catch blocks to return clear string errors instead of crashing the local MCP process when an external API fails.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, Model Context Protocol, MCP, FastMCP, SDK, Tools Integration, Claude.*
