# SRF_CalculoItem_Entrada_Importacao.md

## 📖 Descrição
Sistema de fórmula para cálculo de itens em documentos de entrada de importação no ERP Strema. O script gerencia a tributação complexa de comércio exterior (II, IPI, PIS, COFINS, ICMS), conversões de unidades, cotações de moedas estrangeiras e regras automáticas de CFOP.

## 🎯 Finalidade
Automatizar o cálculo fiscal e financeiro de itens em notas fiscais de entrada de importação, garantindo que a base de cálculo do ICMS (que inclui impostos federais e despesas aduaneiras) seja composta corretamente conforme a legislação vigente e o tipo de operação (PCD).

## 👥 Público-Alvo
* Departamento Fiscal
* Departamento de Compras / Importação
* Desenvolvedores e Suporte Técnico
* Contabilidade

## ⚙️ Configuração
* **Recursos Necessários:** * Classe `SRF_CalculoItem_Entrada_Importacao`
  * Pacote `strema.formulas.srf`
* **Localização:** `sam.server.samdev.formula` (Tipo de Fórmula: `SCV_SRF_ITEM_DO_DOCUMENTO`)

## 📊 Dados e Fontes
### Tabelas Principais:
* **EAA01 / EAA0103** - Cabeçalho e Itens do Documento de Entrada
* **AAC10** - Dados da Empresa Ativa
* **ABE01** - Entidade (Fornecedor Estrangeiro)
* **ABM01 / ABM0101** - Cadastro de Materiais e Definições por Empresa
* **ABM12 / ABM13** - Configurações Fiscais e Comerciais do Item
* **ABD01** - Tipo de Documento (PCD)
* **AAG10 / AAG1001** - Moedas e Histórico de Cotações
* **AAJ15** - Tabela de CFOP

## ⚙️ Parâmetros do Processo
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| eaa0103 | Object | Sim | Objeto do item do documento em processamento. |
| data_cotacao | Date | Não | Data específica no JSON para busca de cotação de moeda. |
| abd01codigo | String | Sim | Código do PCD (ex: "114", "314") que define a lógica de cálculo. |

## 📋 Saídas do Processo
| Campo | Descrição | Tipo |
| :--- | :--- | :--- |
| eaa0103total | Valor total líquido do item (FOB/CIF convertido). | BigDecimal |
| eaa0103totDoc | Valor total do item com todos os impostos integrados. | BigDecimal |
| eaa0103cfop | CFOP recalculado automaticamente. | Entity (AAJ15) |
| jsonEaa0103 | Campos livres com memória de cálculo (bases e alíquotas). | TableMap |

## 🔄 Fluxo do Processo
1.  **Inicialização e Validação:**
  * Recupera o item (`eaa0103`) e valida se a entidade não é Pessoa Física contribuinte de ICMS.
2.  **Recuperação de Cadastros:**
  * Busca dados da empresa ativa, fornecedor e configurações fiscais/comerciais do material.
3.  **Gestão de Moeda e Cotação:**
  * Para operações específicas (PCD 314), busca a cotação da moeda estrangeira baseada na data do documento ou data informada.
4.  **Processamento Fiscal Sequencial:**
  * **II:** Calcula Imposto de Importação sobre o CIF.
  * **IPI:** Define base (CIF + II) e busca alíquota do cadastro de NCM.
  * **PIS/COFINS:** Calcula alíquotas baseadas no cadastro do item ou exceções de UF ("EX").
  * **ICMS:** Realiza o cálculo "por dentro", onde a base é a soma de todos os impostos federais e despesas, dividida pelo complemento da alíquota interna.
5.  **Finalização:**
  * Calcula o custo de aquisição, ajusta o valor total do documento e popula os campos JSON de retorno.

## ⚠️ Regras de Negócio
### Filtros e Validações
* **Impedimento PF:** Gera erro se tentar importar via Pessoa Física caracterizada como contribuinte.
* **Configuração Fiscal:** Exige que o item possua configuração de impostos (`ABM12`) ativa.
* **Endereço:** É obrigatório haver um endereço marcado como "Principal" no documento para definir a UF de entrada.

### Regras de CFOP
* **Prefixo Fixo:** Sempre inicia com o dígito "3" por se tratar de operação externa.
* **Dinâmica:** O sufixo é alterado conforme o tipo de material (venda/revenda), status de contribuinte do fornecedor e presença de IVA.

### Particularidades do ICMS
* **Cálculo por Dentro:** A base de cálculo do ICMS é recalculada para incluir o próprio imposto na sua base.
* **Diferimento:** Caso o CST de ICMS termine em "51", os valores de imposto e alíquota são zerados no item.

## 🔧 Dependências
* **Bibliotecas:** `br.com.multiorm`, `br.com.multitec.utils`, `sam.server.samdev`.
* **Serviços:** Acesso ao banco de dados para consulta de cotações (`AAG1001`) e joins de tabelas fiscais.

## 📝 Observações Técnicas
* **Precisão Decimal:** Cálculos financeiros utilizam `round(2)`, enquanto fatores de conversão e pesos utilizam precisão estendida.
* **Flexibilidade:** Utiliza intensamente o objeto `TableMap` (JSON) para persistir campos que não possuem colunas fixas no banco de dados.

## 🔄 Métodos Principais
### executar()
Orquestrador principal que carrega as dependências do documento e dispara o fluxo de cálculo.

### calcularItem()
Realiza o cálculo aritmético dos impostos, conversões de unidade de medida e aplicação das regras de CFOP.

### obterTipoFormula()
Identifica a fórmula como do tipo `SCV_SRF_ITEM_DO_DOCUMENTO`.