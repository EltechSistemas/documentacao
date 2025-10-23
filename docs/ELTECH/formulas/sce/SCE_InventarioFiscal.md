
## 📋 **Propósito Geral**

Calcula valores unitários e totais de itens para fins de inventário fiscal, considerando diferentes regras fiscais.

## 🏢 **Tabelas Utilizadas**

- **Bcb11**: Item do inventário
- **Aac10**: Empresa
- **Abm01**: Item cadastral
- **Abm0101**: Configurações do item por empresa
- **Abm12**: Configuração fiscal do item
- **Abm40**: Grupo de inventário
- **Bcc01**: Saldo de estoque

### **Validações e Configurações**
- Obtém dados do item de inventário (Bcb11)
- Define período de 12 meses para cálculo (data atual - 11 meses)
- Verifica existência da empresa ativa
- Obtém configurações do item (Abm0101)
- Verifica existência da configuração fiscal do item (Abm12)

###  📊 **Cálculos**

###  **Cálculo do Preço Unitário**

- Obtém taxa do grupo de inventário
- Busca preço unitário conforme método definido no grupo
- Se preço zero, busca valor em inventário anterior

#📌 **`precoMedioUnit`**
 **Calcula o preço médio unitário do item baseado no último saldo de estoque (Bcc01) antes da data do inventário.**
 Busca no Bcc01 (saldos de estoque):
   - Filtra por data <= data do inventário
   - Filtra pelo item específico
   - Aplica critérios padrão do sistema
   - Ordena por ID decrescente (mais recente primeiro)
   - Retorna o campo bcc01pmu (preço médio unitário)
   
#📌  **`ultimnoUnitatioSaida`**
**Obtém o último preço unitário de saída do item dentro do período de 12 meses.**

Consulta documentos de saída (Eaa01) no período
 Filtra por:
   - Item específico
   - Documentos de saída (eaa01esMov = 1)
   - Documentos da classe 1 (eaa01clasdoc = 1)
   - Período de 12 meses
   - Ordena pelo documento mais recente

#📌 **`maiorPrecoUnitario`**
**Encontra o maior preço unitário de saída do item dentro do período especificado.**
- Consulta todas as saídas do período
- Aplica filtros específicos:
	   - Item específico
	   - Documentos não cancelados
	   - Operações de código tipo 1 (abb10tipoCod = 1)
	   - Apenas movimentações de saída válidas
- Calcula o valor máximo dos preços unitários  
###  **Cálculo do Valor Total**
`bcb11.bcb11unit` = preço unitário calculado
`bcb11.bcb11total` = quantidade × preço unitário (arredondado para 2 casas)
Atualiza JSON com saldo da conta contábil na data do inventário

##  **Estrutura de Dados de Saída**

### **Campos Atualizados**

- `bcb11unit`: Preço unitário calculado
- `bcb11total`: Valor total (quantidade × unitário)
- `bcb11json`: Campos livres

## ⚠️ **IMPORTANTE!!!** ⚠️

- **Configuração Fiscal**: Item deve ter configuração fiscal válida
- **Grupo de Inventário**: Item deve pertencer a grupo de inventário (próprio ou terceiro)
- **Tipos Fiscais**: Itens dos tipos 7,8,9,99 não exigem grupo de inventário

## 🔧 **Configurações por Item**

O cálculo considera:
- Tipo de item (próprio/terceiro)
- Grupo de inventário associado
- Configurações fiscais
- Valores e percentuais definidos no grupo de inventário
