# SCV_OrcamentoItem.md

## 📖 Descrição
Sistema de fórmula para o processamento de itens em orçamentos de venda no ERP Strema. O script automatiza a busca de preços de custo (via tabela de preço ou composição de produto), calcula pesos, volumes e toda a tributação incidente (ICMS, IPI, PIS, COFINS), consolidando o valor total do documento.

## 🎯 Finalidade
Garantir a precisão financeira e fiscal na fase de orçamento, permitindo que o vendedor tenha o custo real e os impostos calculados automaticamente com base na localização do cliente e nas configurações do material.

## 👥 Público-Alvo
* Equipe de Vendas / Comercial
* Departamento Fiscal
* Controladoria
* Desenvolvedores / Suporte Técnico

## ⚙️ Configuração
* **Recursos Necessários:** * Classe `SCV_OrcamentoItem`
    * Pacote `strema.formulas.scv`
* **Localização:** `sam.server.samdev.formula`
* **Tipo de Fórmula:** `SCV_ITEM_ORCAMENTO` (ID correspondente ao tipo de fórmula de item de orçamento).

## 📊 Dados e Fontes
### Tabelas Principais:
* **CBE10 / CBE1001** - Cabeçalho e Itens do Orçamento de Venda.
* **ABE40 / ABE4001** - Tabelas de Preço e seus Itens.
* **ABP20 / ABP2001 / ABP20011** - Estrutura de Composição de Produtos (Ficha Técnica).
* **ABE01 / ABE0101** - Entidade (Cliente) e Endereços.
* **ABM01 / ABM0101** - Cadastro de Materiais e Definições por Empresa.
* **ABM10 / 1001 / 1002 / 1003** - Matriz de Valores e Impostos por UF, Município e Entidade.
* **AAG02 / AAG0201** - Estados (UF) e Municípios.

## ⚙️ Parâmetros do Processo
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| cbe1001 | Object | Sim | Objeto do item do orçamento atual em processamento. |
| procInvoc | String | Não | Identificador do processo invocador (bloqueia execução se for "CAS0240" ou "CAS0242"). |

## 📋 Saídas do Processo
| Campo | Descrição | Tipo |
| :--- | :--- | :--- |
| cbe1001total | Valor total líquido do item (Quantidade x Unitário). | BigDecimal |
| cbe1001totDoc | Valor total do item com impostos, frete, seguro e despesas. | BigDecimal |
| cbe1001totFinanc| Valor final que será integrado ao módulo financeiro. | BigDecimal |
| jsonCbe1001 | Memória de cálculo (Bases e alíquotas de IPI, ICMS, PIS, COFINS). | TableMap |

## 🔄 Fluxo do Processo
1.  **Carga de Contexto:** O script recupera os dados do orçamento, cliente (incluindo UF de destino), empresa logada (UF de origem) e as configurações fiscais do material.
2.  **Definição do Preço de Custo:**
    * Busca primeiramente na **Tabela de Preço** associada (`ABE4001`).
    * Caso não exista, executa uma consulta SQL na **Composição do Produto** (`ABP20`), somando os insumos e aplicando percentual de mão de obra sobre o item principal.
3.  **Cálculo de Logística:** Calcula automaticamente o Peso Líquido, Peso Bruto e o Volume (Vlme) utilizando os fatores de conversão do cadastro do item.
4.  **Cálculo de IPI:** A base é composta pelo (Total + Frete + Seguro + Outras Despesas). A alíquota é extraída do NCM (`ABG01`).
5.  **Cálculo de ICMS:**
    * Determina a alíquota seguindo a hierarquia de prioridade: Entidade > Município > UF do Item > Cadastro do Item > Regra de UF Origem/Destino.
    * Aplica a alíquota de 4% para produtos de origem estrangeira (CSTs 100, 300, 800) em operações interestaduais.
    * Realiza a inclusão do IPI na base de ICMS caso o cenário seja de **Consumo Final** ou cliente **Não Contribuinte**.
6.  **Cálculo de PIS/COFINS:** Calcula os valores aplicando as alíquotas cadastradas no item, deduzindo o valor do ICMS da base de cálculo (Exclusão do ICMS da base do PIS/COFINS).
7.  **Totalização:** Consolida o `cbe1001totDoc` somando impostos e despesas e subtraindo os descontos incondicionais.

## ⚠️ Regras de Negócio
### Formação de Preço por Composição
> Quando o custo é derivado da composição, o sistema identifica o item de sequência 1 como o componente principal. Itens do tipo "Serviço" (tipo 3) são tratados como percentual de mão de obra sobre o valor do principal, enquanto os demais insumos são somados ao custo nominalmente.

### Hierarquia de Alíquotas de ICMS
O sistema busca a alíquota mais específica para o cenário, nesta ordem:
1.  Exceção por **Cliente/Entidade** (`ABM1003`).
2.  Exceção por **Município** (`ABM1002`).
3.  Exceção por **Estado/UF** do item (`ABM1001`).
4.  Configuração genérica no **Cadastro do Item por Empresa** (`ABM0101`).
5.  Alíquotas internas/interestaduais padrão das tabelas de **UF** (`AAG02`).

## 🔧 Dependências
* **Consultas SQL:** Utiliza o método `buscarItemComposicao` para realizar joins complexos entre a estrutura de produto e tabelas de preço.
* **Validações:** O processo é interrompido caso o município do cliente não esteja cadastrado ou se o item não possuir uma configuração fiscal ativa (`ABM12`).

## 🔄 Métodos Principais
### executar()
Ponto de entrada que carrega todas as entidades relacionadas e prepara os campos JSON para o cálculo.

### calcularItem()
Realiza a lógica aritmética dos impostos e totalizadores do documento.

### buscarItemComposicao()
Executa a query SQL para decompor a ficha técnica do produto e retornar os valores de custo de cada insumo.