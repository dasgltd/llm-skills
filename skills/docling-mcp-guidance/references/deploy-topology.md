# docling-mcp-server (DASG hub)

Wrapper de deploy do [docling-mcp](https://github.com/docling-project/docling-mcp)
(IBM) para o hub MCP da DASG. OCR real de PDFs e imagens/scan -> Markdown/JSON
estruturado, **sem LLM** (layout-aware + EasyOCR).

## Topologia (importante)

Diferente dos outros MCPs do hub, o docling é **pesado** (torch + EasyOCR, imagem
~9.5 GB, CPU-bound no OCR). Para não arriscar OOM na produção, ele **NÃO roda no
VPS** — roda no **Vostro** (24/7, mais forte), e o VPS só faz proxy pela tailnet:

```
cliente -> your.server.url/docling/mcp (VPS, publico)
        -> [proxy EasyPanel] -> 100.85.101.119:3000 (Vostro, via Tailscale)
        -> Caddy do container (gate X-MCP-Key + strip /docling)
        -> docling-mcp em 127.0.0.1:8000 (Streamable HTTP, FastMCP)
```

- O **gate `X-MCP-Key`** vive no Caddy do container no Vostro (mesmo padrão do hub).
- O VPS é gateway público; **não processa nada** (zero peso de OCR/modelos lá).
- Trade-off: se o Vostro cair (energia/internet), o endpoint fica fora. Por isso o
  **fallback é obrigatório em cada automação** (ver abaixo).

## ⚠️ Fallback OBRIGATÓRIO nas automações

**Toda automação (n8n) que consome o docling-mcp DEVE ter um fallback configurado.**
Como o docling roda no Vostro, ele pode ficar indisponível quando o Vostro perde
energia ou internet. O fallback **não** é configurado neste wrapper nem no hub —
é responsabilidade de **cada automação**, direto no n8n:

- Padrão: `try` chama `your.server.url/docling/mcp`; no `catch` (timeout/erro) o
  fluxo cai para **Gemini vision** (`gemini-2.0-flash`, aceita imagem/PDF) como
  plano B, mantendo a automação viva.
- Sem esse ramo de fallback, uma queda do Vostro **quebra a automação inteira**.
- Regra: **nenhuma automação nova de OCR entra em produção sem o fallback montado.**

## Execução (Vostro)

```bash
docker run -d --name docling-mcp --restart always \
  -p 0.0.0.0:3000:3000 \
  -e DOCLING_MCP_KEY="$DOCLING_MCP_KEY" \
  -e DOCLING_CONVERSION_MODE=local \
  -e FASTMCP_HOST=127.0.0.1 -e FASTMCP_PORT=8000 \
  -v docling-models:/root/.cache \
  --memory=5g \
  docling-mcp:local
```

## Variaveis

- `DOCLING_MCP_KEY` — chave do gate `X-MCP-Key` (sem ela, `/docling/*` = 401).
- `DOCLING_CONVERSION_MODE=local` — engine embutida (sem `docling-serve` externo).
- `FASTMCP_HOST=127.0.0.1`, `FASTMCP_PORT=8000` — bind interno do backend.

## Rotas

- `GET /health` — `200 ok`, sem auth (healthcheck/uptime).
- `/docling/mcp` — endpoint MCP publico (exige `X-MCP-Key`).

## Nota sobre o Caddyfile

O FastMCP/uvicorn valida o header `Host` (proteção anti DNS-rebinding) e recusa
acesso via IP da tailnet. Por isso o Caddy reescreve `Host: localhost:8000` ao
encaminhar para o backend (ver `dasg/hub/Caddyfile`).
