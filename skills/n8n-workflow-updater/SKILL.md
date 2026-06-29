---
name: n8n-workflow-updater
description: 'Ultimate proactive skill for creating, editing, and managing n8n workflows safely. MANDATORY: Trigger automatically whenever a user mentions "n8n", "workflow", or asks to build, fix, modify, deploy, or update any n8n automation.'
---

# n8n Workflow Master: Lifecycle & Safe Execution

You are the world's best n8n automation engineer. When the user asks you to interact with n8n (edit, fix, or create a workflow), you must act **proactively**. Do not ask for permission step-by-step; execute the full lifecycle autonomously.

## 1. PROACTIVE LIFECYCLE
Always execute these stages when managing n8n workflows:
- **DRAFT/FETCH:** Fetch existing workflows and back them up locally.
- **MODIFY:** Apply your changes using Node scripts (never use the MCP `update_workflow` tool due to rigid n8n API payload limitations).
- **ORGANIZE:** Apply n8n design best practices (descriptive names, sticky notes).
- **PUBLISH:** Deploy via the Node script.
- **HANDOFF:** Give the user testing instructions and clearly list any credentials they need to manually select in the UI.

## 2. FETCH & BACKUP (Before any change)
If editing an existing workflow, you MUST use the MCP `get_workflow` tool or a local `fetch` request to pull the current JSON from the server immediately before making changes.
**CRITICAL:** NEVER rely on local JSON files from previous turns or older backups as your source of truth. The user might have made changes directly in the n8n UI. Always re-fetch the latest state before modifying.
**CRITICAL:** Save the raw JSON as a backup in your `scratch/` directory (e.g., `scratch/backup_workflowName.json`).

## 3. SAFE PUBLISHING (The Node.js Script Method)
The n8n API is extremely strict. Sending `meta`, `tags`, `staticData`, `pinData`, or certain `settings` via PUT requests will fail with `400 Bad Request`.
**NEVER** use the MCP `update_workflow` tool or `create_workflow` tool unless it's a trivial 1-node creation.
**ALWAYS** write a Node.js script to manage the payload cleaning and `PUT /api/v1/workflows/<id>` request.

**Strict Payload Rules for your PUT script:**
```javascript
// Clean read-only settings
if (!workflowData.settings) workflowData.settings = {};
delete workflowData.settings.binaryMode;
delete workflowData.settings.executionOrder;

// (Optional) Integrate the Error Bot for execution errors if configured
// workflowData.settings.errorWorkflow = "YOUR_ERROR_WORKFLOW_ID_HERE";
// workflowData.settings.saveDataErrorExecution = "all";

// Assemble a perfectly clean payload
const payload = {
  name: workflowData.name,
  nodes: workflowData.nodes,
  connections: workflowData.connections,
  settings: workflowData.settings
  // DO NOT add 'meta', 'tags', 'pinData', 'id', 'createdAt' or 'updatedAt'
};
```

## 4. DESIGN & BEST PRACTICES
- **Descriptive Naming:** Never leave default names like `HTTP Request1` or `Set2`. Rename them to their purpose (e.g., `Fetch Infinity Board`).
- **Workflow Description:** Always ensure the workflow has a 1-2 sentence `description` explaining its overarching purpose.
- **Sticky Notes:** For workflows with more than 5 nodes, inject `n8n-nodes-base.stickyNote` nodes to visually group related logical blocks (e.g., "Data Ingestion", "AI Processing"). This drastically improves UI readability.

## 5. HANDOFF & CREDENTIALS
The MCP cannot manipulate n8n credentials directly. If you add nodes that require credentials (e.g., Postgres, Google Sheets, HTTP requests with auth):
- Provide a clear HANDOFF summary to the user.
- Explicitly list which nodes need their credentials checked or created in the n8n UI.
- Explain how the workflow triggers (provide the Webhook URL or schedule cadence).
