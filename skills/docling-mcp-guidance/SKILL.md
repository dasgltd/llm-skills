---
name: docling-mcp-guidance
description: "Quando e como usar o MCP docling (OCR/conversao de documento pesado, sem LLM) versus markitdown (conversao leve de arquivo nativo-digital) e versus vision nativo/Gemini (entendimento visual). Trigger sempre que o usuario ou uma automacao n8n precisar extrair texto/estrutura de PDF, imagem, scan, nota fiscal, contrato ou qualquer documento — antes de decidir qual conversor chamar. Trigger tambem em qualquer tarefa envolvendo o servidor docling-mcp-server (deploy, topologia Vostro/VPS, fallback, X-MCP-Key, chave dmk_)."
---

# Docling MCP — Guia de Uso (Hermes/Craudinho + n8n)

Este skill cobre o MCP `docling` do hub DASG: quando puxar ele em vez de `markitdown`
ou de vision nativo, e a topologia/operacao por tras (Vostro + proxy VPS).
Fonte de verdade do deploy: `~/projects/docling-mcp-server/README.md` e
`~/projects/docling-mcp-server/n8n-routing-rule.md` (leia estes arquivos antes de
qualquer mudanca de infra no docling).

## 1. Decisao: qual conversor usar (3 niveis)

**NUNCA chame docling "por padrao"** — ele e pesado (torch + EasyOCR, ~9.5GB,
CPU-bound). Escolha pelo tipo de conteudo:

1. **Nativo-digital → `markitdown`** (leve, roda no VPS/EasyPanel, nao depende do
   Vostro): `.docx`/`.pptx`/`.xlsx` gerados por app (nao escaneados), HTML, CSV,
   JSON, XML, EPUB, PDF **digital** com texto selecionavel e layout simples.
2. **OCR / layout pesado → `docling`** (precisa "ler" pixel ou reconstruir
   estrutura): PDF **escaneado**, foto/print de documento, nota fiscal/contrato/
   extrato **em imagem**, tabela de scan, formulario, layout em colunas.
3. **Imagem visual (entender, nao extrair) → vision nativo/Gemini**: screenshot de
   UI, diagrama, print de erro, "o que ha nesta imagem?". Nem markitdown nem
   docling servem aqui — os dois so cospem texto cru, nenhum "entende" a imagem.

**Atalho para PDF:** tente markitdown primeiro; se o texto sair limpo, fica por
isso. Se sair bagunçado (ou o PDF for scan/foto), cai para docling.

## 2. Topologia (por que docling e diferente dos outros 3 MCPs do hub)

Infinity, n8n-brain e Omie rodam inteiramente na VPS (Docker Swarm/EasyPanel).
**Docling NAO** — e pesado demais para arriscar OOM na VPS de producao:

```
cliente -> your.server.url/docling/mcp (VPS, publico, gate X-MCP-Key)
        -> [proxy EasyPanel/Swarm, porta 8104->3000 na tailnet]
        -> 100.85.101.119:3000 (Vostro, via Tailscale) — Caddy do container
        -> docling-mcp em 127.0.0.1:8000 (Streamable HTTP, FastMCP, IBM docling)
```

- O backend real (OCR/torch/EasyOCR) roda no **Vostro**, nao na VPS.
- A VPS so faz proxy (porta `8104`, mesmo padrao dos outros 3 no hub) — zero peso
  de modelo la.
- O gate `X-MCP-Key` vive no Caddy do container, mesmo padrao do hub.
- Health check sem auth: `GET /health` → `200 ok`.
- FastMCP valida o header `Host` (anti DNS-rebinding) — por isso o Caddy reescreve
  `Host: localhost:8000` ao encaminhar (ver `dasg/hub/Caddyfile` no repo). Se mexer
  no Caddyfile e esquecer isso, o backend passa a recusar toda requisicao via IP.
- **Trade-off aceito:** se o Vostro cair (energia/internet), o endpoint docling
  fica fora do ar. Por isso o fallback abaixo e regra dura, nao sugestao.

## 3. Fallback OBRIGATORIO em automacao n8n

**Nenhuma automacao de OCR entra em producao sem fallback montado.** Padrao:

```
try:
    resultado = docling(https://your.server.url/docling/mcp, header X-MCP-Key)
catch (timeout | 5xx | conexao recusada):
    resultado = gemini_vision(arquivo)   # plano B quando o Vostro esta fora
```

- Trigger do fallback: timeout, `5xx`, ou conexao recusada no no docling — nunca
  "silenciar e seguir sem OCR".
- `markitdown` roda no VPS (EasyPanel) → nao precisa desse fallback de
  disponibilidade, mas falha em **qualidade** se a entrada for scan/foto — nesse
  caso o roteamento (secao 1) ja deveria ter mandado para docling desde o inicio.
- Ver `references/n8n-routing-rule.md` para o bloco pronto para colar na
  descricao/nota de qualquer workflow n8n que processe arquivos.

## 4. Credenciais e variaveis

- Header de auth do MCP publico e do proxy: `X-MCP-Key` (prefixo `dmk_`) — valor
  em `~/.hermes/secrets/docling_mcp_key` no Vostro. Nunca colar a key em
  texto/commit; sempre via credential do n8n ou variavel de ambiente.
- `DOCLING_MCP_KEY` — env var que injeta a key no container.
- `DOCLING_CONVERSION_MODE=local` — engine embutida (sem `docling-serve` externo).
- `FASTMCP_HOST=127.0.0.1` / `FASTMCP_PORT=8000` — bind interno do backend no
  Vostro (nao expor externamente sem o Caddy na frente).

## 5. Quando usar `easypanel-vps-deployer` / `systematic-debugging` junto

- Mudanca no proxy (Caddyfile, porta `8104`, X-MCP-Key na VPS) → puxar
  `easypanel-vps-deployer` (e o `channel_prompt` do canal `{tools}` ja cobre isso).
- Docling respondendo mas com output ruim (OCR errado, tabela quebrada) → nao e
  bug de infra, e limite do modelo/imagem de entrada — normalmente resolve
  trocando de nivel (voltar para markitdown se digital, ou aceitar a limitacao e
  documentar).
- Docling fora do ar (timeout/5xx) → primeiro checar se e o Vostro que caiu
  (energia/internet), NAO o proxy da VPS — os dois tem pontos de falha distintos.
