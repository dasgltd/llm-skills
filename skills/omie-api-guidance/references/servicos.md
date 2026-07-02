# Limitações: Omie Serviços API

Este documento consolida as limitações, restrições arquiteturais e o que funciona (ou não) nos endpoints da família de **Serviços** da Omie (`/servicos/os/` e afins), utilizados na nossa integração MCP e em projetos como o OmieAPI e Validador.

## 1. ListarOS (`/servicos/os/` - Listar Ordens de Serviço)

**Propósito:** Listar as Ordens de Serviço (OS) de faturamento de serviços.
- **Protocolo Suportado:** Somente aceita método POST via JSON (ou SOAP). Rejeita sumariamente GET. Tentar bater via GET retorna erro 405 ou Timeout.
- **Filtros e Parâmetros:**
  - A paginação (`pagina`, `registros_por_pagina`) é idêntica à de finanças. Max 500 registros.
  - A API possui um filtro nativo chamado `etapa`. Exemplo: `etapa: "10"` (que significa faturada) ou `etapa: "20"` (encerrada).
  - É possível usar o campo `filtrar_por_data_de` (que na OS se refere, por padrão, à data de emissão/previsão, dependendo de como a base está configurada). No entanto, não confie plenamente no filtro de datas complexas. 
  - Projetos: Diferente de finanças, a OS costuma vir fortemente atrelada ao `codigo_cliente` e ao `codigo_projeto`. Mas filtros dinâmicos de projeto via payload podem falhar e resultar em `Nenhum registro encontrado` se a tipagem estiver incorreta. Sempre valide a pesquisa enviando o ID em vez do Nome.
- **Limitação de Campos Retornados:** 
  - A OS no Omie é dividida em Abas (Cabeçalho, Departamento, Serviço Prestado). A listagem básica pode não retornar todas as tags personalizadas, dependendo do payload.
  - Campos não documentados oficialmente podem vir ocultos e causar `undefined` nas propriedades do Typescript. Trate tudo com `Optional (?)`.

## 2. IncluirOS (`/servicos/os/` - Criar Ordens de Serviço)

**Propósito:** Emitir ou criar rascunho de uma OS.
- **Exigências Estritas (O que faz crachar com 500):**
  - O código do cliente (`codigo_cliente`) deve existir previamente. Não se pode passar texto.
  - O Código do Serviço Municipal (`codigo_servico_municipal` ou código interno de integração `codigo_servico`) tem que bater milimetricamente com o cadastro na aba Serviços. Errar 1 dígito retorna *"Serviço não encontrado"*.
  - Rateio de Departamento e Projetos. A distribuição nas abas financeiras (se habilitadas na OS) precisa somar obrigatoriamente `100%`. Se somar `99.99%`, a API rejeita com erro de validação do banco (HTTP 500 / 400).
  - Valores quebrados: Valores (`valor_unitario`, `valor_total`) devem vir como tipo numérico flutuante (ex: `150.50`), **nunca em String com vírgula** `"150,50"`, pois o parse JSON da Omie quebra e dispara Internal Server Error.

## 3. Geral para Serviços

- **Rate Limits Compartilhados:** A chamada para o módulo `/servicos/` concorre no mesmo Rate Limit de 240/minuto que o `/financas/`.
- **Dica de Faturamento (Invoice):** A criação de uma OS não gera imediatamente uma NF-e ou NFS-e. É criada em estágio de proposta/aprovada. O faturamento efetivo precisa ser feito pelo faturamento da OS no próprio Omie ou chamando o endpoint subsequente de faturar a OS.
- **Filtros Inexistentes:** Filtro textual por descritivo da OS não existe nativamente. A inteligência artificial (IA) ou código MCP precisa fazer `.find` ou `.filter` no campo `cDescricao` da OS via software, percorrendo o cache recebido das chamadas da API.

---
*Para dúvidas e testes de integração, sempre rodar um POST cru usando cURL no Developer Portal do Omie antes de implementar o código, pois a mensagem de erro do servidor deles é a única fonte da verdade.*
