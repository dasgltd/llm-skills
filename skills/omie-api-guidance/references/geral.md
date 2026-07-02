# Limitações: Omie Geral e Cadastros API

Este documento consolida as melhores práticas para a família de endpoints **Geral** da Omie (`/geral/`), que trata do coração do ERP: Clientes, Projetos, Departamentos, Vendedores, Categorias, etc.

## 1. Cadastros Estáticos vs Dinâmicos (Regra do N+1)

A maior causa de gargalo em integrações MCP com a Omie é ignorar a natureza desses cadastros.
- **Categorias (`/geral/categorias/`)**, **Departamentos (`/geral/departamentos/`)** e **Contas Correntes (`/geral/contacorrente/`)** são cadastros *Estáticos*.
  - **Limitação:** Não mude de página a página sempre que for listar uma conta a pagar para perguntar a categoria dela.
  - **Ação:** Sempre faça a paginação completa no *Boot* da aplicação e guarde em `Map()`.
- **Clientes e Fornecedores (`/geral/clientes/`)** são cadastros *Dinâmicos* gigantes.
  - **Limitação:** Algumas bases têm 50.000 clientes. Salvar tudo na RAM no boot derruba a aplicação via `Heap Out of Memory`.
  - **Ação:** Nunca carregue a tabela de clientes no boot. Use o endpoint sob demanda passando o `codigo_cliente_omie` quando precisar resolver apenas aquele nome no relatório final do LLM (ou traga o Resumido, se estritamente necessário).
  - Use listagens incrementais (`filtrar_apenas_inclusao`, `filtrar_por_data_de`) ao sincronizar clientes para o BI ou Datalake.

## 2. Inclusão de Clientes e Fornecedores

- **Duplicidade e CPF/CNPJ:** O ERP Omie tem travas rígidas de validação de CPF e CNPJ. Se tentar incluir um cliente (`IncluirCliente`) com um CPF inválido, com formato numérico errado, ou de um cliente já existente, ele lançará Erro `500` (Erro de Validação) e rejeitará o bloco inteiro.
- **CNAE e Regimes Tributários:** Assim como produtos, o cadastro de clientes afeta diretamente os impostos. Recomenda-se criar clientes na Omie com o máximo de tags corretas, preenchendo as tags do Simples Nacional ou Lucro Presumido para evitar gargalos na NF.

## 3. Rate Limits e Bloqueio Definitivo

- O endpoint `/geral/clientes/` é um dos mais vigiados pela Cloudflare e pelos Gateways da Omie. Bater nesse endpoint em um loop `while` infinito (por esquecer de iterar a `pagina++`) fará a Omie travar seu IP com o infame HTTP 425 (bloqueio de 30 minutos).
- **Tratamento Seguro:**
  ```javascript
  let pg = 1, tot = 1;
  do {
    // try/catch dentro do loop com sleep em caso de HTTP 429
    tot = res?.total_de_paginas || 1;
    pg++;
  } while (pg <= tot);
  ```

## 4. O Problema das Strings vs Inteiros
- A Omie é construída em banco de dados relacional antigo (Firebird/Delphi no backend raiz).
- Para a Omie, `codigo_cliente_omie` (INT, ex: 1423412341) é totalmente diferente de `codigo_cliente_integracao` (STRING, ex: "CRM-CLI-403").
- Quando um endpoint exigir um ID nativo (`_omie`), passar `String` ou passar a `_integracao` fará a API dizer *"Cliente não encontrado"*.
- **Dica:** O MCP sempre deve expor a versão `_omie` nas tipagens Typescript e fazer a LLM utilizar apenas identificadores válidos.
