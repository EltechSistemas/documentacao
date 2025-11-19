# SCF_CaixaFinanceiro.md

## 📖 Descrição
Sistema de relatório de Caixa Financeiro que apresenta o fluxo de movimentações financeiras por conta corrente, com cálculo de saldos e suporte a diferentes formas de visualização.

## 🎯 Finalidade
Gerar relatório consolidado das movimentações financeiras, permitindo acompanhamento do saldo por conta corrente, com opções de visualização por natureza financeira e exportação em diferentes formatos.

## 👥 Público-Alvo
- Departamento Financeiro
- Contabilidade
- Controladoria
- Gestores

## ⚙️ Configuração
**Recursos Necessários:**
- Relatório `SCF_CaixaFinanceiro` - Versão PDF
- Relatório `SCF_LancamentosXLS` - Versão Excel

**Localização:** `eltech/relatorios/scf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAB10` - Lançamentos financeiros
- `DAB01` - Contas correntes
- `DAB1002` - Rateio por conta corrente
- `DAB1001` - Departamentos
- `DAB10011` - Naturezas financeiras
- `DAB0101` - Saldos mensais
- `ABF10` - Naturezas

**Entidades Envolvidas:**
- `Dab10` - Lançamento financeiro
- `Dab01` - Conta corrente
- `Dab1002` - Rateio financeiro
- `Abf10` - Natureza financeira

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| contaCorrente | List<Long> | Não | Filtro por contas correntes específicas |
| periodo | LocalDate[] | Sim | Período de análise (data inicial e final) |
| imprimir | Integer | Sim | Formato de saída (0=XLSX, 1=PDF) |
| isSaltarPagina | Boolean | Não | Controla quebra de página por conta no PDF |
| isNatureza | Boolean | Não | Agrupa por natureza financeira quando true |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define parâmetros padrão (mês corrente)
   - Configura título e informações da empresa
   - Prepara formatação do período

2. **Busca de Dados**
   - Consulta lançamentos financeiros no período
   - Aplica filtros por conta corrente
   - Opção de agrupamento por natureza

3. **Cálculo de Saldos**
   - Busca saldo inicial do período
   - Calcula entradas e saídas acumuladas
   - Atualiza saldo a cada movimentação

4. **Processamento por Conta**
   - Agrupa movimentações por conta corrente
   - Calcula saldo individual por conta
   - Mantém sequência por código de conta

5. **Geração de Saída**
   - **PDF:** Relatório formatado com quebras de página
   - **XLSX:** Planilha exportável para análise

## ⚠️ Regras de Negócio

### Cálculo de Saldos
- **Saldo Inicial:** Último saldo conhecido antes do período
- **Entradas:** Movimentações com dab10mov = 0
- **Saídas:** Movimentações com dab10mov = 1
- **Saldo Atual:** Saldo anterior + entradas - saídas

### Agrupamento de Dados
- **Por Natureza:** Quando isNatureza = true, usa valores de Dab10011
- **Sem Natureza:** Quando isNatureza = false, usa valores de Dab1002
- **Ordenação:** Por código da conta e data do lançamento

### Validações de Período
- Período é obrigatório
- Data final não pode ser anterior à data inicial
- Suporte a períodos que cruzam anos diferentes

### Formatação de Saída
- **PDF:** Layout otimizado para impressão, com quebras opcionais
- **XLSX:** Estrutura tabular para análise em planilhas
- Ambos incluem cabeçalho com empresa e período

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| Relatório PDF | Caixa Financeiro formatado | DadosParaDownload |
| Planilha XLSX | Dados em formato Excel | DadosParaDownload |

## 🔧 Dependências

**Bibliotecas:**
- `JasperReports` - Geração de relatórios PDF
- `Apache POI` - Exportação para Excel
- `multiorm` - Persistência e consultas

**Serviços:**
- `DateUtils` - Manipulação de datas e períodos

## 📝 Observações Técnicas

- **Performance:** Consultas SQL otimizadas com joins
- **Escalabilidade:** Suporte a grandes volumes de lançamentos
- **Flexibilidade:** Múltiplas formas de agrupamento e filtro

### Cálculo de Saldo Inicial
- Busca recursiva por saldos anteriores quando necessário
- Considera movimentações do início do mês até o dia anterior
- Suporte a contas sem histórico prévio

### Tratamento de Dados
- Normalização de valores nulos para zero
- Formatação de datas no padrão brasileiro
- Ordenação consistente por conta e data
- Cálculo incremental de saldos

### Personalização de Relatório
- Título dinâmico com nome da empresa
- Período formatado no cabeçalho
- Opção de quebra de página por conta
- Suporte a diferentes níveis de detalhe