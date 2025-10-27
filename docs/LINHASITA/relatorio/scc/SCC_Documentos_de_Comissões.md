## 📖 Descrição
Relatório de comissões para representantes com versões analítica e sintética, permitindo acompanhamento detalhado ou consolidado dos valores a pagar.

## 🎯 Finalidade
Controlar e analisar as comissões de representantes, oferecendo visão detalhada por documento ou visão consolidada por representante, com flexibilidade de filtros por datas e status de cálculo.

## 👥 Público-Alvo
- Departamento Comercial
- Controladoria
- Gestão de Representantes
- Departamento Financeiro

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DCB01` - Base de cálculo de comissões
- `ABB01` - Documentos de comissões
- `ABE01` - Representantes e entidades
- `AAH01` - Tipos de documento
- `DAA01` - Documentos a receber
- `DCD01` - Cálculos de comissões

**Entidades Envolvidas:**
- `Dcb01` - Cálculo de comissões
- `Abe01` - Representantes e clientes
- `Abb01` - Documentos
- `Aac10` - Empresa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| layout | Integer | Sim | Tipo de relatório | 0=Analítico, 1=Sintético |
| dataCalculo | Integer | Sim | Status do cálculo | 0=Não calculado, 1=Calculado, 2=Todos |
| numeroInicial | Integer | Não | Número documento inicial | ≥ 0 |
| numeroFinal | Integer | Não | Número documento final | ≥ 0 |
| representantes | List<Long> | Não | Representantes específicos | IDs representantes |
| emissao | LocalDate[] | Não | Período de emissão | Data inicial e final |
| dataCredito | LocalDate[] | Não | Período de crédito | Data inicial e final |
| imprimir | Integer | Sim | Formato de saída | 0=PDF, 1=XLSX |

## 📋 Campos do Relatório

### Layout Analítico
| Campo | Descrição | Tipo |
|-------|-----------|------|
| repCod | Código representante | String |
| repNome | Nome representante | String |
| abb01num | Número documento | Integer |
| abb01parcela | Parcela documento | String |
| entCod | Código cliente | String |
| entNa | Nome reduzido cliente | String |
| abb01data | Data emissão | Date |
| dcb01dtCredito | Data crédito | Date |
| abb01valor | Valor documento | BigDecimal |
| dcb01bc | Base cálculo comissão | BigDecimal |
| dcb01taxa | Taxa comissão | BigDecimal |
| dcb01valor | Valor comissão | BigDecimal |

### Layout Sintético
| Campo | Descrição | Tipo |
|-------|-----------|------|
| repCod | Código representante | String |
| repNome | Nome representante | String |
| abb01valor | Soma valores documentos | BigDecimal |
| dcb01bc | Soma base cálculo | BigDecimal |
| dcb01valor | Soma valores comissão | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Define layout (analítico/sintético)
   - Processa filtros de datas (emissão/crédito)
   - Valida status de cálculo

2. **Seleção de Consulta**
   - Analítico: Detalhamento por documento
   - Sintético: Consolidação por representante
   - Aplica filtros comuns a ambos

3. **Construção da Consulta**
   - Monta SQL com joins específicos
   - Aplica filtros dinâmicos
   - Define ordenação conforme layout

4. **Processamento de Dados**
   - Busca dados do banco
   - Calcula totais (sintético)
   - Formata informações

5. **Geração de Saída**
   - Seleciona template conforme layout
   - Exporta para PDF ou XLSX
   - Formata cabeçalho com período

## ⚠️ Regras de Negócio

### Filtros de Data
- **Emissão:** Data do documento original
- **Crédito:** Data de crédito da comissão
- Período exibido no cabeçalho conforme filtro aplicado

### Status de Cálculo
- **Não calculado:** `centralCalc.abb01data is null`
- **Calculado:** `centralCalc.abb01data is not null`
- **Todos:** Sem filtro de cálculo

### Validações
- Apenas endereços principais (`abe0101principal = 1`)
- Relacionamento com documentos a receber (`abb0101tabela = 'Daa01'`)
- Aplica filtros padrão do sistema (`obterWherePadrao`)

## 🎨 Saídas Disponíveis

| Layout | Formato | Template | Método |
|--------|---------|----------|---------|
| Analítico | PDF | `SCC_DocumentosComissoes_Analitico` | `gerarPDF()` |
| Analítico | XLSX | `SCC_DocumentosComissoes_Analitico` | `gerarXLSX()` |
| Sintético | PDF | `SCC_DocumentosComissoes_Sintetico` | `gerarPDF()` |
| Sintético | XLSX | `SCC_DocumentosComissoes_Sintetico` | `gerarXLSX()` |

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários e datas
- `jasperreports` - Geração de relatórios

**Configurações:**
- Templates Jasper para layouts analítico e sintético
- Filtros padrão do sistema para entidade `DCB01`

## 📝 Observações Técnicas

- SQL dinâmico com múltiplos joins para relacionamentos
- Dois templates distintos para analítico/sintético
- Flexibilidade de filtros por datas diferentes
- Ordenação específica por layout
- Suporte a múltiplos representantes
- Controle de status de cálculo das comissões