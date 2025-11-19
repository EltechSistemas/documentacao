# SRF_CalculoItem_Entrada_ImportacaoDrawback.md

## 📖 Descrição
Fórmula para cálculo de itens em documentos fiscais de entrada com regime especial de Drawback, responsável pelo processamento de tributos e valores em importações.

## 🎯 Finalidade
Calcular automaticamente os valores tributários, quantidades e custos de itens em documentos de entrada com regime de Drawback, garantindo conformidade fiscal e precisão nos cálculos.

## 👥 Público-Alvo
- Departamento Fiscal
- Comércio Exterior
- Contabilidade
- Almoxarifado/Estoque

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_CalculoItem_Entrada_ImportacaoDrawback` - Cálculo de itens com Drawback

**Localização:** `strema/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens do documento
- `ABM01` - Cadastro de itens
- `ABM0101` - Configurações do item por empresa
- `ABM12` - Dados fiscais do item
- `ABM13` - Dados comerciais do item

**Entidades Envolvidas:**
- `Eaa0103` - Item do documento
- `Abm01` - Item
- `Abm0101` - Configuração empresa-item
- `Abm12` - Configuração fiscal
- `Abm13` - Configuração comercial

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento a ser calculado |

## 🔄 Fluxo do Processo

1. **Inicialização e Validações**
   - Obtém item do documento (EAA0103)
   - Carrega documento pai (EAA01) e central (ABB01)
   - Recupera dados da PCD, entidade e empresa

2. **Carregamento de Dados Complementares**
   - Endereços da entidade e empresa
   - Configurações do item (fiscal, comercial, valores)
   - Dados tributários (CFOP, NCM, CSTs)
   - Campos livres (JSON) de diversas entidades

3. **Cálculos Principais**
   - Valores do item (total, descontos)
   - Tributos (II, IPI, PIS, COFINS, ICMS)
   - Quantidades e conversões de unidades
   - Pesos e volumes
   - Custo de aquisição para importação

4. **Processamento Específico Drawback**
   - Cálculo de base de custo (CIF)
   - Tratamento para operações com exterior
   - Ajustes fiscais específicos

## ⚠️ Regras de Negócio

### Validações Críticas
- Item deve existir no documento
- Endereço principal da entidade é obrigatório
- Configuração fiscal do item deve existir
- Dados da empresa e entidade devem estar completos

### Cálculos Tributários
- **IPI:** Calculado com alíquota do NCM, com isenção para alíquota zero
- **ICMS:** Cálculo com base na alíquota e ajuste de base de cálculo
- **PIS/COFINS:** Tratamento especial para operações com exterior
- **II:** Utiliza alíquota específica do item

### Conversões e Ajustes
- Conversão de quantidade comercial para estoque
- Cálculo de peso bruto e líquido
- Ajuste de volumes quando aplicável
- Cálculo de custo de aquisição considerando todos os tributos

### Campos Livres (JSON)
- `vlrfcientrada`: Valor fiscal de entrada
- `ii_aliq`: Alíquota do Imposto de Importação
- `vlr_vlme`: Valor do volume
- `ipi_aliq`, `ipi_bc`, `ipi_ipi`: Dados do IPI
- `custo_aquisicao`: Custo total de aquisição
- `cif_imp`: Valor CIF da importação

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| eaa0103 | Item do documento com cálculos atualizados | Eaa0103 |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `sam.dicdados` - Definições de tipos de fórmula
- `sam.model.entities` - Entidades do sistema

**Entidades Relacionadas:**
- Todas entidades AA, AB, EA do pacote `sam.model.entities`

## 📝 Observações Técnicas

- **Processamento:** Síncrono, executado para cada item do documento
- **Tratamento de Erros:** Validações com `ValidacaoException` para dados inconsistentes
- **Performance:** Utiliza carregamento lazy de entidades relacionadas
- **Flexibilidade:** Campos livres (JSON) permitem extensibilidade sem alteração de schema
- **Especificidade:** Foco em operações de importação com regime Drawback
- **Arredondamentos:** Aplicados em valores monetários (2 casas) e quantidades (4 casas)

### Estrutura de Cálculo
- Separação clara entre carregamento de dados e cálculos
- Reutilização de fatores de conversão comerciais
- Consideração de múltiplos endereços na entidade
- Suporte a diferentes unidades de medida

### Tributação
- Tratamento diferenciado para operações com exterior (UF "EX")
- Cálculo de base de ICMS com ajuste de divisor
- Isenção automática de IPI quando alíquota zero
- Acumulação de despesas acessórias