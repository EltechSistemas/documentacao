# SPP - Ordens de Produção - Linhasita

## 📖 Descrição
Relatório de ordens de produção da Linhasita, detalhando processos de acabamento, pesagem e tingimento. Permite diferentes layouts (designs) conforme necessidade, incluindo tingimento único e ajustado, com sub-relatórios de processos específicos.

## 🎯 Finalidade
Fornecer visão completa das ordens de produção, incluindo itens, componentes, quantidades, unidades e processos aplicados, permitindo controle operacional e acompanhamento de produção.

## 👥 Público-Alvo
- Produção
- Planejamento de Produção
- Qualidade
- Controle de Estoque
- Auditoria Interna

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-----------------|
| numeroInicial | Integer | Não | Número inicial da ordem de produção | Inteiro ≥ 0 |
| numeroFinal | Integer | Não | Número final da ordem de produção | Inteiro ≥ 0 |
| numAjustada1..4 | Integer | Não | Números de ordens ajustadas manualmente | Inteiros |
| tipoDoc | List<Long> | Não | Tipos de documentos a considerar | Lista de IDs |
| desingRelatorio | Integer | Sim | Layout do relatório | 0=Acabamento, 1=Romaneio Pesagem, 2=Tinturaria Única, 3=Tinturaria Ajustada |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abb01num | Número da ordem de produção | Integer |
| abb01data | Data da ordem | Date |
| prodNa | Nome do produto | String |
| niquelina | Niquelina do produto | Integer |
| turbo | Código turbo do produto | Integer |
| kg / bab01qt | Quantidade produzida | BigDecimal |
| compDescr | Nome do componente | String |
| ITENS | Itens detalhados do processo | String |
| PROCESSO | Código do processo | String |
| NOMEPROCESSO | Nome do processo | String |
| UNIDADE | Unidade de medida | String |

## 🔄 Fluxo do Processo
1. Carrega parâmetros de filtro e números de ordem.
2. Inicializa listas de ordens ajustadas e sub-relatórios.
3. Seleciona layout (`desingRelatorio`) e busca dados correspondentes:
    - 0: Acabamento
    - 1: Romaneio de Pesagem
    - 2: Tinturaria Única (com sub-relatório de processos)
    - 3: Tinturaria Ajustada (com itens detalhados)
4. Monta `TableMap` ou `TableMapDataSource` para os dados principais e sub-relatórios.
5. Adiciona parâmetros necessários para sub-relatórios.
6. Gera PDF final usando `gerarPDF(nomeRelatorio, dados)`.

## ⚠️ Regras de Negócio
- Layouts distintos determinam quais dados e sub-relatórios serão carregados.
- Processos de tinturaria são mapeados conforme tabela interna `codProc`.
- Itens de tingimento só são incluídos se correspondem a processos válidos.
- Ordens ajustadas são filtradas manualmente pelo usuário.
- Totalização e detalhamento seguem lógica específica de produção.

## 🎨 Saídas Disponíveis
| Formato | Descrição |
|---------|-----------|
| PDF | Relatório pronto para impressão, com sub-relatórios quando aplicável |