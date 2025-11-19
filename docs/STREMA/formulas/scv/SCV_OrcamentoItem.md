# SCV_OrcamentoItem - Cálculo de Itens de Orçamento

## 📖 Descrição
Classe responsável pelo cálculo de valores, impostos e totais para itens de orçamento, considerando configurações específicas por empresa, entidade, estado e município.

## 🎯 Finalidade
Realizar cálculos completos de preços, impostos e totais para itens de orçamento, incluindo ICMS, IPI, PIS, COFINS, descontos e valores comerciais, com base nas configurações fiscais e comerciais do sistema.

## 👥 Público-Alvo
- Departamento Comercial
- Vendedores
- Departamento Fiscal
- Gestores de Orçamentos

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| cbe1001 | Cbe1001 | Sim | Item do orçamento a ser calculado |
| procInvoc | String | Não | Processo de invocação (CAS0240, CAS0242) |

## 📋 Estrutura de Dados Principais

### Entidades Envolvidas:
- **Cbe1001** - Item do orçamento
- **Cbe10** - Cabeçalho do orçamento
- **Abe40** - Tabela de preço
- **Abe4001** - Item da tabela de preço
- **Abb01** - Central de documento
- **Abe01** - Entidade (cliente)
- **Abe0101** - Endereço da entidade
- **Abm01** - Item cadastral
- **Abm0101** - Configuração do item por empresa
- **Abm10** - Valores do item
- **Abm12** - Dados fiscais do item
- **Abm13** - Dados comerciais do item

### Campos Calculados:
- Preço de custo
- Totais do item
- Peso bruto e líquido
- Descontos
- Base de cálculo e valores de impostos (ICMS, IPI, PIS, COFINS)
- Total do documento

## 🔄 Fluxo do Processo

### 1. **Inicialização e Validação**
- Carrega dados do item do orçamento
- Verifica processo de invocação
- Valida existência do item

### 2. **Carregamento de Dados Relacionados**
- Orçamento (Cbe10)
- Tabela de preço (Abe40)
- Entidade e endereço (Abe01, Abe0101)
- Configurações do item (Abm0101, Abm12, Abm13)
- Dados geográficos (município, UF)

### 3. **Cálculo do Preço de Custo**
- Busca preço na tabela de preço
- Para itens compostos, calcula valor da composição
- Considera mão de obra e itens componentes

### 4. **Cálculos Comerciais**
- Total do item (quantidade × unitário)
- Conversão para volume
- Cálculo de pesos
- Aplicação de descontos

### 5. **Cálculos Fiscais**
- **IPI**: Base de cálculo e valor
- **ICMS**: Base, alíquota e valor
- **PIS**: Base, alíquota e valor
- **COFINS**: Base, alíquota e valor

### 6. **Consolidação de Totais**
- Total do documento
- Total financeiro
- Atualização do JSON do item

## ⚠️ Regras de Negócio

### Hierarquia de Alíquotas ICMS:
1. **Entidade específica** (Abm1003)
2. **Município** (Abm1002)
3. **Estado** (Abm1001)
4. **Valores padrão do item** (Abm10)
5. **Configuração do item** (Abm0101)
6. **Regras por UF** (interior × interestadual)

### Cálculo de Preço para Itens Compostos:
- Item principal (seq = 1) tem valor base
- Itens do tipo serviço representam % de mão de obra
- Demais itens somam ao custo
- Total = custo + item principal + (item principal × % mão de obra)

### Base de Cálculo de Impostos:
- **IPI**: Total + frete + seguro + outras despesas
- **ICMS**: Total + frete + seguro + outras despesas - desconto
- **PIS/COFINS**: Total + frete + seguro + outras despesas - ICMS

### Regras Específicas por CST:
- CST 100, 300, 800: Alíquota interestadual de 4%
- Demais CSTs: Alíquota de saída do estado

### Validações Críticas:
- Município da entidade obrigatório
- Configuração fiscal do item obrigatória
- Tipo fiscal do item obrigatório
- Composição de produto única por item

## 🎨 Saídas Geradas

| Campo | Tipo | Descrição |
|-------|------|-----------|
| preco_custo | BigDecimal | Preço de custo do item |
| vlr_vlme | BigDecimal | Valor do volume |
| vlr_pl | BigDecimal | Valor do peso líquido |
| vlr_pb | BigDecimal | Valor do peso bruto |
| ipi_bc | BigDecimal | Base de cálculo do IPI |
| ipi_aliq | BigDecimal | Alíquota do IPI |
| ipi_ipi | BigDecimal | Valor do IPI |
| icm_bc | BigDecimal | Base de cálculo do ICMS |
| icm_aliq | BigDecimal | Alíquota do ICMS |
| icm_icm | BigDecimal | Valor do ICMS |
| pis_bc | BigDecimal | Base de cálculo do PIS |
| pis_aliq | BigDecimal | Alíquota do PIS |
| pis_pis | BigDecimal | Valor do PIS |
| cofins_bc | BigDecimal | Base de cálculo do COFINS |
| cofins_aliq | BigDecimal | Alíquota do COFINS |
| cofins_cofins | BigDecimal | Valor do COFINS |

## 🔧 Dependências

### Bibliotecas:
- `br.com.multiorm` - Acesso a dados
- `br.com.multitec.utils` - Utilitários e validações

### Entidades Principais:
- Cbe1001 (Item do orçamento)
- Abm01 (Item cadastral)
- Abe40 (Tabela de preço)
- Abe01 (Entidade)

### Configurações:
- Campos livres (JSON) das entidades
- Parâmetros de cálculo fiscal
- Hierarquia de valores por localidade

## 📝 Observações Técnicas

### Tratamento de Dados:
- Uso extensivo de TableMap para campos JSON
- Arredondamento para 2-4 casas decimais conforme necessidade
- Validações com MultiValidationException

### Performance:
- Carregamento lazy de entidades relacionadas
- Consultas otimizadas para composição de produtos
- Cache de configurações por item/empresa

### Hierarquia de Configurações:
- Implementa fallback para valores não encontrados
- Considera especificidade (entidade > município > estado > padrão)
- Suporte a regras interestaduais

### Casos de Exceção:
- Processos CAS0240 e CAS0242 são ignorados
- Itens sem configuração fiscal geram erro
- Município obrigatório para entidade
- Composição duplicada não permitida