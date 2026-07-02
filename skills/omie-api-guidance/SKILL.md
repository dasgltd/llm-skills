---
name: omie-api-guidance
description: "Best practices, architectural patterns, and known quirks/limitations for integrating with the Omie ERP API. Trigger this skill whenever the user asks to create, modify, or debug any integration, script, tool, backend service, or MCP server communicating with Omie. Trigger on mentions of: 'Omie', 'Contas a Pagar', 'Contas a Receber', 'Omie API', 'ERP Omie', or any of its modules."
---

# Omie API Universal Guidance

This skill contains the collective knowledge of integrating the Omie ERP API into any application, script, automation, or MCP server. Omie's APIs are powerful but have several legacy quirks, rate limits, and native limitations that must be handled in software to avoid unexpected errors, timeouts, or blocking.

## 1. Rate Limits and Blocks (Crucial)

Omie imposes strict rate limits to protect its infrastructure. **These apply regardless of where your code is running (Scripts, Backends, MCPs, n8n, etc.).**
- **Global Limit:** 960 requests per minute per IP address.
- **Specific Limit:** 240 requests per minute per (IP + App Key + Method).
- **Concurrency:** Maximum of 4 concurrent requests per (IP + App Key + Method).
- **The 30-Minute Block (HTTP 425):** Se fizer 10 requisições incorretas consecutivas para o mesmo (IP + App Key + Método), o acesso será bloqueado por exatos 30 minutos. Sempre valide parâmetros localmente!
- **HTTP 429 Too Many Requests:** Implement Retry with Exponential Backoff. Do not crash. 
- **HTTP 500 Internal Server Error:** Muitas vezes é um campo mal formatado (vírgula no lugar de ponto, data errada).

## 2. Pagination and Caching Aux Tables

- **Max Page Size:** 500 records per page. Use `pagina` e `registros_por_pagina`.
- **N+1 Queries:** For auxiliary tables (Projects, Categories, Sellers, Departments, Customers, Bank Accounts), do **NOT** fetch them on demand per record. Fetch all pages (or use list/sync endpoints) on application boot, cron job, or cache initialization, and cache them in memory (e.g., using `Map`) or a database (Redis/Postgres).

## 3. Resolving IDs to Names & Token Optimization (For LLMs / AI Agents)

When building AI tools or MCP servers that send data back to an LLM:
- **Best Practice:** Before returning the data to the LLM (which wastes tokens trying to guess the names), map numeric IDs to their human-readable names using the caches (e.g., `nome_projeto`, `nome_fornecedor`, `nome_categoria`).
- **AI Pagination:** Return chunks (e.g., `limite_retorno_ia: 10`) so the LLM doesn't hit context limits.
- **Data Grouping:** Provide `agrupar_por: "PROJETO"` to summarize thousands of rows into a few totals via software before sending to the LLM.

## 4. Context-Specific Architectural Patterns

### For MCP Servers (e.g., n8n Integrations)
- **DO NOT** reuse a single `new McpServer()` instance across concurrent connections (like multiple n8n webhooks). 
- Instantiate `new SSEServerTransport("/message", res)` on `GET`, map it by `transport.sessionId`, and create a fresh server instance `createServer().connect(transport)`.

### For General Backends / Automations (Node.js/Python)
- **Queueing:** For bulk operations (e.g., creating 100 invoices), use a task queue (like BullMQ, Celery) with a concurrency of 2-3 to naturally respect Omie's 4-concurrent limit.
- **Idempotency:** Always check if a record exists before creating it, as Omie will often throw a 500 ou 400 error if you try to insert a duplicate key (like `codigo_integracao`).

## 5. Módulos Específicos: Limitações e Filtros (REFERÊNCIAS)

A Omie tem limitações gigantescas de filtros dependendo do módulo (Ex: Data de Vencimento não existe em Finanças, e Valores string quebram a criação de OS em Serviços).

Para detalhes microscópicos de cada módulo, leia obrigatoriamente a referência correspondente ANTES de programar:

- **Para Finanças** (Contas a Pagar, Receber, Conta Corrente, Resumo): 
  Leia o arquivo: `references/financas.md`
- **Para Serviços** (Ordens de Serviço, Inclusão de OS):
  Leia o arquivo: `references/servicos.md`
- **Para Produtos e Estoque** (Pedidos de Venda, Controle de Estoque):
  Leia o arquivo: `references/produtos.md`
- **Para CRM** (Oportunidades, Funil de Vendas):
  Leia o arquivo: `references/crm.md`
- **Para Geral** (Clientes, Fornecedores, Categorias, Cadastros Auxiliares):
  Leia o arquivo: `references/geral.md`
