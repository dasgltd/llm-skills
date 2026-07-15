# Regra de roteamento — OCR / conversão de documento (n8n)

> Cole este bloco na descrição/nota de toda automação n8n que processe arquivos.
> Define **qual conversor usar por tipo de entrada** e o **fallback obrigatório**.

## Roteamento por tipo de conteúdo (3 níveis)

1. **NATIVO-DIGITAL → `markitdown`**
   Arquivo cujo texto já é extraível, sem OCR.
   - `.docx`, `.pptx`, `.xlsx` (Office gerado por app, não escaneado)
   - HTML, CSV, JSON, XML, EPUB
   - PDF **digital** (texto selecionável, layout simples)
   - Vantagem: leve, instantâneo, **não depende do Vostro estar de pé** nem de OCR.

2. **OCR / LAYOUT PESADO → `docling`**
   Precisa "ler" pixel ou reconstruir estrutura.
   - PDF **escaneado**, foto/print de documento
   - Nota fiscal, contrato, extrato **em imagem**
   - Tabelas de scan, layout em colunas, formulários
   - Endpoint: `https://your.server.url/docling/mcp` (header `X-MCP-Key`)

3. **IMAGEM VISUAL → vision (nativo / Gemini)**
   Entender tela/foto, não extrair texto.
   - Screenshot de UI, diagrama, print de erro, foto
   - "O que há nesta imagem?" / raciocínio visual
   - `markitdown` e `docling` NÃO servem aqui (só cospem texto cru).

### Decisão rápida para PDF
- Texto sai limpo no markitdown → usa markitdown.
- Veio bagunçado, ou é scan/foto → cai para docling.

## Fallback OBRIGATÓRIO (regra dura)

Nenhuma automação de OCR entra em produção sem fallback montado.
O docling processa no **Vostro** (24/7, mas cai em falta de energia/internet).

```
try:
    resultado = docling(  https://your.server.url/docling/mcp , header X-MCP-Key )
catch (timeout | 5xx | conexão recusada):
    resultado = gemini_vision(arquivo)   # fallback quando o Vostro está fora
```

- Trigger do fallback: timeout, 5xx, ou conexão recusada no nó docling.
- `markitdown` roda no VPS (EasyPanel) → não precisa do mesmo fallback,
  mas se a entrada for scan ele falha em qualidade: nesse caso o roteamento
  já deve ter mandado para docling (com fallback Gemini).

## Credenciais
- Header: `X-MCP-Key` — valor em `~/.hermes/secrets/docling_mcp_key` (Vostro).
- Nunca colar a key em texto/commit. Usar credential do n8n.
