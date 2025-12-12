# SCV_PedidoEstoqueProducao.md

## 📖 Descrição
Relatório "SCV - Pedido Venda x Estoque x Produção - Eltech" que consolida informações de pedidos de venda, estoque atual e produção em andamento para análise de atendimento e planejamento.

## 🎯 Finalidade
Fornecer uma visão integrada da situação de itens entre pedidos em aberto, saldo em estoque e produção programada, permitindo análise de capacidade de atendimento e planejamento de produção.

## 👥 Público-Alvo
- Departamento de Vendas
- Planejamento de Produção
- Controle de Estoque
- Gestão Comercial

## 📁 Localização do Código
**Caminho:** `eltech/relatorios/scv/SCV_PedidoEstoqueProducao.groovy`

## 🧱 Estrutura do Projeto
- **Classe:** `SCV_PedidoEstoqueProducao`
- **Pacote:** `eltech.relatorios.scv`
- **Herança:** `RelatorioBase` (Framework SAMDev)
- **Linguagem:** Groovy

## ⚙️ Configuração
**Recursos Necessários:**
- Framework SAMDev
- Módulo de relatórios Streama
- Conexão com banco de dados corporativo
- Bibliotecas: `DateUtils`, `Utils`, `TableMap`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `eaa01` - Documentos SCV
- `abb01` - Cabeçalho de documentos
- `abe01` - Entidades
- `abm01` - Itens/Cadastro de produtos
- `eaa0103` - Itens de documentos SCV
- `eaa01032` - Vinculação entre itens SCV e SRF
- `bcc02` - Saldo de estoque
- `bab01` - Ordens de produção
- `abp20` - Componentes de produção

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| exportar | Integer | Sim | 0=PDF, 1=XLSX |
| tipo | List<Long> | Não | Tipo(s) de documento |
| status | List<Long> | Não | Status de estoque |
| emissao | LocalDate[] | Sim | Período de emissão [início, fim] |
| entrega | LocalDate[] | Não | Período de entrega [início, fim] |
| itemIni | String | Não | Item inicial (código) |
| itemFim | String | Não | Item final (código) |
| atendimentoN | Boolean | Sim | Incluir não atendidos |
| atendimentoP | Boolean | Sim | Incluir parcialmente atendidos |
| atendimentoE | Boolean | Sim | Incluir entregues |
| mpms | List<Integer> | Não | Tipos de MPM |
| entIni | String | Não | Entidade inicial |
| entFim | String | Não | Entidade final |
| aprovado1 | Boolean | Não | Aprovados |
| aprovado2 | Boolean | Não | Não aprovados |
| bloqueado1 | Boolean | Não | Bloqueados |
| bloqueado2 | Boolean | Não | Não bloqueados |

## 📋 Saídas do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abm01codigo | Código do item | String |
| abm01na | Descrição do item | String |
| aam06codigo | Unidade de medida | String |
| abm01tipo | Tipo do item | Integer |
| qtdePedido | Quantidade pedida | BigDecimal |
| qtdeEntregue | Quantidade entregue | BigDecimal |
| qtdeEntregar | Quantidade a entregar | BigDecimal |
| qtdeEstoque | Saldo em estoque | BigDecimal |
| qtdeProducao | Quantidade em produção | BigDecimal |
| saldo | Saldo disponível (estoque + produção - entregar) | BigDecimal |

## 🔄 Fluxo de Execução

1. **Validação e Preparação de Parâmetros**
   - Coleta e converte parâmetros de entrada
   - Processa flags booleanas para listas (aprovado, bloqueado)
   - Define valores padrão para filtros

2. **Busca de Itens Base**
   - Executa `buscarItens()` para obter lista de itens filtrados
   - Aplica filtros: item, MPM, tipo, datas, entidade, aprovação, bloqueio

3. **Agregação de Quantidades**
   - Para cada item encontrado, busca:
     - `buscarQtdPedidos()`: Total pedido
     - `buscarQtdEntregue()`: Total já entregue
     - `buscarQtdAEntregar()`: Total pendente de entrega
     - `buscarSaldoAtualItem()`: Estoque atual
     - `buscarQtdeProducao()`: Em produção

4. **Cálculo de Saldo**
   - Para cada item: `saldo = (estoque - aEntregar) + produção`

5. **Geração de Saída**
   - Se `exportar = 0`: Gera relatório PDF
   - Se `exportar = 1`: Gera planilha XLSX

## ⚠️ Regras de Negócio

### Filtros de Atendimento
- **Não atendidos (N):** `eaa01scvAtend = 0`
- **Parcialmente atendidos (P):** `eaa01scvAtend = 1`
- **Entregues (E):** Considera apenas itens com vinculação via `eaa01032`

### Critérios de Inclusão
- Documentos não cancelados (`eaa01cancdata IS NULL`)
- Documentos SCV (`eaa01clasDoc = 0`)
- Movimentação de entrada (`eaa01esMov = 1`)
- Status de produção = 1 (em aberto)

### Cálculo de Pendências
Quantidade a entregar = Quantidade pedida - Quantidade já entregue
Considera apenas pendências onde `quantidade pedida >= quantidade entregue`

## 🔍 Métodos Principais

### `buscarItens()`
Retorna lista de itens filtrados com base nos parâmetros:
- Filtros de item (código)
- Filtros de MPM (tipo de item)
- Filtros de datas (emissão, entrega)
- Filtros de entidade
- Filtros de aprovação e bloqueio

### `buscarQtdPedidos()`
Calcula quantidade total pedida por item, considerando:
- Itens específicos
- Tipo de documento
- Período de emissão
- Período de entrega
- Status de atendimento

### `buscarQtdEntregue()`
Calcula quantidade já entregue por item através da tabela de vinculação `eaa01032`

### `buscarQtdAEntregar()`
Calcula quantidade pendente de entrega usando CTE (Common Table Expression):
```sql
WITH base AS (...)
SELECT abm01id, SUM(eaa0103qtcoml) AS total
FROM base
GROUP BY abm01id