# Limitações: Omie Finanças API

Este documento consolida TODAS as limitações, quirks (comportamentos peculares) e filtros suportados ou não suportados para os endpoints do módulo de **Finanças** da Omie que utilizamos (`/financas/contapagar/`, `/financas/contareceber/`, `/financas/resumo/`, `/geral/contacorrente/`).

## 1. ListarContasPagar (`/financas/contapagar/`) e ListarContasReceber (`/financas/contareceber/`)

Ambos compartilham a maioria das características estruturais.
**Propósito:** Retorna o espelho financeiro da empresa (contas a pagar ou a receber).

### Limitações de Filtros de Data
- **O que NÃO é suportado nativamente:** Você **NÃO PODE** filtrar por **Data de Vencimento** nativamente.
- **O que É suportado nativamente:** Os parâmetros `filtrar_por_data_de` e `filtrar_por_data_ate` referem-se APENAS à **Data de Registro/Inclusão** no sistema. 
- **Workaround Obrigatório:** Para buscar títulos que vencem em um determinado mês, você deve:
  1. Trazer todas as contas em aberto (ou buscar por um range amplo de data de registro).
  2. Fazer o `.filter()` na propriedade `data_vencimento` (ex: `15/10/2026`) via software no seu MCP.

### Limitações de Filtros de Entidade (Projetos e Vendedores)
- **O que NÃO é suportado:** Não há como passar `codigo_projeto` ou `codigo_vendedor` no corpo (payload) do `ListarContasPagar` ou `ListarContasReceber` para o Omie filtrar lá no banco de dados deles. A documentação oficial não prevê esses filtros na requisição raiz.
- **Como a informação vem:** 
  - O código do projeto vem em `codigo_projeto` (ou dentro do array `distribuicao` se for rateado).
  - O código do vendedor vem em `id_vendedor` ou `codigo_vendedor`.
- **Workaround:** Filtragem por Projeto ou Vendedor deve ocorrer estritamente **via software** (Javascript/Typescript) *após* a busca dos registros paginados.

### Filtros Nativos que FUNCIONAM bem
- `filtrar_por_status`: Aceita valores como `EMABERTO`, `PAGO`, `ATRASADO`, `LIQUIDADO`. Note que a Omie usa `EMABERTO` e não `ABERTO` para contas a pagar, mas para contas a receber geralmente `ABERTO` funciona. No nosso MCP, nós sempre forçamos o mapeamento para evitar erros de tipagem.
- `filtrar_por_data_de` / `filtrar_por_data_ate` (Apenas Registro, lembre-se).

### Valor liquidado (baixa) nos itens do Listar — QUIRK confirmado em produção (2026-07)
- O item de `ListarContasReceber` **NÃO traz `valor_baixado` na raiz** (sai sempre 0). A informação de baixa vem no bloco aninhado `resumo` do item (`nValPago`, `nValAberto`, `cLiquidado`...), e/ou só é inferível pelo `status_titulo` (`RECEBIDO`/`LIQUIDADO`/`PAGO`).
- Mesmo padrão vale para `ListarContasPagar` (`valor_pago` na raiz não é confiável).
- **Cálculo robusto (usar sempre, em cascata):** 1) `item.resumo.nValPago` se existir → 2) campo raiz (`valor_baixado`/`valor_pago`) se existir → 3) fallback por status: título com status de baixa total = `valor_documento`; senão 0.

## 2. PagarContaPagar e IncluirContaPagar (`/financas/contapagar/`)

- **Inclusão (`IncluirContaPagar`):**
  - O Omie é altamente rigoroso com a data. A `data_vencimento` e `data_previsao` devem ser mandadas no exato formato `DD/MM/YYYY`.
  - Códigos de categoria (Plano de Contas) devem existir previamente e não podem estar inativos. Além disso, remover o ponto final da categoria (`limparCodigoCategoriaLocal`) é vital para bater com o banco de dados.
- **Baixa/Pagamento (`LancarPagamento` ou `PagarContaPagar`):**
  - Requer o envio do `codigo_lancamento_omie` (ou `codigo_titulo`).
  - É obrigatório informar a `id_conta_corrente` (Conta Bancária) válida e ativa de onde o dinheiro saiu. Se enviar de conta inativa, retorna erro 500 ou 400 genérico.

## 3. ListarResumoFinancas (`/financas/resumo/`)

- **Finalidade:** Traz totalizadores (saldo do dia, recebimentos, pagamentos).
- **Limitação Principal:** Ele retorna um macro, não retorna quebra por centro de custo, projeto ou vendedor.
- **Comportamento da IA:** Geralmente a IA quer "Resumo Financeiro por Projeto". Como a API de Resumo NÃO aceita filtros por projeto, o nosso MCP precisa **ignorar a API de Resumo** e bater nas APIs `ListarContasPagar` + `ListarContasReceber`, aplicar o filtro do projeto em memória e criar um "Resumo Personalizado" via software (agrupando e somando os valores em um objeto).

