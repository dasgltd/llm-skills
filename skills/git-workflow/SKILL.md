---
name: git-workflow
description: Use this skill whenever the user asks to save, backup, commit, or push code to Git or GitHub. This skill dictates how to perform git operations efficiently in Antigravity to minimize user intervention and approval clicks.
---

# Antigravity Git Workflow

## Why this skill exists
When interacting with Git in Antigravity, running separate commands for `git status`, `git add`, `git commit`, and `git push` creates a poor user experience. Every individual command proposed requires the user to manually click an "Approve" button in the UI.

To minimize user intervention and make the experience seamless, we must chain git commands together in a single execution step and proactively generate commit messages without asking.

## The Optimal Git Workflow

Whenever the user asks to backup, save, or push code, do **NOT** ask them for permission to proceed. Do **NOT** ask them what the commit message should be. Do **NOT** run `git status` as a separate preliminary step unless absolutely necessary for your own context.

Instead, proactively infer a high-quality commit message based on the work you just completed, and execute a single chained command to stage, commit, and push the files.

### Execution Pattern

Use the `run_command` tool with the following chained command structure:

```bash
git add . && git commit -m "type(scope): [your descriptive message here]" && git push
```

If you only modified specific files and want to be precise to avoid committing untracked files, you can list them explicitly:
```bash
git add file1.ts file2.tsx && git commit -m "fix(ui): [message]" && git push
```

### Key Principles
1. **Zero Verbal Permission**: Just propose the command directly. The user will review the command in the Antigravity UI popup and approve it there. 
2. **Minimize Clicks**: Always chain `git add`, `git commit`, and `git push` with `&&`. This reduces the UI approval clicks from 3 down to exactly 1.
3. **Conventional Commits**: Use semantic prefixes (e.g., `feat:`, `fix:`, `refactor:`, `style:`) summarizing the exact tasks you just completed in the conversation.
