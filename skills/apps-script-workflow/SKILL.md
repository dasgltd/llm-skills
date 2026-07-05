---
name: apps-script-workflow
description: Standardized workflow for Google Apps Script projects using clasp. Defines rules for deployment, custom menus, UI logs (toasts), and error handling (alerts). Also covers discipline for importing external API data into Google Sheets for Looker Studio dashboards (tab safety, date/header standards) and function versioning/documentation. Use when building, deploying, or importing data into Sheets/Looker/dashboards via Apps Script.
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

## 6. Data Import Discipline (Looker-ready)

When pulling data from any API into Google Sheets for analysis or Looker Studio dashboards, data integrity comes first. Never destroy existing data on the promise of new data — validate success before touching the destination.

- **Never clear before success**: NEVER wipe/clear a tab before the API call has returned success AND the final data structure is validated. On any failure, the previous data in the sheet must be preserved. Only clear and overwrite after the API result and the prepared dataset are confirmed good.
- **Verify-or-create the tab**: Never assume a destination tab exists. Check for it and create it if missing (use a shared `verifyOrCreateTab` / `{Platform}Utils.verificarOuCriarAba` helper — see §7).
- **Date column is always first**: The date field must be the first column of every imported table.
- **Dates for Looker**: Format dates in Brazilian style `dd/MM/yyyy`, and whenever possible write them as real `Date` objects into the cell (not strings) so Looker Studio recognizes them as a date dimension for time-series analysis.
- **Standardized headers**: Build clear, standardized headers — no accents, no unnecessary variation. Route them through a shared `standardizeHeaders` / `padronizarHeaders` helper so every import is consistent.
- **Dynamic columns across sources**: When importing multiple folders/sources into one sheet, keep column consistency, create new columns dynamically when a source introduces one, and preserve header order and standardization.
- **Progress logging**: Log progress with `Logger.log()` at function start/end, on API calls, on pagination, on total items processed, on tab creation/validation, on data writes, and at critical transformation steps.

## 7. Function Versioning & Internal Documentation

Apps Script has no built-in git history in the online editor, so versioning and documentation live inline in the code.

- **Version at the top of every main function**: Declare a semantic version in the function header, e.g. `listarRealizadoOmie v1.5.0`.
  - **Patch** (`x` in `v1.5.x`): small fixes or revisions.
  - **Minor / structural** (`y` in `v1.y.0`): larger changes or structural reworks.
  - Always bump the version when the function changes, and keep the function's documentation coherent with the new version.
- **Objective docstrings**: Document every function at the top with short, objective comments covering: purpose, key parameters, return value (if any), and relevant notes on integration, pagination, logging, or normalization. Do NOT use the word "Implementa:" in comments.
- **Centralized utilities first**: Before writing a new helper, reuse the centralized `{Platform}Utils` library (e.g. `OmieUtils`, `InfinityUtils`, `RDmktUtils`, `HubspotUtils`) — `verifyOrCreateTab`, `standardizeHeaders`, `fetchPaginated`, `formatDate`, `logError`, `getCredentials`, etc. Only create a new script if nothing existing fits.
- **Fix the pattern everywhere**: When correcting a problem, review the whole file and fix every occurrence of the same incorrect pattern — never patch a single isolated spot while the same mistake survives elsewhere.

## 8. API Precision (no hallucinated endpoints)

- Never invent method names, routes, parameters, field names, or return structures.
- Rely only on official API documentation, the exact call URL provided, real examples, or raw JSON supplied by the user.
- When unsure about an API's response shape, ask for or work from the real raw JSON — do not assume unconfirmed structures.
- **Infinity API specifics**: extract item attributes via `values.attribute.name` and `values.data`; normalize fields (strip HTML tags, format ISO dates, extract plain text from structured fields); for link/attachment fields extract the raw URL, and for `attachment` types extract only the `link` value; for arrays of objects (e.g. multiple benefits) map each item's `.name` or `.label` and join by comma.
