# SRF - Cálculo de Item de Entrada por Importação

## 📖 Descrição
Fórmula para cálculo de impostos e valores em itens de documentos fiscais de entrada por importação, aplicando regras específicas de importação, tributação federal e estadual, conversões de unidades e tratamento de moedas estrangeiras.

## 🎯 Finalidade
Calcular automaticamente os valores fiscais e financeiros de itens em documentos de importação, incluindo impostos de importação (II), IPI, ICMS, PIS/COFINS, conversões monetárias e ajustes de base de cálculo.

## 👥 Público-Alvo
- Departamento Fiscal
- Importação/Exportação
- Contabilidade
- Faturamento

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa0103` - Itens do documento fiscal
- `Eaa01` - Documentos fiscais
- `Eaa0102` - Dados gerais do documento
- `Eaa0101` - Endereços do documento
- `Abb01` - Central de documento
- `Abb10` - Operações comerciais
- `Abd01` - Parâmetros de cálculo de documentos (PCD)
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Produtos
- `Abm0101` - Configurações do produto por empresa
- `Abm12` - Dados fiscais do item
- `Abm13` - Dados comerciais do item
- `Abm1301` - Fatores de conversão de unidade
- `Abm10` - Valores do produto
- `Abm1001` - Valores do produto por UF
- `Aaj15` - CFOP (Código Fiscal de Operações)
- `Abg01` - NCM (Nomenclatura Comum do Mercosul)
- `Aaj10` - CST ICMS
- `Aaj11` - CST IPI
- `Aaj12` - CST PIS
- `Aaj13` - CST COFINS
- `Aac10` - Empresas
- `Aag02` - Estados (UF)
- `Aag0201` - Municípios

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento fiscal a ser calculado |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação do item do documento (Eaa0103)
- Carregamento do documento fiscal (Eaa01)
- Validação de pessoa física como contribuinte de ICMS
- Carregamento da operação comercial (Abb10) e PCD (Abd01)
- Obtenção de dados da entidade (Abe01) e empresa (Aac10)

### 2. **Carregamento de Dados Geográficos**
- Identificação do endereço principal da entidade
- Obtenção de município e UF da entidade
- Obtenção de município e UF da empresa
- Determinação se operação é intraestadual ou interestadual

### 3. **Carregamento de Dados do Produto**
- Configurações fiscais (Abm12) e comerciais (Abm13) do item
- Fatores de conversão de unidades (Abm1301)
- NCM (Abg01) e CFOP (Aaj15)
- Valores do produto por UF (Abm1001)
- Códigos de situação tributária (CSTs)

### 4. **Cálculo de CFOP**
- Determinação automática de CFOP baseado em:
  - Tipo de operação comercial
  - Localização geográfica (dentro/fora do estado)
  - Tipo de inscrição da entidade (CNPJ/CPF)
  - Presença de IVA no item
  - Tipo de produto (revenda/produção)

### 5. **Cálculo de Valores do Item**
- **PCD 114**: Cálculo simples (quantidade × valor unitário)
- **PCD 314**: Cálculo complexo de importação:
  - Conversão monetária (FOB para Real)
  - Soma de fretes, seguros e acréscimos
  - Cálculo de CIF (Custo, Seguro e Frete)

### 6. **Cálculo de Impostos**
- **II (Imposto de Importação)**: Base CIF × Alíquota
- **IPI**: Base (CIF + II) × Alíquota NCM
- **PIS/COFINS**: Base valor total × Alíquotas
- **ICMS**: Cálculo complexo com base em fórmula específica
- Tratamento de isenções e alíquotas zero

### 7. **Ajustes Finais**
- Conversão de quantidades para unidades de estoque
- Cálculo de pesos brutos e líquidos
- Aplicação de descontos incondicionais
- Cálculo de valor total do documento
- Definição de custo de aquisição

## ⚠️ Regras de Negócio

### Validações Críticas
- Pessoa física não pode ser contribuinte de ICMS
- Item deve ter configuração fiscal cadastrada
- CFOP deve existir no cadastro após determinação automática
- Cotação monetária obrigatória para importação em moeda estrangeira

### Cálculo de CFOP
- Dígito inicial fixo em "3" para importação
- CFOPs específicos para operações com IVA (401)
- Diferencição entre venda (102/108) e revenda (101/107)
- CFOP 109 para operações específicas

### Cálculo de Importação (PCD 314)
- Conversão obrigatória de moeda estrangeira
- CIF = FOB + Frete + Seguro + Acréscimos
- Base de cálculo do ICMS inclui todos os impostos
- Valor total do documento igual à base de cálculo do ICMS

### Tratamento de Impostos
- **II**: Zerado quando alíquota for zero
- **IPI**: Base inclui CIF + II, isenção move valor para campo específico
- **PIS/COFINS**: Alíquotas diferenciadas para operações com exterior
- **ICMS**: Cálculo por fórmula ((CIF+impostos)/(1-aliq))×aliq
- **CST 51**: Zera ICMS e alíquota automaticamente

### Conversões e Ajustes
- Conversão de quantidade comercial para quantidade de uso
- Cálculo de volume baseado em fator do produto
- Desconto incondicional aplicado sobre valor total
- Arredondamentos em 2 casas decimais para valores monetários

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o cálculo do item de importação.

### `calcularItem()**
Executa todos os cálculos fiscais e financeiros do item, incluindo:
- Determinação de CFOP
- Cálculo de valores totais
- Cálculo de todos os impostos
- Conversões de unidades e moedas
- Ajustes finais do item

## 📊 Estrutura de Cálculo

**Dados de Entrada:**
- Item do documento (Eaa0103) com quantidade e valor unitário
- Documento fiscal (Eaa01) com PCD e dados de importação
- Dados geográficos de entidade e empresa
- Configurações do produto e NCM

**Cálculos Intermediários:**
- Determinação de CFOP
- Conversão monetária (para importação)
- Cálculo de CIF
- Bases de cálculo de cada imposto

**Resultados:**
- Valores calculados de cada imposto (II, IPI, ICMS, PIS, COFINS)
- Valores totais do item (comercial, documento, financeiro)
- Bases de cálculo ajustadas
- Campos JSON com detalhes do cálculo

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários e exceções
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.math` - Operações com BigDecimal

**Módulo:** SRF (Sistema de Faturamento)

## 📝 Observações Técnicas

### Estrutura de JSON
- `jsonEaa0103`: Campos calculados do item (impostos, bases, valores)
- `jsonAbm0101`: Configurações do produto por empresa
- `jsonAbm1001_UF_Item`: Valores do produto por UF
- `jsonAag02Ent`: Configurações da UF da entidade
- `jsonAbe01`: Dados adicionais da entidade

### Tratamento de Moedas
- Cotação obtida por data específica ou mais recente
- Validação de cotação existente
- Conversão de valores FOB para Real
- Cálculo de valores em moeda nacional

### Validações de Negócio
- Interrupção por exceção em erros críticos
- Validações preventivas em dados obrigatórios
- Mensagens claras para correção pelo usuário

### Performance
- Carregamento otimizado de entidades relacionadas
- Uso de Criteria para consultas eficientes
- Minimização de consultas ao banco de dados

---

**Última Alteração:** 27/11/2025 às 15:00  
**Autor:** Bruno  
**Tipo:** Fórmula de Cálculo de Item de Importação  
**Versão:** 1.0