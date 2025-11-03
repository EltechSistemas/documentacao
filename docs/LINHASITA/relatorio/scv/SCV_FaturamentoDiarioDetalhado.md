# SCV_FaturamentoDiarioDetalhado.md

## 📖 Descrição
Sistema de geração de relatório de faturamento diário detalhado para a Linhasita, com consolidação de dados por filial e tipo de operação.

## 🎯 Finalidade
Fornecer uma visão detalhada do faturamento diário da empresa, permitindo o acompanhamento do processo de faturamento por filiais e tipos de operação comercial.

## 👥 Público-Alvo
- Departamento Financeiro
- Faturamento
- Controladoria
- Gestão Comercial

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SCV_FaturamentoDiarioDetalhado` - Layout do relatório

**Localização:** `linhasita/relatorios/scv/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `AAC10` - Empresas/Filiais
- `ABB01` - Cabeçalho de documentos
- `ABB10` - Operações comerciais
- `ABA20` - Repositório de dados (dias não úteis)

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Aac10` - Empresas do grupo
- `Abb01` - Cabeçalho documentos
- `Abb10` - Operações comerciais

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| periodo | String | Sim | Período de análise (MM/aaaa) | Mês/ano atual |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| dtMes | Data do mês | LocalDate |
| flagDiaUtil | Indicador de dia útil | Integer |
| vlr_Exp | Valor exportação | BigDecimal |
| kg_Exp | Peso exportação | BigDecimal |
| vlr_Ref | Valor refugo/sucata | BigDecimal |
| kg_Ref | Peso refugo/sucata | BigDecimal |
| vlr_Ind | Valor industrialização | BigDecimal |
| kg_Ind | Peso industrialização | BigDecimal |
| vlr_Fat | Valor faturamento normal | BigDecimal |
| kg_Fat | Peso faturamento normal | BigDecimal |
| vlr_Birigui | Valor Birigui | BigDecimal |
| kg_Birigui | Peso Birigui | BigDecimal |
| vlr_Franca | Valor Franca | BigDecimal |
| kg_Franca | Peso Franca | BigDecimal |
| vlr_Hamburgo | Valor Hamburgo | BigDecimal |
| kg_Hamburgo | Peso Hamburgo | BigDecimal |
| vlr_Serrana | Valor Serrana | BigDecimal |
| kg_Serrana | Peso Serrana | BigDecimal |
| vlr_Total | Valor total matriz | BigDecimal |
| kg_Total | Peso total matriz | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Processa período informado (MM/aaaa)
   - Define primeiro e último dia do mês

2. **Busca de Dados**
   - Consulta documentos fiscais do período
   - Obtém devoluções do mês
   - Busca dias não úteis do repositório

3. **Processamento de Dias**
   - Cria lista com todos os dias do mês
   - Classifica dias (útil, sábado, domingo, não útil)
   - Consolida valores por tipo de operação e filial

4. **Cálculo de Totais**
   - Soma valores e pesos por categoria
   - Calcula totais gerais por filial
   - Registra devoluções do período

5. **Geração de Saída**
   - Gera PDF com dados consolidados
   - Apresenta totais por filial e matriz

## ⚠️ Regras de Negócio

### Validações
- Considera apenas documentos fiscais ativos (`eaa01cancData IS NULL`)
- Apenas documentos com status NFe = 3 (autorizada)
- Filtra por tipos de documento específicos (NF-e própria)
- Considera apenas operações com `resumo_venda = 1`

### Classificação de Operações
- **EXPORTACAO:** Códigos 331927255, 285603711
- **SUCATA:** Códigos 331931980, 331931983
- **INDUSTRIALIZACAO:** Código 285367776
- **NORMAL:** Demais operações

### Classificação de Dias
- **Flag 1:** Dias não úteis (repositório)
- **Flag 2:** Sábados
- **Flag 3:** Domingos
- **Flag null:** Dias úteis normais

### Filiais Incluídas
- LINHASITA MATRIZ
- LINHASITA BIRIGUI
- LINHASITA FRANCA
- LINHASITA HAMBURGO
- LINHASITA SERRANA

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Relatório formatado | `gerarPDF()` |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multitec.utils` - Utilitários e cálculos
- `java.time` - Manipulação de datas

**Serviços:**
- Acesso ao banco de dados

## 📝 Observações Técnicas

- Implementa consulta otimizada para grandes volumes de dados
- Suporte a múltiplas filiais do grupo Linhasita
- Classificação automática de dias úteis e não úteis
- Consolidação por tipo de operação comercial
- Cálculo de totais parciais e gerais
- Integração com repositório de dias não úteis
- Formatação específica para relatório financeiro