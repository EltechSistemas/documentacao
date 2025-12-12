# SCF - Lançamentos (El Tech)

## 📖 Descrição
Relatório de lançamentos financeiros por conta corrente, apresentando o fluxo completo com cálculo de saldo acumulado, ideal para conciliação bancária e acompanhamento diário do caixa.

## 🎯 Finalidade
Controlar e analisar todos os lançamentos financeiros por conta corrente, permitindo conciliação, identificação de movimentações e acompanhamento do saldo em tempo real.

## 👥 Público-Alvo
- Departamento Financeiro
- Contabilidade
- Tesouraria
- Auditoria Interna

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Dab10` - Lançamentos financeiros
- `Dab1002` - Vinculação de lançamentos com contas correntes
- `Dab01` - Contas correntes
- `Dab0101` - Saldos mensais das contas correntes
- `Aac10` - Empresas
- `Aac01` - Grupos de empresas
- `Abb01` - Central de documentos

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| contaCorrente | List<Long> | Não | Filtro por IDs das contas correntes |
| periodo | LocalDate[] | Sim | Período do relatório [início, fim] |
| imprimir | Integer | Sim | Formato de saída (0=XLSX, 1=PDF) |
| isSaltarPagina | Boolean | Sim | Controla quebra de página por conta no PDF |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Carregamento dos filtros padrão (mês atual)
- Configuração da empresa ativa
- Definição dos parâmetros do relatório

### 2. **Processamento dos Filtros**
- Validação do período informado
- Aplicação de filtros de contas correntes
- Configuração da quebra de página

### 3. **Busca e Processamento de Dados**
- Busca dos lançamentos financeiros no período
- Cálculo do saldo inicial para cada conta
- Processamento sequencial para cálculo de saldo acumulado
- Ordenação por conta e data

### 4. **Cálculo de Saldos**
- **Saldo inicial**: Busca saldo do mês anterior ou último disponível
- **Saldo acumulado**: Calculado linha a linha considerando entradas e saídas
- **Saldo final**: Resultado após processamento de todos os lançamentos

### 5. **Geração do Relatório**
- Formato XLSX ou PDF conforme seleção
- Quebra de página por conta (quando habilitado)
- Inclusão de cabeçalho com empresa e período

## ⚠️ Regras de Negócio

### Cálculo de Saldo Inicial
1. Busca saldo do mês anterior na tabela `Dab0101`
2. Se não encontrado, retrocede meses/anos até encontrar saldo
3. Adiciona entradas e subtrai saídas ocorridas antes do período
4. Resultado é o saldo inicial para o período informado

### Processamento de Lançamentos
- **Entradas** (`dab10mov = 0`): Aumentam o saldo
- **Saídas** (`dab10mov = 1`): Diminuem o saldo
- **Ordenação**: Por código da conta e data do lançamento
- **Saldo inicial**: Calculado apenas na primeira linha de cada conta

### Formato de Saída
- **PDF**: Ideal para impressão e visualização, com opção de quebra por conta
- **XLSX**: Ideal para análise e manipulação dos dados
- **Quebra de página**: Aplicada apenas no PDF quando habilitada

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra a geração do relatório.

### `buscarSaldoInicial()`
Calcula o saldo inicial para uma conta em uma data específica.

### `somarValorEntrada()`
Soma as entradas de uma conta antes da data informada.

### `somarValorSaida()`
Soma as saídas de uma conta antes da data informada.

### `buscarValorInicialDoMes()`
Busca o saldo inicial registrado em meses anteriores.

### `obterDadosRelatorio()`
Busca dados de lançamentos com filtros aplicados.

## 📊 Estrutura de Saída

**Campos do Relatório:**
- `codigoConta` - Código da conta corrente
- `dab01nome` - Nome da conta corrente
- `dab10data` - Data do lançamento
- `dab10historico` - Histórico do lançamento
- `dab10mov` - Tipo de movimento (0=Entrada, 1=Saída)
- `dab10valor` - Valor do lançamento
- `SALDOINICIAL` - Saldo inicial da conta
- `SALDO` - Saldo acumulado após o lançamento

**Parâmetros do Relatório:**
- `TITULO_RELATORIO`: "Lançamentos"
- `EMPRESA`: Nome da empresa ativa
- `PERIODO`: Período formatado do relatório

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários (DateUtils, Utils, TableMap)
- `sam.server.samdev.relatorio` - Base para relatórios
- `sam.server.samdev.utils` - Utilitários do SAM (Parametro)
- `java.time` - Manipulação de datas
- `sam.core.variaveis` - Variáveis do sistema (MDate)

**Módulo:** SCF (Sistema Contábil Financeiro)

## 📝 Observações Técnicas

### Performance
- Consultas otimizadas com filtros aplicados no banco
- Cálculo de saldo inicial com busca progressiva
- Processamento em lote para grandes volumes

### Tratamento de Períodos
- Período padrão: mês atual
- Suporte a qualquer intervalo de datas
- Formatação específica para exibição

### Cálculo de Saldo Inicial
- Busca recursiva por saldos anteriores
- Tratamento de mudança de ano (janeiro busca dezembro do ano anterior)
- Soma de movimentações parciais antes do período

### Segurança
- Aplicação de where padrão do sistema para controle de acesso
- Validação de parâmetros de entrada
- Tratamento de valores nulos e vazios

### Comparação com Caixa Financeiro
- **Este relatório**: Foco em lançamentos individuais com detalhes completos
- **Caixa Financeiro**: Opção de agrupamento por natureza
- **Ambos**: Cálculo de saldo acumulado e conciliação

---

**Última Alteração:** 04/12/2025 às 08:30  
**Autor:** Bruno  
**Tipo:** Relatório de Lançamentos Financeiros  
**Versão:** 1.0 (Customizado para El Tech)