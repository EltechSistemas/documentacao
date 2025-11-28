# SCE - Saldo Por Local - LCB

## 📖 Descrição
Relatório que exibe o saldo de itens por local, com opções de filtragem por classes, status, lotes, séries e validade. Permite gerar PDF ou XLSX e também listar locais sem saldo.

## 🎯 Finalidade
Fornecer controle detalhado de estoque por local, permitindo análise de disponibilidade, vencimentos e classificação de itens, auxiliando gestão de inventário e auditoria.

## 👥 Público-Alvo
- Controle de Estoque
- Departamento Fiscal
- Auditoria Interna
- Gestão de Armazéns
- Diretoria Administrativa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-----------------|
| itens | List<Long> | Não | IDs dos itens a incluir | Lista de IDs |
| classes | List<Long> | Não | IDs das classes de itens | Lista de IDs |
| status | List<Long> | Não | Status dos itens | Lista de IDs |
| local | List<String> | Não | Nomes de locais | Lista de Strings, suportando `%` |
| lote | String | Não | Lotes específicos separados por `;` | Ex: `L001;L002` |
| serie | String | Não | Séries específicas separadas por `;` | Ex: `S01;S02` |
| validade | LocalDate[] | Não | Intervalo de validade | `[dataInicial, dataFinal]` |
| saldoItem | Boolean | Não | Indica se o saldo do item deve ser exibido | true/false |
| exibirSaldo | Boolean | Não | Indica se os saldos devem ser exibidos | true/false |
| localSemSaldo | Boolean | Não | Lista locais que não possuem saldo | true/false |
| imprimir | Integer | Sim | Tipo de saída | 0=PDF, 1=XLSX |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abm15nome | Nome do local | String |
| abm01codigo | Código do item | String |
| abm01descr | Descrição do item | String |
| bcc0201lote | Lote do item | String |
| bcc0201serie | Série do item | String |
| bcc0201qt | Quantidade em estoque | BigDecimal |
| bcc0201validade | Data de validade | LocalDate |
| aam06codigo | Unidade de medida do item | String |
| fcua | Quantidade em unidade auxiliar | String |

## 🔄 Fluxo do Processo
1. Carrega parâmetros do usuário e valores padrão (`imprimir=0`).
2. Obtém empresa ativa e adiciona parâmetros globais (`EMPRESA`, `SALDOITEM`, `EXIBIRSALDO`).
3. Decide se deve buscar locais sem saldo (`localSemSaldo`) ou dados completos de saldo:
    - **Locais com saldo:** filtra por itens, classes, status, local, lotes, séries e validade.
    - **Locais sem saldo:** filtra por locais que não possuem registros de saldo.
4. Monta consultas SQL dinâmicas com `WHERE` condicional conforme filtros.
5. Executa a query e retorna lista de `TableMap` com os resultados.
6. Gera saída final:
    - `imprimir=0` → PDF via `gerarPDF("SCE_SaldoPorLocal", dados)`
    - `imprimir=1` → XLSX via `gerarXLSX("SCE_SaldoPorLocalXLSX", dados)`

## ⚠️ Regras de Negócio
- Filtros nulos ou vazios são ignorados na query.
- Para locais sem saldo, retorna apenas locais sem registros em `bcc02`.
- O campo `fcua` calcula quantidade em unidade auxiliar, evitando divisão por zero (`nullif`).
- Ordenação padrão: `abm15nome`, `abm01codigo`, `bcc0201lote`.
- O relatório suporta múltiplos lotes e séries, separados por `;`.

## 🎨 Saídas Disponíveis
| Formato | Descrição |
|---------|-----------|
| PDF | Relatório completo, pronto para impressão |
| XLSX | Planilha Excel com os mesmos dados do PDF |