---
name: apps-script-workflow
description: Standardized workflow for Google Apps Script projects using clasp. Defines rules for deployment, custom menus, UI logs (toasts), and error handling (alerts).
---

# Pro Google Apps Script Workflow

This skill defines the standard methodology for building and deploying Google Apps Script (GAS) projects, particularly those bound to Google Sheets or integrating external APIs (like Omie).

Always follow these guidelines when creating, modifying, or deploying Apps Script projects.

## 1. Local Development & Deployment (`clasp`)

All GAS development should happen locally, using `clasp` to push changes.

- **Force Push**: When deploying code, always use the force flag to avoid conflicts with online changes:
  ```bash
  npx clasp push -f
  ```
- **Versioning**: If the project is used as a Library by other scripts, create a new version after pushing so client scripts bypass cache:
  ```bash
  npx clasp version "Descrição da mudança"
  ```
- **Cache Awareness**: Google Apps Script caches library versions heavily. If you push code to a library, you MUST bump the version via `npx clasp version` and update the dependency version in the client's `appsscript.json` (or bring the dependent code into the main script to bypass the cache entirely).

## 2. Menu Personalizado: "Pro Tools"

Every user-facing script bound to Google Sheets MUST create a custom menu upon opening the document (`onOpen`). The root menu should always be named **"Pro Tools"**.

### Implementation Pattern (`menu.js` or `onOpen.js`)
```javascript
function onOpen() {
  SpreadsheetApp.getUi()
    .createMenu('Pro Tools')
    .addItem('Nome da Ação 1', 'nomeDaFuncao1')
    .addItem('Nome da Ação 2', 'nomeDaFuncao2')
    .addSeparator()
    .addItem('DEBUG: Inspecionar Sistema', 'funcaoDeDebug')
    .addToUi();
}
```

## 3. UI Feedback: Logs and Toasts

Never let scripts fail silently or run without progress indicators. Apps Script's `Logger.log()` is invisible to the end-user. You must use UI elements to communicate.

- **Toasts for Progress**: Use Toasts to show progress loops, successful page fetches, and data loading so the user knows the script hasn't frozen.
  ```javascript
  try {
    SpreadsheetApp.getActiveSpreadsheet().toast(`Processando página ${pagina}...`, "Sincronização", 5);
  } catch(e) {} // Silent catch to prevent crashing if triggered via triggers/cron
  ```

## 4. UI Feedback: Error Handling (Alerts)

When an error occurs, it must be captured and shown to the user via a modal alert (`MsgBox` style), instead of terminating with a generic "script concluído" message.

- **Alerts for Exceptions**:
  ```javascript
  function logErro(funcao, mensagem, erroOriginal) {
    try {
      const msgErro = typeof erroOriginal === 'object' ? (erroOriginal.message || JSON.stringify(erroOriginal)) : String(erroOriginal);
      SpreadsheetApp.getUi().alert(
        "Erro no Sistema",
        `Falha na função: ${funcao}\n\nMensagem: ${mensagem}\n\nDetalhes:\n${msgErro}\n\nVerifique o painel de Execuções para mais detalhes.`,
        SpreadsheetApp.getUi().ButtonSet.OK
      );
    } catch(e) {}
    
    // Always keep internal logger for developers
    Logger.log(`[${funcao}] ❌ ERRO: ${mensagem} | Detalhes: ${erroOriginal}`);
  }
  ```

## 5. Defensive Programming (Google Sheets Specifics)

- **Date Formats**: Google Sheets `getValues()` can return corrupted `Date` objects ("Invalid Date") if the user applies bad formatting to a column containing integers. When extracting values where you need string-based exact matching (e.g., IDs, codes), PREFER `getDisplayValues()` to extract the exact text seen by the user.
- **Empty Rows**: Always handle empty cells and filter them out.
- **Duplicate Prevention**: Be aware that financial APIs (like Omie's `ListarMovimentos`) may return multiple historical statuses for the same title (e.g., "A VENCER" and "PAGO"). Explicitly filter by `cStatus` to avoid duplicating records or double-counting in maps. Use `Math.max()` instead of `+=` when consolidating single values that might appear duplicated in history logs.
