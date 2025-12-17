## SCE_PleEntSaiPrecoMedio

## 📖 Descrição
Fórmula de cálculo de preço médio e custo para lançamentos de entrada e saída no sistema SCE (Sistema de Controle de Estoque), aplicada em movimentações de itens com base no preço médio atual e nos dados de entrada anteriores.

## 🎯 Finalidade
Calcular automaticamente o custo de saída de itens do estoque com base no preço médio atual e atualizar o preço médio unitário (PMU) conforme os valores de entrada registrados no JSON do movimento.

## 👥 Público-Alvo
- Departamento de Estoque
- Controladoria
- Departamento Financeiro
- Desenvolvedores de fórmulas do sistema

## 📊 Dados e Fontes
**Tabelas Principais:**
- `BCC01` - Movimentações de estoque
- `ABM0101` - Itens do estoque (preço médio atual)
- `ABM20` - Tipos de movimentação

**Entidades Envolvidas:**
- `Bcc01` - Movimentação atual
- `Abm0101` - Item do estoque
- `Abm20` - Tipo de movimento

## ⚙️ Parâmetros da Fórmula
A fórmula não possui parâmetros de entrada configuráveis via interface. Os dados são obtidos diretamente das entidades vinculadas ao contexto de execução.

## 🔧 Métodos Principais

### `executar()`
Método principal de execução da fórmula, responsável por:
- Obter as entidades `Bcc01` e `Abm0101` do contexto
- Calcular o custo da movimentação com base no preço médio atual
- Buscar a última entrada correspondente ao item, lote e série
- Atualizar o preço médio unitário (PMU) com base no JSON da entrada anterior
- Atribuir os valores calculados de volta à movimentação

### `obterTipoFormula()`
Retorna o tipo da fórmula como `FormulaTipo.LCTO_SCE`, indicando que se trata de uma fórmula de lançamento do módulo SCE.

## 📝 Fluxo de Execução

1. **Inicialização**
   - Obtém a movimentação atual (`Bcc01`) e o item (`Abm0101`)
   - Carrega o JSON associado à movimentação (se existir)

2. **Cálculo do Custo**
   - Recupera a quantidade e o preço médio atual do item
   - Calcula o custo base: `qtde * precoMedioAtual`
   - Arredonda o resultado para 2 casas decimais
   - Atribui o custo calculado à movimentação

3. **Busca da Entrada Anterior**
   - Consulta a última entrada correspondente ao mesmo item, lote, série, status e controle
   - Aplica filtros padrão do sistema via `getSamWhere().getCritPadrao()`
   - Ordena por data decrescente e limita a 1 resultado

4. **Atualização do Preço Médio**
   - Se o tipo de movimento não for "205" (saída específica) e houver entrada anterior:
     - Recupera os valores unitários do JSON da entrada
     - Atualiza o PMU da movimentação atual com o valor da entrada
   - Se não houver entrada anterior, mantém o PMU atual

5. **Persistência**
   - Atualiza o JSON da movimentação com os novos valores
   - Mantém os campos de custo fixo e variável como zero

## ⚠️ Regras de Negócio

### Cálculo de Custo
- O custo é calculado com base no preço médio atual do item (`abm0101.abm0101pmu`)
- O resultado é arredondado para 2 casas decimais
- Campos de custo variável e fixo são sempre zerados

### Atualização do PMU
- Apenas movimentos de saída (código diferente de "205") atualizam o PMU
- O PMU é atualizado apenas se existir uma entrada anterior com JSON contendo `vlr_unit`
- Caso contrário, o PMU permanece inalterado

### Validações
- Apenas endereços principais são considerados (implícito nos filtros padrão)
- Movimentações devem estar relacionadas a itens, lotes e séries válidos
- A busca da entrada anterior aplica filtros de status e controle correspondentes

## 🔄 Dependências

**Classes:**
- `FormulaBase` - Classe base para fórmulas do sistema
- `Bcc01` - Entidade de movimentação de estoque
- `Abm0101` - Entidade de item de estoque
- `Abm20` - Entidade de tipo de movimento

**Bibliotecas:**
- `br.com.multiorm` - ORM e critérios de consulta
- `br.com.multitec.utils` - Utilitários e coleções
- `sam.dicdados` - Definição de tipos de fórmula
- `sam.model.entities` - Modelos de entidades do sistema

## 🎨 Saída da Fórmula
A fórmula não gera relatórios ou arquivos de saída. Sua execução resulta na atualização dos seguintes campos da entidade `Bcc01`:
- `bcc01custo` - Custo calculado da movimentação
- `bcc01pmu` - Preço médio unitário atualizado
- `bcc01json` - JSON atualizado com valores unitários da entrada anterior

## 📌 Observações Técnicas
- Uso de `TableMap` para manipulação de JSON armazenado no banco
- Consulta otimizada com `setMaxResults(1)` e ordenação por data decrescente
- Implementação compatível com o framework de fórmulas do SAM
- Código identificado por metadados no final do arquivo: 