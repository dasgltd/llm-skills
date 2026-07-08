# Docling MCP Guidance

Este skill define quando usar o MCP `docling` (OCR/conversao pesada de documento,
sem LLM) versus `markitdown` (conversao leve de arquivo nativo-digital) versus
vision nativo/Gemini (entendimento visual), alem de documentar a topologia por
tras do servico e a regra de fallback obrigatoria em automacao n8n.

## O que ele faz
1. **Roteamento por tipo de conteudo**: ensina a decisao em 3 niveis (nativo-digital
   → markitdown; OCR/layout pesado → docling; imagem visual → vision nativo/Gemini),
   incluindo o atalho de decisao para PDF.
2. **Topologia real**: docling e o UNICO dos 4 MCPs do hub que nao roda inteiro na
   VPS — o backend (torch + EasyOCR) roda no Vostro, a VPS so faz proxy pela
   tailnet. Documenta o caminho completo, o gate `X-MCP-Key`, e a reescrita de
   `Host` no Caddy (necessaria por causa da protecao anti DNS-rebinding do FastMCP).
3. **Fallback obrigatorio**: nenhuma automacao n8n de OCR entra em producao sem um
   `catch` que cai para Gemini vision quando o Vostro estiver fora do ar.
4. **Credenciais e variaveis**: onde vive a `X-MCP-Key` (prefixo `dmk_`), e as env
   vars do container (`DOCLING_MCP_KEY`, `DOCLING_CONVERSION_MODE`, `FASTMCP_HOST/PORT`).

## References
- `references/n8n-routing-rule.md` — bloco pronto para colar na descricao/nota de
  qualquer workflow n8n que processe arquivos, com o roteamento e o fallback.
- `references/deploy-topology.md` — README completo do wrapper de deploy
  (`docling-mcp-server`), com o diagrama de topologia, comando de execucao no
  Vostro e variaveis de ambiente.

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
