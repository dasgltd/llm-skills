# Limitações: Omie Produtos e Vendas API

Este documento consolida as limitações, restrições e o que funciona (ou não) nos endpoints da família de **Produtos, Pedidos de Venda e Estoque** da Omie (`/produtos/`), essenciais para não esbarrar em erros misteriosos.

## 1. Pedidos de Venda (`/produtos/pedidovenda/`)

**Propósito:** Listar e incluir pedidos de venda de produtos físicos.
- **Filtros e Paginação:**
  - A paginação (`pagina`, `registros_por_pagina`) segue a regra de ouro: **máximo de 100 a 500 registros por página** dependendo da volumetria dos itens.
  - O filtro `apenas_importado_api` é bastante útil se quiser listar apenas o que o MCP criou.
  - Filtros como `numeroPedidoExterno` e `tipoDeEntrega` existem, mas o uso massivo para varrer banco de dados sem passar intervalo de datas gera timeout.
- **Inclusão de Pedido (IncluirPedidoVenda):**
  - Códigos obrigatórios: Precisa informar `codigo_cliente_omie`, `codigo_produto`, `codigo_local_estoque`. Se algum deles estiver inativo ou incorreto, retorna Erro 500 ou 400.
  - Ao invés de usar nomes, use sempre as IDs mapeadas via Cache no MCP.
  - Diferente do serviço, pedidos de produto exigem formatação contábil rígida (NCM, CFOP). Faltar um imposto retorna erro caso o cliente não tenha parametrização tributária padrão na Omie.
  
## 2. Produtos e Posição de Estoque (`/geral/produtos/` e `/estoque/`)

**Propósito:** Listar produtos e verificar saldo no estoque.
- **ListarProdutos:**
  - Muitos produtos = Payload gigantesco. É vital utilizar **Listagens Incrementais**. Passe `filtrar_apenas_omiept` ou use os parâmetros de data de alteração/inclusão `filtrar_por_data_de` e `filtrar_por_data_ate` para evitar estourar os limites da Omie.
- **Estoque (Posição e Ajuste):**
  - Ajustes de Estoque (`/estoque/ajuste/`) não aceitam requisições simultâneas. Tentar disparar 5 `IncluirAjuste` ao mesmo tempo gera *deadlocks* ou Timeout no banco deles.
  - Para obter saldo (`get_stock_position`), prefira buscar itens que tiveram movimentação e sempre passe a `dDataPosicao` no formato `DD/MM/YYYY`.

## 3. Geral para Produtos

- **Concorrência (Limitação Crítica):** 
  - A documentação oficial avisa: **Não há suporte para requisições simultâneas em operações de inclusão ou alteração de dados.** Envie pedidos de venda em fila (um por um, sequencial), e não via `Promise.all()`.
- **Cache Local:** 
  - Consultar o mesmo ID duas vezes em menos de 60 segundos fará a API ignorar ou retornar do cache interno da Cloudflare deles. Confie no seu cache.
