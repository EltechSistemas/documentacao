## 📖 Descrição
Relatório de extrato contábil para documentos a receber e a pagar, com opções de visualização por diferentes critérios e exportação para múltiplos formatos.

## 🎯 Finalidade
Fornecer uma visão consolidada da situação financeira da empresa, permitindo análise de contas a receber/pagar por diversos filtros e períodos, com cálculo automático de juros, multas e encargos.

## 👥 Público-Alvo
- Departamento Financeiro
- Contabilidade
- Controladoria
- Gestores Financeiros

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA01` - Documentos a receber/pagar
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades (clientes/fornecedores)
- `ABC10` - Plano de contas contábil
- `ABF15` - Portadores
- `AAH01` - Tipos de documento
- `AAC10` - Empresas
- `DAB10` - Históricos de documentos

**Entidades Envolvidas:**
- `Daa01` - Documentos financeiros
- `Abe01` - Clientes/Fornecedores
- `Aac10` - Empresa
- `Abf15` - Portador
- `Abc10` - Conta contábil

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| classe | Integer | Sim | Tipo de documento | 0=A receber, 1=A pagar |
| ordem | Integer | Sim | Ordenação | 0=Código entidade, 1=Nome entidade |
| exportar | Integer | Sim | Formato saída | 0=PDF, 1=XLSX |
| tp | Integer | Sim | Tipo de registro | 0=Todos, 1=Reais |
| numeroInicial | Integer | Não | Número documento inicial | 000000000 |
| numeroFinal | Integer | Não | Número documento final | 999999999 |
| port | List<Long> | Não | Portadores específicos | IDs portadores |
| documento | List<Long> | Não | Tipos documento | IDs tipos documento |
| entInicial | String | Não | Código entidade inicial | Código inicial |
| entFinal | String | Não | Código entidade final | Código final |
| emps | List<Long> | Não | Empresas específicas | IDs empresas |
| dataPer | LocalDate[] | Sim | Período de análise | Data inicial e final |
| op | Integer | Sim | Layout do relatório | 0=Padrão, 1=Alternativo |
| opEnt | Integer | Sim | Nome entidade | 0=Nome reduzido, 1=Nome completo |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| ent_cod | Código da entidade | String |
| ent_nome | Nome da entidade | String |
| contabil | Conta contábil | String |
| dt_emis | Data emissão | Date |
| tp | Tipo documento | String |
| doc_num | Número documento | Integer |
| dt_vcto | Data vencimento | Date |
| dt_baixa | Data baixa | Date |
| portador | Nome portador | String |
| aRealizar | Valor a realizar | BigDecimal |
| realizado | Valor realizado | BigDecimal |
| jme | Juros/Multas/Encargos | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Define classe (receber/pagar)
   - Configura ordenação e formatação
   - Processa filtros de empresas e entidades

2. **Construção da Consulta**
   - Monta SQL dinâmico com filtros aplicados
   - Define joins específicos por classe
   - Aplica ordenação conforme parâmetro

3. **Busca de Dados**
   - Consulta documentos no período
   - Calcula campos financeiros (JME)
   - Agrupa por entidade e documento

4. **Ajuste de Período**
   - Separa valores realizados e a realizar
   - Filtra documentos fora do período
   - Calcula totais de juros, multas e encargos

5. **Geração de Saída**
   - Seleciona template conforme layout
   - Exporta para PDF ou XLSX
   - Formata dados para visualização

## ⚠️ Regras de Negócio

### Filtros de Período
- Considera documentos com data de liquidação dentro do período
- Inclui documentos em aberto com vencimento no período
- Exclui documentos emitidos após o período final

### Cálculos Financeiros
- **JME:** Soma de juros, multas e encargos
- **A Realizar:** Valores em aberto no período
- **Realizado:** Valores liquidados no período
- Descontos zerados para valores realizados

### Classificação
- **Classe 0 (Receber):** Join com `Abe02` e `Abe0201`
- **Classe 1 (Pagar):** Join com `Abe03` e `Abe0301`
- Conta contábil da primeira sequência

## 🎨 Saídas Disponíveis

| Formato | Descrição | Template | Método |
|---------|-----------|----------|---------|
| PDF | Layout padrão | `SCF_ExtratoContabil` | `gerarPDF()` |
| PDF | Layout alternativo | `SCF_ExtratoContabil_R1` | `gerarPDF()` |
| XLSX | Planilha padrão | `SCF_ExtratoContabil` | `gerarXLSX()` |
| XLSX | Planilha alternativo | `SCF_ExtratoContabil_R1` | `gerarXLSX()` |

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários e datas
- `jasperreports` - Geração de relatórios (PDF)

**Configurações:**
- Templates Jasper para diferentes layouts
- Parâmetros JSON para campos financeiros

## 📝 Observações Técnicas

- SQL dinâmico com múltiplos filtros opcionais
- Join específico por classe (receber/pagar)
- Processamento de campos JSON para valores financeiros
- Ajuste automático de datas e períodos
- Suporte a múltiplas empresas no cabeçalho
- Ordenação flexível por código ou nome de entidade