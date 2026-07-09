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

## 6. Controle de Superfície de Ferramentas por Instância/Workflow (omie-mcp-server)

O `omie-mcp-server` (repo `dasgltd/omie-mcp-server`) agrupa cada tool em um "módulo" lógico (`clientes`, `produtos`, `vendas`, `notas_fiscais`, `financeiro`, `compras`, `servicos`, `estoque`, `cadastros_auxiliares`, `empresa`) via o mapa `TOOL_MODULES` em `src/index.ts`. Existem **duas camadas** de filtro, para dois níveis de necessidade diferentes:

### Camada 1 — Nível de instância (env var `OMIE_ENABLED_MODULES`)
- Controla quais módulos existem **globalmente** naquela instância do MCP server (ex: uma instância de teste que só deve expor Financeiro).
- Configurar: env var `OMIE_ENABLED_MODULES` (lista separada por vírgula, ex: `financeiro,cadastros_auxiliares`). Vazia/ausente = todos os módulos ativos (retrocompatível).
- **Onde editar de forma visual:** no EasyPanel, aba *Environment* do serviço `omie-mcp` — é um editor de texto simples (`KEY=VALUE`), já documentado com a lista de módulos disponíveis em `.env.example`. Depois de editar, precisa **restart do container** (a env var só é lida no boot).
- **Para checar o status atual sem entrar no EasyPanel:** chame a tool `list_modules_status` (sem parâmetros) — retorna a lista de módulos com ✅/❌ direto no chat do n8n/Claude, incluindo instrução de como alterar.

### Camada 2 — Nível de workflow n8n (nó "MCP Client Tool", sem precisar mexer no servidor)
- Para restringir quais tools aparecem **em um workflow n8n específico** (sem afetar outros workflows nem exigir restart), use o parâmetro nativo do próprio nó `MCP Client Tool` do n8n (`@n8n/n8n-nodes-langchain.mcpClientTool`): campo **"Tools to Include"**, com 3 opções:
  - `All` — expõe tudo que a instância permite (default).
  - `Selected` — multi-select visual (checkboxes) das tools específicas a incluir.
  - `All Except` — multi-select visual das tools a excluir.
- Essa é a forma **mais simples e visual** de aplicar um filtro por workflow — é um campo nativo do n8n, sem precisar editar código, `.env` ou reiniciar nada. Preferir esta camada quando o filtro é específico de um agente/workflow; usar a Camada 1 (env var) quando o filtro é para a instância inteira (ex: instância de homologação vs. produção).

## 7. Permissões de Escrita por Header (`X-MCP-Key`) — omie-mcp-server

Desde o commit `96acf2e`, as tools de escrita (create_*/update_*/pay_* etc., 11 tools no Set `WRITE_TOOLS`) seguem o mesmo padrão de auth do `start-infinity-mcp-server`: o **nível de permissão é resolvido por sessão a partir do header HTTP `X-MCP-Key`**, e tools de escrita nem aparecem na lista quando o nível não autoriza.

### Níveis e env vars (instância)
- Níveis: `read` (só consultas) e `write` (tudo). Sem nível `admin` — não há tools destrutivas no domínio Omie hoje.
- `OMIE_WRITE_KEY` — chave secreta; sessão que enviar `X-MCP-Key` igual a ela ganha nível `write`.
- `OMIE_DEFAULT_LEVEL` — nível para sessões sem chave/chave errada. Default `read` (mais seguro). 
- `OMIE_READ_ONLY=true` — kill switch: força `read` SEMPRE, vence tudo (inclusive chave correta).
- `OMIE_MCP_KEY` — opcional, defesa em profundidade: se configurada, o path interno `/mcp` e `/message` passa a exigir `X-MCP-Key` (aceita esta chave ou a própria `OMIE_WRITE_KEY`). Vazia = path interno aberto (retrocompatível com n8n sem header). **Só preencher DEPOIS de configurar o header no n8n**, senão o n8n perde acesso.

### Como configurar o header no n8n (visual)
1. No nó **MCP Client Tool**, abra as opções de conexão/credencial do endpoint.
2. Adicione um header customizado: nome `X-MCP-Key`, valor = a `OMIE_WRITE_KEY` (para agente com escrita) ou nada (agente somente leitura).
3. Sem restart de nada — o nível é resolvido por sessão, na conexão.

### Regra prática
- Agentes de consulta/relatório → **sem header** (ficam `read`, não veem tools de escrita — o LLM nem tenta).
- Agentes operacionais (criar cliente, baixar conta) → header com `OMIE_WRITE_KEY`.
- Emergência ("congelar tudo") → `OMIE_READ_ONLY=true` no EasyPanel + restart.
- As 3 camadas se compõem: módulos (`OMIE_ENABLED_MODULES`) ∩ nível do header (`X-MCP-Key`) ∩ filtro do nó n8n ("Tools to Include").
