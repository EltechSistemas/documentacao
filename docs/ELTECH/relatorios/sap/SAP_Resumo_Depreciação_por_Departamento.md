## 📖 Descrição
Relatório que apresenta o resumo da depreciação patrimonial agrupado por departamentos/classificações patrimoniais da empresa El Tech.

## 🎯 Finalidade
Controlar e analisar a depreciação de ativos imobilizados por centro de custo/departamento, permitindo acompanhar:
- Saldo anterior de depreciação
- Depreciação do período atual
- Baixas de ativos
- Transferências entre departamentos
- Saldo atual acumulado

## 👥 Público-Alvo
- Departamento Financeiro
- Contabilidade
- Gestores Patrimoniais
- Controladoria

## 📊 Dados e Fontes
**Tabelas Principais:**
- `ABB11` - Departamentos/Centro de Custos
- `ECB01` - Imobilizados
- `ECB0101` - Reclassificações de Imobilizados
- `ECB0102` - Depreciação Mensal
- `ABB20` - Bens Patrimoniais
- `ABA01` - Parâmetros do Sistema

**Entidades Envolvidas:**
- `Abb11` - Departamento
- `Ecb01` - Imobilizado
- `Ecb0101` - Reclassificação
- `Ecb0102` - Depreciação
- `Aba01` - Parâmetro

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| dtReferencia | String | Sim | Mês/Ano de referência | Formato MM/yyyy |
| imprimir | Integer | Sim | Tipo de saída | 0=PDF, 1=XLSX |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo | Fórmula |
|-------|-----------|------|---------|
| Código | Código do departamento | String | - |
| Nome | Nome do departamento | String | - |
| Saldo Anterior | Saldo depreciado acumulado anterior | BigDecimal | `buscaSaldo(dataRef-1)` |
| Depreciação | Depreciação do mês atual | BigDecimal | `buscaDepreciacao(dataRef)` |
| Baixas | Baixas de ativos no período | BigDecimal | `buscaBaixas(dataRef)` |
| Saldo Atual | Saldo depreciado acumulado atual | BigDecimal | `buscaSaldo(dataRef)` |
| Transferências | Variação por transferências | BigDecimal | `saldoAtual - saldoAnterior - deprec + baixas` |

## 🔄 Fluxo do Processo

1. **Validação de Estrutura**
   - Busca parâmetro `ABB11-ESTRCODDEPTO` para estrutura de códigos
   - Valida existência de pelo menos 2 grupos na estrutura

2. **Processamento de Reclassificações**
   - Identifica última reclassificação por imobilizado até a data de referência
   - Mapeia imobilizados por departamento considerando reclassificações

3. **Cálculo de Valores**
   - Busca departamentos ativos
   - Calcula saldos anteriores e atuais
   - Calcula depreciação do período
   - Identifica baixas de ativos
   - Calcula transferências entre departamentos

4. **Consolidação Hierárquica**
   - Soma valores dos níveis inferiores para superiores baseado na estrutura de códigos
   - Filtra registros com valores zerados

## ⚠️ Regras de Negócio

### Validações
- Estrutura de códigos deve ter pelo menos 2 grupos
- Data de referência obrigatória
- Considera apenas imobilizados ativos (não baixados)

### Cálculos
- **Saldo Anterior:** Soma depreciação acumulada até mês anterior
- **Depreciação:** Valor depreciado no mês atual
- **Baixas:** Depreciação de ativos baixados no período
- **Transferências:** Diferença entre variação do saldo e movimentações conhecidas

### Filtros
- Aplica `getSamWhere()` para filtros padrão do sistema
- Considera apenas departamentos com movimentação no período

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Relatório formatado para visualização | `gerarPDF()` |
| XLSX | Planilha para análise e exportação | `gerarXLSX()` |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e queries
- `multitec.utils` - Utilitários (DateUtils, StringUtils, Utils)
- `sam.core` - Core do sistema SAM
- `sam.model` - Entidades do sistema

**Parâmetros do Sistema:**
- `ABB11-ESTRCODDEPTO` - Estrutura de codificação de departamentos

## 📝 Observações Técnicas

- Utiliza recursividade para processar estrutura hierárquica
- Considera reclassificações históricas de imobilizados
- Otimizado com consultas SQL nativas para performance
- Filtra automaticamente registros com todos valores zerados
