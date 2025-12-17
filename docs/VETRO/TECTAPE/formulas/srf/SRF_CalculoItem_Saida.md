## SRF_CalculoItem_Saida

## 📖 Descrição
Fórmula de cálculo fiscal para itens de documentos de saída no sistema SRF (Sistema de Registro Fiscal), responsável por calcular impostos, definir CFOPs, CSTs e tratar operações especiais como Zona Franca e diferencial de alíquota.

## 🎯 Finalidade
Calcular automaticamente os valores fiscais (ICMS, IPI, PIS, COFINS) e definir as configurações fiscais (CFOP, CST) para itens de documentos de saída, considerando as regras tributárias vigentes e operações especiais como Zona Franca, Amazônia Ocidental e diferencial de alíquota interestadual.

## 👥 Público-Alvo
- Departamento Fiscal
- Controladoria
- Departamento Financeiro
- Departamento Comercial
- Desenvolvedores de fórmulas do sistema

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA0103` - Itens do documento fiscal
- `EAA01` - Documento fiscal
- `EAA0101` - Endereços do documento
- `EAA0102` - Dados gerais do documento
- `ABB01` - Central de documentos
- `ABD01` - Tipo de documento (PCD)
- `ABE01` - Entidades (clientes/fornecedores)
- `ABE0101` - Endereços da entidade
- `ABM01` - Itens
- `ABM0101` - Configurações do item por empresa
- `ABM10` - Valores do item
- `ABM1001` - Valores do item por estado
- `ABM1002` - Valores do item por município
- `ABM1003` - Valores do item por entidade
- `ABM12` - Dados fiscais do item
- `ABM13` - Dados comerciais do item
- `ABM1301` - Fatores de conversão de unidade
- `AAC10` - Empresa
- `AAG01` - Países
- `AAG02` - Estados
- `AAG0201` - Municípios
- `AAJ10` - CST ICMS
- `AAJ11` - CST IPI
- `AAJ12` - CST PIS
- `AAJ13` - CST COFINS
- `AAJ14` - CSOSN
- `AAJ15` - CFOP
- `AAM06` - Unidades de medida
- `ABG01` - NCM
- `ABB10` - Operação comercial
- `ABE30` - Condição de pagamento
- `ABE40` - Tabela de preços

**Entidades Envolvidas:**
- `Eaa0103` - Item do documento atual
- `Eaa01` - Documento fiscal
- `Eaa0101` - Endereço principal do documento
- `Eaa0102` - Dados gerais do documento
- `Abb01` - Central de documento
- `Abd01` - Tipo de documento (PCD)
- `Abe01` - Entidade (cliente)
- `Abe0101` - Endereço principal da entidade
- `Abm01` - Item
- `Abm0101` - Configuração do item por empresa
- `Abm10` - Valores do item
- `Abm1001` - Valores do item por estado
- `Abm1002` - Valores do item por município
- `Abm1003` - Valores do item por entidade
- `Abm12` - Dados fiscais do item
- `Abm13` - Dados comerciais do item
- `Aac10` - Empresa
- `Aag01` - País
- `Aag02` - Estado
- `Aag0201` - Município
- `Aaj10` - CST ICMS
- `Aaj11` - CST IPI
- `Aaj12` - CST PIS
- `Aaj13` - CST COFINS
- `Aaj14` - CSOSN
- `Aaj15` - CFOP
- `Aam06` - Unidade de medida
- `Abg01` - NCM
- `Abb10` - Operação comercial
- `Abe30` - Condição de pagamento do documento
- `Abe30Item` - Condição de pagamento do item
- `Abe40` - Tabela de preços

## ⚙️ Parâmetros da Fórmula
A fórmula não possui parâmetros de entrada configuráveis via interface. Os dados são obtidos diretamente das entidades vinculadas ao contexto de execução, principalmente do item do documento (`Eaa0103`).

## 🔧 Métodos Principais

### `executar()`
Método principal de execução da fórmula, responsável por:
1. **Carregar entidades relacionadas** (documento, empresa, entidade, item, configurações fiscais)
2. **Validar dados básicos** (endereço principal, tipo de documento, configuração fiscal do item)
3. **Carregar campos livres (JSON)** de todas as entidades relevantes
4. **Definir preço unitário e taxas de comissão** através do método `setarObterPrecoUnitarioTaxasComissaoItem()`
5. **Executar cálculo fiscal completo** através do método `calcularItem()`
6. **Atualizar o item do documento** com os resultados calculados

### `setarObterPrecoUnitarioTaxasComissaoItem()`
- Busca o preço unitário na tabela de preços (`Abe40`) considerando:
  - Item específico
  - Tabela de preços vigente
  - Condição de pagamento (do item ou do documento)
  - Quantidade comercial
  - Taxa de desconto informada no JSON
- Busca taxas de comissão na configuração comercial do item (`Abm13`)
- Valida vencimento da tabela de preços
- Define preço unitário e taxas de comissão no item

### `calcularItem()`
Método complexo que realiza todos os cálculos fiscais:
1. **Ajuste de CFOP** conforme operação, tipo de item e localização
2. **Cálculo de quantidades** (uso, volume) com fatores de conversão
3. **Cálculo de pesos** (líquido e bruto)
4. **Cálculo de desconto incondicional**
5. **Cálculo de IPI** com diferentes CSTs (50, 51, 52, 53, 54, 55, 99)
6. **Cálculo de ICMS** com múltiplos CSTs (00, 10, 20, 30, 40, 41, 50, 51, 60, 70, 90) e tratamento de ST
7. **Cálculo de PIS** com CSTs (01, 02, 03, 04, 05, 06, 07, 08, 09, 49)
8. **Cálculo de COFINS** com CSTs (01, 02, 03, 04, 05, 06, 07, 08, 09, 49)
9. **Tratamento especial para Zona Franca/Área de Livre Comércio/Amazônia Ocidental**
10. **Cálculo de diferencial de alíquota interestadual** (a partir de 2016)
11. **Cálculo de valores aproximados de impostos** para venda a consumidor final

## 📝 Fluxo de Execução

### 1. **Inicialização e Validação**
- Obtém o item do documento (`Eaa0103`) e suas relações
- Valida se a entidade é pessoa física contribuinte de ICMS (lança exceção se for)
- Identifica o endereço principal da entidade no documento
- Valida se o documento é de saída (PCD)
- Carrega todas as entidades relacionadas (empresa, estado, município, configurações do item)

### 2. **Configuração de Preços e Comissões**
- Verifica validade da tabela de preços
- Busca preço unitário na tabela de preços considerando múltiplos critérios
- Busca taxas de comissão na configuração do item
- Define preço unitário e taxas no item

### 3. **Cálculos Fiscais Principais**
- **Definição de CFOP** baseada em operação, tipo de item e localização
- **Cálculo de valores totais** e quantidades convertidas
- **Cálculo de IPI** com diferentes regimes tributários
- **Cálculo de ICMS** incluindo substituição tributária e reduções de base
- **Cálculo de PIS/COFINS** com diferentes alíquotas
- **Tratamento de operações especiais** (Zona Franca, Amazônia Ocidental)

### 4. **Tratamentos Especiais**
- **Zona Franca/Área Livre Comércio**: ajuste de CFOPs, CSTs e cálculos específicos
- **Amazônia Ocidental**: tratamento especial para IPI
- **Diferencial de alíquota interestadual**: cálculo de partilha entre estados
- **Consumidor final**: cálculo de valores aproximados de impostos

### 5. **Finalização**
- Cálculo do total do documento
- Cálculo do valor financeiro
- Atualização do JSON do item com todos os valores calculados
- Persistência do item atualizado

## ⚠️ Regras de Negócio

### Validações Críticas
1. **Pessoa Física Contribuinte**: Não permitir se a entidade for pessoa física e contribuinte de ICMS
2. **Endereço Principal**: Obrigatório existir endereço principal da entidade no documento
3. **Tipo de Documento**: Apenas documentos de saída são processados
4. **Configuração Fiscal do Item**: Obrigatória existência e tipo fiscal definido
5. **Tabela de Preços**: Não pode estar vencida

### Regras de CFOP
- Determinação automática baseada em operação, tipo de item e localização
- Ajustes para operações interestaduais e com substituição tributária
- CFOPs específicos para Zona Franca (6109, 6110)

### Regras de CST
- **ICMS**: Determinação baseada em tipo de item, operação e localização
- **IPI**: Ajustes conforme alíquota (0% → CST 51, sem alíquota → CST 53)
- **PIS/COFINS**: Aplicação conforme configuração do item
- **Zona Franca**: CSTs específicos (ICMS 040, IPI 55, PIS/COFINS 06)

### Cálculos Especiais
1. **Substituição Tributária**: Cálculo de IVA, base de cálculo e valor do ICMS-ST
2. **Redução de Base**: Aplicação de percentuais de redução na base de cálculo
3. **Diferencial de Alíquota**: Cálculo de partilha entre estados (2016 em diante)
4. **Zona Franca**: Cálculo de desoneração de ICMS e ajustes de PIS/COFINS

## 🔄 Dependências

**Classes:**
- `FormulaBase` - Classe base para fórmulas do sistema
- Todas as entidades do modelo SAM mencionadas na seção "Entidades Envolvidas"

**Bibliotecas:**
- `br.com.multiorm` - ORM e critérios de consulta
- `br.com.multitec.utils` - Utilitários e coleções
- `sam.dicdados` - Definição de tipos de fórmula
- `sam.model.entities` - Modelos de entidades do sistema
- `sam.server.samdev.utils` - Utilitários do sistema

## 🎨 Saída da Fórmula
A fórmula não gera relatórios ou arquivos de saída. Sua execução resulta na atualização dos seguintes campos da entidade `Eaa0103`:

### Campos Diretos
- `eaa0103unit` - Preço unitário definido
- `eaa0103txComis0` a `eaa0103txComis4` - Taxas de comissão
- `eaa0103total` - Valor total do item
- `eaa0103qtUso` - Quantidade de uso convertida
- `eaa0103totDoc` - Total do documento
- `eaa0103totFinanc` - Valor financeiro
- `eaa0103clasReceita` - Classificação da receita
- `eaa0103cfop` - CFOP definido
- `eaa0103cstIcms` - CST ICMS definido
- `eaa0103cstIpi` - CST IPI definido
- `eaa0103cstPis` - CST PIS definido
- `eaa0103cstCofins` - CST COFINS definido
- `eaa0103motDesIcms` - Motivo da desoneração de ICMS

### Campos no JSON (`eaa0103json`)
- **Valores básicos**: `vlr_vlme`, `vlr_pl`, `vlr_pb`, `vlr_desc`
- **IPI**: `ipi_bc`, `ipi_aliq`, `ipi_ipi`, `ipi_isento`, `ipi_outras`
- **ICMS**: `icm_bc`, `icm_reduc_bc`, `icm_aliq`, `icm_icm`, `icm_isento`, `icm_outras`
- **ICMS-ST**: `st_bc`, `st_aliq`, `st_icm`, `tx_iva_st`
- **PIS**: `pis_bc`, `pis_aliq`, `pis_pis`
- **COFINS**: `cofins_bc`, `cofins_aliq`, `cofins_cofins`
- **Zona Franca**: `icmszf_bc`, `tx_icms_zf`, `vlr_descicmszf`, `icm_desonerado`
- **Diferencial de Alíquota**: `partilha_aliq`, `internaufdest_aliq`, `fcpufdest_aliq`, `icmsufdest_bc`, `interesuf_aliq`, `icms_fcp`, `interufdest_icms`, `icms_ufdest`, `uforig_icms`
- **Impostos Aproximados**: `impfed_vlr`, `impest_vlr`, `impaprx_vlr`

## 📌 Observações Técnicas

### Arquitetura
- Fórmula extensa com mais de 1500 linhas de código
- Uso intensivo de `TableMap` para manipulação de JSON
- Múltiplas consultas ao banco de dados para carregar entidades relacionadas
- Lógica condicional complexa para tratamento de diferentes cenários fiscais

### Performance
- Carrega múltiplas entidades relacionadas que podem impactar performance
- Executa consultas SQL adicionais para preços e configurações
- Processamento intensivo de cálculos matemáticos

### Manutenibilidade
- Código altamente especializado em legislação fiscal brasileira
- Múltiplas regras condicionais aninhadas
- Necessidade de atualização constante conforme mudanças na legislação

### Metadados
- Código identificado por metadados no final do arquivo: `meta-sis-eyJ0aXBvIjoiZm9ybXVsYSIsImZvcm11bGF0aXBvIjoiNjIifQ==`
- Tipo de fórmula: `FormulaTipo.SCV_SRF_ITEM_DO_DOCUMENTO`

### Limitações Conhecidas
- Não contempla CST de PIS 03 (alíquota por unidade de medida)
- Não contempla CST de COFINS 03 (alíquota por unidade de medida)
- Lógica específica para legislação brasileira
- Dependência de múltiplas configurações de sistema

## 🔧 Configurações Necessárias

### Pré-requisitos do Sistema
1. **Cadastro Completo de Itens** com configurações fiscais e comerciais
2. **Cadastro de Estados e Municípios** com alíquotas atualizadas
3. **Configuração de Operações Comerciais** com tipos definidos
4. **Tabelas de Preços** vigentes e configuradas
5. **Cadastro de CFOPs e CSTs** completos no sistema

### Configurações Específicas
1. **JSON das Entidades**: Configurações específicas em campos livres
2. **Fatores de Conversão**: Configurados nos dados comerciais dos itens
3. **Alíquotas por Estado**: Configuradas no cadastro de estados
4. **Regimes Especiais**: Configurados nas entidades (REB, regime especial)

## ⚠️ Considerações de Implementação

### Complexidade Fiscal
- Fórmula implementa lógica complexa da legislação tributária brasileira
- Requer conhecimento especializado em ICMS, IPI, PIS, COFINS
- Necessidade de acompanhamento constante de mudanças legais

### Testes
- Requer testes abrangentes para diferentes cenários fiscais
- Necessidade de dados de teste representativos de diferentes estados e operações
- Testes devem cobrir casos especiais (Zona Franca, ST, diferencial de alíquota)

### Monitoramento
- Monitorar performance em documentos com muitos itens
- Logs detalhados para diagnóstico de problemas
- Alertas para configurações ausentes ou inconsistentes

### Atualizações
- Revisão periódica para adequação à legislação vigente
- Atualização de alíquotas e regras conforme publicações oficiais
- Manutenção dos tratamentos especiais (Zona Franca, diferencial de alíquota)