## 4. ListarContasCorrentes (`/geral/contacorrente/`)

- **Limitação de Performance (Aviso N+1):**
  - Nunca chame essa API por demanda. Contas correntes mudam muito pouco.
  - O endpoint não possui bons filtros incrementais. Chamar ele a toda requisição de Conta a Pagar adiciona latência brutal.
- **Boas práticas:** Puxar a página 1 e 2 no boot do MCP (em cache `bankCache`). A resposta tem campos como `descricao` (ou `cDescricao` em outras APIs), `codigo_agencia`, `numero_conta_corrente` e `inativo`. Sempre usar contas `inativo !== 'S'`.
- **Tipo da conta:** `tipo`/`tipo_conta_corrente` (códigos: `CC` Conta Corrente, `CA` Aplicação, `CP` Poupança, `CR` Cartão, `PG` Pagamento, `CX` Caixa). O nome legível (`cDesTipo`) só vem no `ListarExtrato` (seção 6).
- **Fluxo de caixa (`nao_fluxo`):** `'S'` = conta NÃO considerada no fluxo de caixa (UI Omie: "Não Considerada"); `'N'` = considerada. Para saldos/fluxo, filtrar **`nao_fluxo != 'S'` além de `inativo != 'S'`** (ex.: "Sicredi - Provisão"/conta de pagamento é ATIVA mas fora do fluxo). Existe também `nao_resumo` (fora do Resumo) — campo distinto.
- **Data de abertura (`saldo_data`):** data do saldo inicial em `dd/MM/aaaa` = quando a conta passou a ter saldo = **abertura efetiva** no Omie (ex.: Banco do Brasil `01/05/2026`, Caixa `25/06/2026`, Sicredi `28/11/2025`). Usar como corte de mês no waterfall: **não incluir a conta em meses anteriores ao mês de `saldo_data`** (não mostrar 0,00 — omitir a linha). Campo pode vir vazio (conta sem saldo inicial) → não bloquear. `data_inc` é a criação do CADASTRO (não confundir com abertura).

## 5. ListarMovimentos (`/financas/mf/`) — Fluxo de caixa realizado

- **Dialeto próprio de paginação:** `nPagina`/`nRegPorPagina` (teto real 100/página) e `nTotPaginas` na resposta — NÃO usar o helper genérico (`pagina`/`registros_por_pagina`). Cada movimento já traz `detalhes` + `resumo` por padrão.
- **NÃO aceita `cDetalharTitulo`** (HTTP 500 "Tag [CDETALHARTITULO] não faz parte..."). Exige janela de data (`dDtPagtoDe`/`dDtPagtoAte`) senão devolve histórico gigante → 500/429; é sensível a 429, usar backoff generoso.
- **QUIRK RATEIO (confirmado ao vivo 2026-07):** título com categoria **rateada** é ACHATADO — o `mf/` devolve só `detalhes.cCodCateg` = a **1ª categoria do array de rateio** (ordem de cadastro, NÃO a de maior proporção) com o **valor cheio** do título. O array de rateio NÃO é exposto nesse endpoint. Para o rateio real (valor por categoria) só cruzando com `ListarContasPagar`/`Receber` (que trazem `categorias[]`).

## 6. ListarExtrato (`/financas/extrato/`) — Saldo de conta por data

- **Obrigatório `nCodCC`** (ou `cCodIntCC`): sem ele HTTP 500 "[nCodCC] ou [cCodIntCC] é obrigatório". É por conta — não existe "extrato de todas as contas" numa chamada.
- **Período** `dPeriodoInicial`/`dPeriodoFinal` (dd/MM/aaaa). Campos-chave da resposta: `nSaldoAnterior` (saldo no INÍCIO do período = fechamento do dia anterior) e `nSaldoAtual` (saldo no FIM do período = **saldo de fechamento**). Também traz `cDesTipo` (tipo legível da conta) e `listaMovimentos[]`.
- **Uso DASG (waterfall de saldos):** para saldo de fechamento mensal, período = 01→último dia do mês, ler `nSaldoAtual`; SALDO FINAL do mês = soma do `nSaldoAtual` das contas ativas. Alimenta o Saldo Inicial do Previsto x Realizado (fechamento do mês anterior ao início). Conta criada depois do período pedido volta faultstring benigno → tratar como saldo 0.

---

**Resumo de Sobrevivência Finanças:** 
A API financeira da Omie é projetada para sincronização massiva, não para *queries* precisas de relatório. O peso da inteligência (filtros cruzados, agrupamentos, ordenação) DEVE sempre ficar no código do nosso MCP, nunca tentamos delegar um filtro complexo para a Omie.
