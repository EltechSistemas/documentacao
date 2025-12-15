# SRF_CalculoItem_Entrada_Importacao.md
# SRF - Cálculo de Item para Entrada de Importação

## 📖 Descrição
Fórmula responsável por calcular os valores tributários, quantidades e totais de itens em documentos fiscais de entrada por importação, aplicando regras específicas de importação, conversões monetárias e cálculos de impostos.

## 🎯 Finalidade
- Calcular valores de itens em documentos de importação
- Aplicar regras tributárias específicas para importação
- Converter valores em moeda estrangeira para Real
- Calcular impostos federais e estaduais (II, IPI, ICMS, PIS, COFINS)
- Determinar CFOP apropriado para importação
- Calcular custos de aquisição e valores financeiros

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Compras/Importação
- Faturamento

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa0103` - Itens do documento fiscal
- `Eaa01` - Documento fiscal
- `Aac10` - Empresa
- `Abe01` - Entidade/Cliente
- `Abm01` - Item
- `Abm0101` - Configuração do item por empresa
- `Abm12` - Configuração fiscal do item
- `Abm13` - Configuração comercial do item
- `Abm1301` - Fatores de conversão
- `Aaj15` - CFOP
- `Abg01` - NCM
- `Abm10` - Valores do item
- `Abm1001` - Valores do item por UF
- `Aaj10` - CST ICMS
- `Aaj11` - CST IPI
- `Aaj12` - CST PIS
- `Aaj13` - CST COFINS
- `Abd01` - Tipo de documento (PCD)
- `Abb01` - Central de documento
- `Aag02` - UF
- `Aag0201` - Município

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento fiscal |

## 🔄 Fluxo do Processo

### 1. **Inicialização e Validações**
- Carregamento do item do documento (Eaa0103)
- Validação de parâmetros obrigatórios
- Interrupção se item não for fornecido

### 2. **Carregamento de Entidades Relacionadas**
- Documento fiscal (Eaa01)
- Central de documento (Abb01)
- Tipo de documento (Abd01)
- Entidade/Cliente (Abe01)
- Endereço principal da entidade
- Empresa ativa (Aac10)
- Item (Abm01) e suas configurações
- Dados fiscais e comerciais do item
- CFOP, NCM e CSTs

### 3. **Cálculos Específicos por Tipo de Documento**
- **PCD 114**: Cálculo simples
- **PCD 314**: Cálculo com conversão monetária e impostos de importação

### 4. **Cálculos de Impostos**
- Imposto de Importação (II)
- IPI (Imposto sobre Produtos Industrializados)
- PIS (Programa de Integração Social)
- COFINS (Contribuição para o Financiamento da Seguridade Social)
- ICMS (Imposto sobre Circulação de Mercadorias e Serviços)

### 5. **Cálculos de Quantidades e Valores**
- Conversão de quantidades comerciais para estoque
- Cálculo de pesos líquido e bruto
- Aplicação de descontos
- Cálculo de totais do item e documento

### 6. **Atualização do Item**
- Armazenamento de cálculos em campos livres (JSON)
- Atualização dos totais do item
- Retorno do item processado

## ⚠️ Regras de Negócio

### **Regras de CFOP para Importação**
- Se tipo do item = 8, CFOP é fixado como "3351"
- Caso contrário, mantém o CFOP informado

### **Cálculo para PCD 114**
- Total = Quantidade × Valor Unitário
- FOB = Total - Seguro

### **Cálculo para PCD 314 (Importação)**
1. **Conversão Monetária:**
   - Busca cotação da moeda na data informada
   - Se não informada data, usa última cotação disponível
   - Converte FOB, Frete e Seguro para Real

2. **Base de Cálculo:**
   - Total = FOB + Frete (convertido) + Seguro (convertido) + Acréscimos

3. **Imposto de Importação (II):**
   - Aplicado apenas se alíquota > 0
   - Base = Total do item
   - Valor = Base × Alíquota

4. **Outras Despesas:**
   - Inclui PIS, COFINS, SISCOMEX, AFRMM e Multas

### **Regras de IPI**
- Base = Total do item + II
- Alíquota obtida do NCM
- Se alíquota = 0, valor é movido para "IPI Isento"

### **Regras de PIS/COFINS para Exportação**
- Se UF da entidade = "EX", usa alíquotas específicas da UF
- Caso contrário, usa alíquotas do cadastro do item

### **Regras de ICMS**
- Alíquota obtida da configuração da UF
- Redução de base aplicável
- CST terminado em "51": ICMS e alíquota zerados

### **Custos e Valores Finais**
- Custo de aquisição = Total do documento
- Valor unitário sem impostos = Total / Quantidade
- CIF = Total - II

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de cálculo do item.

**Fluxo:**
1. Carrega e valida o item do documento
2. Carrega todas as entidades relacionadas
3. Executa `calcularItem()` para processamento
4. Atualiza e retorna o item processado

### `calcularItem()`
Executa todos os cálculos tributários e valores do item.

**Principais Processos:**
- Ajuste de CFOP para importação
- Cálculos específicos por PCD
- Cálculo de impostos (II, IPI, PIS, COFINS, ICMS)
- Conversões de quantidade
- Cálculo de pesos
- Aplicação de descontos
- Cálculo de totais

## 📊 Estrutura de Campos Livres (JSON)

### **Campos no JSON do Item (Eaa0103)**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| ii_aliq | BigDecimal | Alíquota do Imposto de Importação |
| imp_bc | BigDecimal | Base de cálculo do II |
| ii_ii | BigDecimal | Valor do Imposto de Importação |
| vlr_frete_real | BigDecimal | Frete convertido para Real |
| vlr_seguro_real | BigDecimal | Seguro convertido para Real |
| vlr_vlme | BigDecimal | Volume calculado |
| vlr_pl | BigDecimal | Peso líquido |
| vlr_pb | BigDecimal | Peso bruto |
| tx_desc_incond | BigDecimal | Taxa de desconto incondicional |
| vlr_desc | BigDecimal | Valor do desconto |
| ipi_bc | BigDecimal | Base de cálculo do IPI |
| ipi_aliq | BigDecimal | Alíquota do IPI |
| ipi_ipi | BigDecimal | Valor do IPI |
| ipi_isento | BigDecimal | Valor isento de IPI |
| pis_aliq | BigDecimal | Alíquota do PIS |
| pis_bc | BigDecimal | Base de cálculo do PIS |
| pis_pis | BigDecimal | Valor do PIS |
| cofins_aliq | BigDecimal | Alíquota do COFINS |
| cofins_bc | BigDecimal | Base de cálculo do COFINS |
| cofins_cofins | BigDecimal | Valor do COFINS |
| vlr_outras | BigDecimal | Outras despesas (PIS+COFINS+SISCOMEX+AFRMM+Multa) |
| icm_aliq | BigDecimal | Alíquota do ICMS |
| icm_reduc_bc | BigDecimal | Percentual de redução da base do ICMS |
| icm_bc | BigDecimal | Base de cálculo do ICMS |
| icm_icm | BigDecimal | Valor do ICMS |
| icm_isento | BigDecimal | Valor isento de ICMS |
| custo_aquisicao | BigDecimal | Custo de aquisição |
| vlr_unit_sem | BigDecimal | Valor unitário sem impostos |
| qtde_doc | BigDecimal | Quantidade no documento |
| cif_imp | BigDecimal | Valor CIF da importação |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - ValidacaoException e TableMap
- `sam.dicdados` - FormulaTipo
- `sam.model` - Entidades do sistema
- `sam.server.samdev.formula` - FormulaBase
- `sam.server.samdev.utils` - Parametro

**Entidades do Sistema:**
- Módulo AA: Aac10, Aag02, Aag0201, Aaj10-15
- Módulo AB: Abb01, Abd01, Abe01, Abg01, Abm01-13, Abm0101, Abm1001, Abm1301
- Módulo EA: Eaa01, Eaa0101-0103
