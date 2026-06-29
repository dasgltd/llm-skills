# Apps Script Workflow (LLM Skill)

> [!TIP]
> **LLM Skills** are customized instructions for AI Agents (like Google Antigravity, Claude Code, GitHub Copilot, Codex, etc.) to help them navigate complex, specialized workflows with ease. 

This specific skill transforms an AI Agent into a proactive **Google Apps Script (GAS) Developer**. It teaches the AI how to safely deploy via `clasp`, handle UI interactions (Toasts and Modals) gracefully, and catch errors without failing silently in the background.

## The Problem
When AI agents write Google Apps Script, they tend to use `Logger.log()` for output, which is completely invisible to end-users running the scripts from Google Sheets. They also tend to ignore Google Apps Script's cache mechanisms and write code that crashes silently or creates duplicated menu items.

## The Solution (This Skill)
By injecting this skill into your AI Agent, it learns the **Pro Methodology for Apps Script**:
1. **Local Deployment**: It uses `npx clasp push -f` and creates versions properly so client scripts bypass cache.
2. **Custom Menus**: The agent will always automatically generate a custom menu named "Pro Tools" to expose functions to the spreadsheet UI.
3. **UI Feedback (Toasts)**: It uses `SpreadsheetApp.getActiveSpreadsheet().toast()` to show progress loops and data loading indicators so users don't think the script has frozen.
4. **Defensive Programming**: The agent learns specific Google Sheets quirks, such as preferring `getDisplayValues()` over `getValues()` to avoid "Invalid Date" object corruption and correctly filtering out empty rows.

## Keywords & Discoverability
*Open Source, Automation, LLM Skills, AI Agents, Prompt Engineering, Google Apps Script, GAS, Clasp, Google Sheets, Low Code, JavaScript.*
