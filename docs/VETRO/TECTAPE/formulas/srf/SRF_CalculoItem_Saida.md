# SRF - Cálculo de Item de Saída

## 📖 Descrição
Fórmula responsável pelo cálculo fiscal, tributário e comercial de itens em documentos fiscais de saída. Realiza a apuração de impostos (ICMS, IPI, PIS, COFINS), ajustes de CFOP, cálculo de descontos, comissões e tratamento especial para operações em Zona Franca e Reforma Tributária (CBS/IBS).

## 🎯 Finalidade
Calcular automaticamente os valores fiscais e tributários de itens em documentos de saída, garantindo conformidade com a legislação tributária brasileira e regras comerciais específicas da empresa.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Contabilidade
- Comercial

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa0103` - Itens do documento fiscal
- `Eaa01` - Documentos fiscais
- `Abe40` - Tabela de preços
- `Abe4001` - Preços por item na tabela
- `Abm01` - Itens (produtos/serviços)
- `Abm0101` - Configuração do item por empresa
- `Abm10` - Valores do item
- `Abm1001` - Valores do item por estado
- `Abm1002` - Valores do item por município
- `Abm1003` - Valores do item por entidade
- `Abm12` - Dados fiscais do item
- `Abm13` - Dados comerciais do item
- `Aaj10` - CST ICMS
- `Aaj11` - CST IPI
- `Aaj12` - CST PIS
- `Aaj13` - CST COFINS
- `Aaj14` - CSOSN
- `Aaj15` - CFOP
- `Aaj07` - Classificação tributária CBS/IBS
- `Aaj09` - CST CBS/IBS

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento fiscal a ser calculado |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação do item do documento (Eaa0103)
- Carregamento do documento fiscal relacionado (Eaa01)
- Verificação do tipo de documento (saída)
- Validação de entidade contribuinte de ICMS

### 2. **Carregamento de Dados**
- Dados da empresa (Aac10)
- Dados da entidade/cliente (Abe01)
- Endereços da entidade e empresa
- Configurações do item (Abm01, Abm0101)
- Dados fiscais e comerciais do item
- Operação comercial (Abb10)

### 3. **Cálculo de Preço e Comissões**
- Busca de preço na tabela de preços (Abe40, Abe4001)
- Verificação de vencimento da tabela de preços
- Cálculo de taxas de comissão
- Aplicação de descontos informados nos campos livres

### 4. **Cálculos Fiscais e Tributários**
- **CFOP**: Ajuste automático conforme operação
- **ICMS**: Cálculo de base, alíquota e valor com tratamento para:
  - Operações normais
  - Substituição tributária (ST)
  - Redução de base de cálculo
  - Operações isentas
  - Diferencial de alíquota interestadual
- **IPI**: Cálculo conforme CST
- **PIS/COFINS**: Cálculo de bases e valores
- **Reforma Tributária**: Cálculo de CBS/IBS quando aplicável

### 5. **Tratamentos Especiais**
- **Zona Franca/Área de Livre Comércio**: Regimes especiais de tributação
- **Consumidor Final**: Cálculo de impostos aproximados
- **Retornos**: Ajustes específicos para itens retornados

### 6. **Cálculos Finais**
- Total do documento
- Total financeiro
- Pesos e volumes
- Outras informações complementares

## ⚠️ Regras de Negócio

### Validações Iniciais
- Entidades pessoa física não podem ser contribuintes de ICMS
- Documentos devem ser de saída
- Item deve ter configuração fiscal válida

### Cálculo de Preços
- Tabelas de preço vencidas são rejeitadas
- Busca considera: item, tabela, condição de pagamento, quantidade e desconto
- Taxas de comissão são buscadas na configuração do item se não encontradas na tabela

### Regras Fiscais
- **CFOP**: Ajustado automaticamente conforme tipo de operação (venda, revenda) e localização
- **ICMS**: Tratamento diferenciado para:
  - Contribuintes vs não contribuintes
  - Operações internas vs interestaduais
  - Consumidor final
  - Substituição tributária
- **Zona Franca**: Regime especial com isenções e reduções
- **Reforma Tributária**: Aplicação de CBS/IBS quando configurado

### Campos Livres (JSON)
Utilizados para armazenar valores calculados e configurações:
- `vlr_desc_tx` - Taxa de desconto
- `vlr_frete_dest` - Valor do frete
- `vlr_seguro` - Valor do seguro
- `vlr_outras` - Outras despesas
- `icm_aliq` - Alíquota de ICMS
- `st_aliq` - Alíquota de ST
- `cbs_aliq` - Alíquota de CBS
- `ibs_uf_aliq` - Alíquota de IBS estadual

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de cálculo do item.

### `setarObterPrecoUnitarioTaxasComissaoItem()`
Busca o preço unitário e taxas de comissão na tabela de preços.

### `calcularItem()`
Realiza todos os cálculos fiscais, tributários e comerciais do item.

### `aplicarReformaTributaria()`
Aplica as regras da Reforma Tributária (CBS/IBS) quando configurado.

### `formulaZFMeALC()`
Tratamento especial para Zona Franca e Áreas de Livre Comércio.

### `red_bc_aliq()`
Calcula as alíquotas efetivas com reduções aplicadas.

## 📊 Estrutura de Saída

**Item Calculado (Eaa0103):**
- `eaa0103unit` - Preço unitário calculado
- `eaa0103total` - Valor total do item
- `eaa0103totDoc` - Total do documento
- `eaa0103totFinanc` - Total financeiro
- `eaa0103cfop` - CFOP ajustado
- `eaa0103cstIcms` - CST de ICMS
- `eaa0103cstIpi` - CST de IPI
- `eaa0103cstPis` - CST de PIS
- `eaa0103cstCofins` - CST de COFINS
- `eaa0103json` - Campos livres com todos os valores calculados

**Campos Livres (JSON) Principais:**
- `icm_bc`, `icm_aliq`, `icm_icm` - Base, alíquota e valor do ICMS
- `st_bc`, `st_aliq`, `st_icm` - Base, alíquota e valor do ICMS ST
- `ipi_bc`, `ipi_aliq`, `ipi_ipi` - Base, alíquota e valor do IPI
- `pis_bc`, `pis_aliq`, `pis_pis` - Base, alíquota e valor do PIS
- `cofins_bc`, `cofins_aliq`, `cofins_cofins` - Base, alíquota e valor do COFINS
- `cbs_ibs_bc`, `vlr_cbs`, `vlr_ibs` - Base e valores de CBS/IBS
- `vlr_desc` - Valor do desconto
- `vlr_pl`, `vlr_pb` - Peso líquido e bruto

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários e validações
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Módulo:** SRF (Sistema de Regras Fiscais)

## 📝 Observações Técnicas

### Tratamento de Exceções
- `ValidacaoException` lançada para erros de negócio
- Validações preventivas para evitar cálculos incorretos
- Mensagens de erro claras para o usuário

### Performance
- Uso de caching de sessão para entidades frequentemente acessadas
- Consultas otimizadas com critérios específicos
- Carregamento lazy apenas quando necessário

### Manutenibilidade
- Código modularizado por funcionalidade
- Métodos específicos para cada tipo de cálculo
- Configurações centralizadas em campos livres (JSON)

### Extensibilidade
- Suporte a novos tipos de operação através de configuração
- Campos livres permitem customizações sem alteração de código
- Métodos de cálculo podem ser estendidos para novas regras

---

**Última Alteração:** 12/01/2026 às 16:30  
**Autor:** Bruno  
**Tipo:** Fórmula de Cálculo Fiscal  
**Versão:** 1.0