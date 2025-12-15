# Documentação de Códigos Fonte - Sistema Empresarial

## Sumário

1. [CGS - Parcelamento (Equilibrio)](#1-cgs---parcelamento-equilibrio)
2. [SRF - Documento (Eltech)](#2-srf---documento-eltech)
3. [SRF - Cálculo de Item de Entrada por Importação](#3-srf---cálculo-de-item-de-entrada-por-importação)
4. [SCV - Pedidos por Item (Eltech)](#4-scv---pedidos-por-item-eltech)
5. [SCF - Boleto Banco do Brasil (Linhasita)](#5-scf---boleto-banco-do-brasil-linhasita)
6. [SCF - Demonstrativo de Resultados do Exercício (DRE) - LCB](#6-scf---demonstrativo-de-resultados-do-exercício-dre---lcb)
7. [CGS - Parcelamento (Eltech)](#7-cgs---parcelamento-eltech)
8. [SRF - Cálculo de Ficha de Conteúdo de Importação (FCI)](#8-srf---cálculo-de-ficha-de-conteúdo-de-importação-fci)

---

## 1. CGS - Parcelamento (Equilibrio)

### 📖 Descrição
Fórmula para cálculo e geração de parcelas de condições de pagamento, considerando datas de vencimento, ajustes por dias da semana, feriados, valores mínimos por parcela e configurações específicas do módulo Equilibrio.

### 🎯 Finalidade
Calcular automaticamente as parcelas de uma condição de pagamento, aplicando regras de vencimento, descontos, juros, multas e validações de valores mínimos por parcela.

### 👥 Público-Alvo
- Departamento Financeiro
- Faturamento
- Crédito e Cobrança

### 📊 Dados e Fontes
**Tabelas Principais:**
- `Abe30` - Condições de pagamento
- `Abe3001` - Parcelas da condição de pagamento
- `Abe3002` - Dias complementares (ajustes de vencimento)
- `Eaa01` - Documentos fiscais
- `Abb01` - Documentos da central (data de emissão)

### ⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtBase | LocalDate | Sim | Data base para cálculo das parcelas |
| abe30id | Long | Sim | ID da condição de pagamento |
| valor | BigDecimal | Sim | Valor total a ser parcelado |
| eaa01 | Eaa01 | Não | Documento fiscal relacionado |

### 🔄 Fluxo do Processo
#### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento da condição de pagamento (Abe30)
- Definição da data de emissão (do documento ou atual)

#### 2. **Cálculo dos Ajustes de Data**
- Aplicação de dias adicionais conforme dia da semana da data base
- Cálculo de vencimentos nominais para cada parcela
- Ajustes por vencimentos fixos e dias específicos
- Tratamento de meses com menos dias (ex: fevereiro)
- Aplicação de ajustes complementares (Abe3002)

#### 3. **Cálculo dos Valores das Parcelas**
- Distribuição percentual do valor total
- Cálculo do saldo para última parcela
- Aplicação de juros, multas e encargos
- Cálculo de descontos por antecipação

#### 4. **Validações e Regras Especiais**
- Verificação de valor mínimo por parcela
- Opção de agrupamento em parcela única
- Cálculo de comissões na última parcela
- Validação de documentos financeiros

### ⚠️ Regras de Negócio
#### Configuração de Vencimentos
- **Dias base**: Ajustes por dia da semana na data base (abe30diasDtBase1-7)
- **Vencimentos fixos**: Datas específicas por parcela (dt_vcto_fixo)
- **Dias do mês**: Vencimento em dia específico do mês (diavcto)
- **Ajustes complementares**: Regras por faixa de dias (Abe3002)
- **Mês referência**: Ajustes para mês corrente ou seguinte

#### Cálculo Financeiro
- **Juros**: Percentual aplicado sobre o valor da parcela
- **Multa**: Percentual ou valor fixo (vlr_fixo)
- **Desconto**: Percentual com limite temporal (dias_dtlimite)
- **Encargos**: Valores adicionais fixos

#### Validações de Parcelas
- **Valor mínimo**: Configurável por condição de pagamento (abe30vmpValor)
- **Opções**: 
  - 0 - Agrupar em uma parcela quando valor mínimo não for atingido
  - 1 - Validar valor mínimo e interromper se não atingido
- **Documentos financeiros**: 
  - Tipo 1 - Gera na data de vencimento
  - Tipo 2 - Gera na data de emissão

### 🔧 Métodos Principais
- `executar()` - Método principal que orquestra todo o processo de parcelamento
- `obterDiasAdicionaisAData()` - Calcula dias adicionais baseado no dia da semana
- `buscarCondicaoPagamentoPorId()` - Busca a condição de pagamento pelo ID
- `buscarParcelasPeloIdCondicaoPagamento()` - Busca as parcelas configuradas para a condição
- `buscarDiaComplementarPeloIdCondicaoPagamento()` - Busca ajustes complementares de dias
- `montarParcelaDto()` - Monta o DTO da parcela com todos os dados calculados

### 📊 Estrutura de Saída
**ParcelaDto:**
- `vctoN` - Data de vencimento nominal
- `valor` - Valor da parcela
- `criaDoc` - Tipo de documento financeiro (1 ou 2)
- `abf15id` - ID do portador
- `abf16id` - ID da operação
- `abf01id` - ID do banco
- `abf40id` - ID da forma de pagamento
- `cposLivres` - Campos livres (juros, multa, desconto, etc.)

**Lista de Parcelas:** Retornada no parâmetro `listaParcelas`

---

## 2. SRF - Documento (Eltech)

### 📖 Descrição
Fórmula para processamento e composição de documentos fiscais, aplicando regras de tributação, observações fiscais, cálculo de comissões, validações de itens e tratamento de documentos referenciados.

### 🎯 Finalidade
Processar documentos fiscais (notas fiscais) aplicando regras de negócio específicas, como validação de itens, cálculo de taxas de comissão, composição de observações e tratamento de documentos referenciados.

### 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Tesouraria
- Contabilidade

### 📊 Dados e Fontes
**Tabelas Principais:**
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe02` - Dados comerciais da entidade
- `Abe03` - Dados fiscais da entidade
- `Abe05` - Representantes
- `Abe40` - Tabelas de preço
- `Abb01` - Documentos centrais
- `Abb10` - Operações comerciais
- `Abd01` - Parâmetros de cálculo de documentos (PCD)
- `Abd02` - Campos fiscais do PCD
- `Abd05` - Campos industriais do PCD
- `Eaa01` - Documentos fiscais
- `Eaa0101` - Endereços do documento
- `Eaa0103` - Itens do documento fiscal
- `Eaa01033` - Itens de devolução referenciados
- `Abm01` - Produtos
- `Abm0101` - Configurações do produto por empresa
- `Abm10` - Valores do produto
- `Abm1001` - Valores do produto por UF
- `Aac10` - Empresas
- `Aac13` - Dados fiscais da empresa
- `Aag02` - Estados (UF)
- `Aag0201` - Municípios
- `Aaj10` - Códigos de situação tributária (CST)

### ⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| procInvoc | String | Sim | Processo de invocação da fórmula |

### 🔄 Fluxo do Processo
#### 1. **Configuração Inicial**
- Validação do documento fiscal (Eaa01)
- Carregamento da operação comercial (Abb10)
- Obtenção do PCD (Abd01)
- Carregamento da entidade (Abe01) e dados fiscais
- Carregamento da empresa (Aac10) e dados fiscais

#### 2. **Processamento por Item**
- Validação de itens sem código de produto
- Carregamento de configurações do produto (Abm0101)
- Obtenção de valores do produto por UF (Abm1001)
- Aplicação de observações fiscais específicas por item
- Tratamento de CST ICMS com redução de base de cálculo

#### 3. **Cálculo de Taxas de Comissão**
- Obtenção de taxas fixadas na entidade (Abe02)
- Obtenção de taxas fixadas na tabela de preço (Abe40)
- Obtenção de taxas de representantes (Abe05)
- Definição final das taxas no documento

#### 4. **Composição de Observações**
- Composição de observações de uso interno
- Composição de observações fiscais
- Composição de observações ao contribuinte
- Composição de observações de retenção/indenização
- Composição de observações gerais

#### 5. **Processamento Final**
- Cálculo de fidelidade (para PCD específico)
- Cálculo de crédito/cashback (para PCD específico)
- Composição de observações com documentos referenciados
- Atualização do documento fiscal

### ⚠️ Regras de Negócio
#### Validação de Itens
- Todos os itens devem ter código de produto preenchido
- Configurações do produto são carregadas por empresa
- Valores do produto são aplicados conforme UF do endereço principal

#### Observações Fiscais
- Observações do fisco podem ser definidas por produto/UF
- Tratamento especial para CST ICMS 20 (redução de base de cálculo)
- Regime especial Simples Nacional com aproveitamento de ICMS

#### Taxas de Comissão
- Hierarquia de fontes: Entidade → Tabela de Preço → Representante
- Taxas podem ser fixadas em múltiplos níveis
- Valores zerados são substituídos pelas fontes disponíveis

#### Documentos Referenciados
- Identificação de itens devolvidos referenciados
- Composição automática de observação com dados dos documentos
- Inclusão de número, data e chave de acesso da NFe

---

## 3. SRF - Cálculo de Item de Entrada por Importação

### 📖 Descrição
Fórmula para cálculo de impostos e valores em itens de documentos fiscais de entrada por importação, aplicando regras específicas de importação, tributação federal e estadual, conversões de unidades e tratamento de moedas estrangeiras.

### 🎯 Finalidade
Calcular automaticamente os valores fiscais e financeiros de itens em documentos de importação, incluindo impostos de importação (II), IPI, ICMS, PIS/COFINS, conversões monetárias e ajustes de base de cálculo.

### 👥 Público-Alvo
- Departamento Fiscal
- Importação/Exportação
- Contabilidade
- Faturamento

### 📊 Dados e Fontes
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

### ⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento fiscal a ser calculado |

### 🔄 Fluxo do Processo
#### 1. **Configuração Inicial**
- Validação do item do documento (Eaa0103)
- Carregamento do documento fiscal (Eaa01)
- Validação de pessoa física como contribuinte de ICMS
- Carregamento da operação comercial (Abb10) e PCD (Abd01)
- Obtenção de dados da entidade (Abe01) e empresa (Aac10)

#### 2. **Carregamento de Dados Geográficos**
- Identificação do endereço principal da entidade
- Obtenção de município e UF da entidade
- Obtenção de município e UF da empresa
- Determinação se operação é intraestadual ou interestadual

#### 3. **Carregamento de Dados do Produto**
- Configurações fiscais (Abm12) e comerciais (Abm13) do item
- Fatores de conversão de unidades (Abm1301)
- NCM (Abg01) e CFOP (Aaj15)
- Valores do produto por UF (Abm1001)
- Códigos de situação tributária (CSTs)

#### 4. **Cálculo de CFOP**
- Determinação automática de CFOP baseado em:
  - Tipo de operação comercial
  - Localização geográfica (dentro/fora do estado)
  - Tipo de inscrição da entidade (CNPJ/CPF)
  - Presença de IVA no item
  - Tipo de produto (revenda/produção)

#### 5. **Cálculo de Valores do Item**
- **PCD 114**: Cálculo simples (quantidade × valor unitário)
- **PCD 314**: Cálculo complexo de importação:
  - Conversão monetária (FOB para Real)
  - Soma de fretes, seguros e acréscimos
  - Cálculo de CIF (Custo, Seguro e Frete)

#### 6. **Cálculo de Impostos**
- **II (Imposto de Importação)**: Base CIF × Alíquota
- **IPI**: Base (CIF + II) × Alíquota NCM
- **PIS/COFINS**: Base valor total × Alíquotas
- **ICMS**: Cálculo complexo com base em fórmula específica
- Tratamento de isenções e alíquotas zero

#### 7. **Ajustes Finais**
- Conversão de quantidades para unidades de estoque
- Cálculo de pesos brutos e líquidos
- Aplicação de descontos incondicionais
- Cálculo de valor total do documento
- Definição de custo de aquisição

### ⚠️ Regras de Negócio
#### Validações Críticas
- Pessoa física não pode ser contribuinte de ICMS
- Item deve ter configuração fiscal cadastrada
- CFOP deve existir no cadastro após determinação automática
- Cotação monetária obrigatória para importação em moeda estrangeira

#### Cálculo de CFOP
- Dígito inicial fixo em "3" para importação
- CFOPs específicos para operações com IVA (401)
- Diferenciação entre venda (102/108) e revenda (101/107)
- CFOP 109 para operações específicas

#### Cálculo de Importação (PCD 314)
- Conversão obrigatória de moeda estrangeira
- CIF = FOB + Frete + Seguro + Acréscimos
- Base de cálculo do ICMS inclui todos os impostos
- Valor total do documento igual à base de cálculo do ICMS

#### Tratamento de Impostos
- **II**: Zerado quando alíquota for zero
- **IPI**: Base inclui CIF + II, isenção move valor para campo específico
- **PIS/COFINS**: Alíquotas diferenciadas para operações com exterior
- **ICMS**: Cálculo por fórmula ((CIF+impostos)/(1-aliq))×aliq
- **CST 51**: Zera ICMS e alíquota automaticamente

---

## 4. SCV - Pedidos por Item (Eltech)

### 📖 Descrição
Relatório que consolida informações de pedidos agrupados por item, permitindo análise de quantidades solicitadas versus quantidades atendidas, com filtros flexíveis para diferentes tipos de pedidos e situações.

### 🎯 Finalidade
Fornecer uma visão consolidada dos pedidos por item, facilitando o acompanhamento de atendimento, identificação de saldos pendentes e análise de movimentação de produtos.

### 👥 Público-Alvo
- Comercial/Vendas
- Compras
- Planejamento
- Estoques
- Gerência

### 📊 Dados e Fontes
**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Abb01` - Central de documento
- `Eaa0103` - Itens do documento fiscal
- `Eaa01032` - Itens atendidos do SCV
- `Abm01` - Produtos
- `Aam01` - Classes de produtos
- `Abe01` - Entidades (cliente)