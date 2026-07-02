# Limitações: Omie CRM API

Este documento consolida as limitações da API do módulo **CRM** da Omie (`/crm/`), com foco no gerenciamento de Oportunidades, Funil e Contatos.

## 1. Oportunidades (`/crm/oportunidades/`)

**Propósito:** Gerenciar os "cards" do funil de vendas.
- **Limitações de Edição e Criação (Rate Limit Severo):**
  - Diferente do painel da Omie onde o usuário pode editar em lote até 500 registros, a API **não permite** criação ou alteração simultânea (paralela) de muitas oportunidades em menos de 1 segundo. Se utilizar Node, evite `Promise.all()`. Envie as edições usando `for...of` com um pequeno delay.
  - Limite recomendado: **4 requisições por segundo** para consultas e **1 por segundo** para edição/inclusão.
- **Filtros e Listagem Incrementais:**
  - Em listagem em lote (`ListarOportunidades`), use no **máximo 500 registros por página**.
  - O volume de oportunidades ao longo dos anos fica colossal. Se a aplicação MCP pedir todas as oportunidades, limite via software (e peça ao usuário filtros de Funil).
  - Use parâmetros de "data e hora da última atualização" (`data_alteracao` ou equivalentes) para puxar somente as Oportunidades movidas de etapa hoje.
- **Metadados (Tags e Campos Customizados):**
  - Oportunidades possuem "características" ou campos personalizados. Nem todos vêm no payload enxuto por padrão.

## 2. Contatos e Funis (`/crm/contatos/` e `/crm/funil/`)

- Funis mudam muito raramente. Deve-se fazer cache das Etapas e dos Funis (`ListarFunil`) durante o *boot* do MCP para mapear o `codigo_etapa` para nomes legíveis (ex: "Em Negociação").
- Quando for retornar os dados para o LLM, aplique a mesma técnica de Agrupamento ("Agrupar por Etapa do Funil") para que a Inteligência consiga ver o Pipeline completo sem precisar varrer milhares de objetos JSON gastando tokens.
