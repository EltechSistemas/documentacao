# SCF – Pagamentos – Strema

## 📖 Descrição
Relatório que apresenta os pagamentos registrados no módulo financeiro (SCF), detalhando documentos, parcelas, entidades, valores, datas de emissão, vencimento e baixa, com possibilidade de filtros por número de documento, entidade e períodos.

## 🎯 Finalidade
Permitir o controle e a análise dos pagamentos financeiros da empresa, possibilitando:
- Acompanhamento de documentos financeiros e parcelas
- Análise de valores pagos
- Consulta por período de emissão ou vencimento
- Identificação de entidades, bancos e centros de custo relacionados
- Suporte à conciliação financeira e auditoria

## 👥 Público-Alvo
- Departamento Financeiro
- Contabilidade
- Controladoria
- Auditoria

## 📊 Dados e Fontes

### Tabelas Principais
- DAA01 – Títulos Financeiros / Pagamentos
- ABB01 – Documento Financeiro (Central)
- ABE01 – Entidades
- ABF01 – Bancos
- DAB10 – Lançamentos Financeiros
- DAB1002 – Rateio de Centro de Custo
- DAB01 – Centro de Custo

### Entidades Envolvidas
- Aac10 – Empresa
- Daa01 – Pagamento
- Abb01 – Documento Financeiro
- Abe01 – Entidade
- Abf01 – Banco
- Dab10 – Lançamento Financeiro
- Dab01 – Centro de Custo

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|----------|------|-------------|-----------|-------------------|
| numeroInicial | Integer | Não | Número inicial do documento financeiro | Numérico |
| numeroFinal | Integer | Não | Número final do documento financeiro | Numérico |
| entidadeInicial | String | Não | Código inicial da entidade | Código alfanumérico |
| entidadeFinal | String | Não | Código final da entidade | Código alfanumérico |
| emissao | Intervalo | Não | Período de emissão do documento | Data inicial/final |
| vencimento | Intervalo | Não | Período de vencimento do título | Data inicial/final |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo | Origem / Regra |
|------|-----------|------|----------------|
| Entidade | Nome da entidade | String | ABE01.abe01nome |
| Documento | Número do documento financeiro | Integer | ABB01.abb01num |
| Emissão | Data de emissão do documento | Date | ABB01.abb01data |
| Parcela | Número da parcela | Integer | ABB01.abb01parcela |
| Valor | Valor do pagamento | BigDecimal | DAA01.daa01valor |
| Vencimento | Data de vencimento do título | Date | DAA01.daa01dtVctoN |
| Baixa | Data de baixa do pagamento | Date | DAA01.daa01dtBaixa |
| Banco | Nome do banco | String | ABF01.abf01nome |
| Centro de Custo | Nome do centro de custo | String | DAB01.dab01nome |

## 🔄 Fluxo do Processo

### Validação de Parâmetros
- Inicializa período padrão de emissão com o mês atual
- Valida parâmetros informados (número, entidade e períodos)

### Definição de Cabeçalho
- Obtém empresa ativa (AAC10)
- Define razão social da empresa no relatório
- Define período exibido com base em emissão ou vencimento

### Processamento dos Dados
- Aplica filtros dinâmicos conforme parâmetros informados
- Considera apenas títulos financeiros ativos (`daa01rp = 1`)
- Executa consulta SQL com joins entre documentos, entidades, bancos e centros de custo

### Ordenação
- Ordena por número do documento e parcela

## ⚠️ Regras de Negócio

### Validações
- Considera apenas pagamentos ativos
- Período de emissão tem prioridade sobre vencimento para exibição
- Parâmetros são opcionais e aplicados dinamicamente

### Cálculos
- Não realiza cálculos financeiros derivados
- Valores são apresentados conforme registrados no título

### Filtros
- Aplica `obterWherePadrao()` para filtros padrão do sistema
- Filtra por número de documento, entidade, emissão e vencimento quando informados

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|--------|-----------|--------|
| PDF | Relatório formatado para visualização | gerarPDF() |

## 🔧 Dependências

### Bibliotecas
- multitec.utils – Utilitários (DateUtils, Utils)
- sam.server.samdev.relatorio – Base de relatórios
- sam.server.samdev.utils – Parâmetros
- sam.model – Entidades do sistema

### Parâmetros do Sistema
- Filtros padrão aplicados via `obterWherePadrao()`

## 📝 Observações Técnicas
- Utiliza SQL nativo para melhor performance
- Filtros são montados dinamicamente conforme parâmetros informados
- Período exibido no cabeçalho é definido automaticamente
- Estrutura preparada para grandes volumes de dados
- Relatório disponível apenas no formato PDF
