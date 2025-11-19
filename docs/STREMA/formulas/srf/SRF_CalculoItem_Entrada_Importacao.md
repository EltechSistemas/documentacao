# SRF_CalculoItem_Entrada_Importacao.md

## 📖 Descrição
Fórmula especializada para cálculo de itens em documentos fiscais de entrada de importação, com tratamento específico para diferentes modalidades de importação (PCD 114 e 314).

## 🎯 Finalidade
Calcular valores tributários, conversões monetárias e custos de aquisição em operações de importação, garantindo conformidade fiscal e precisão nos cálculos de impostos.

## 👥 Público-Alvo
- Departamento Fiscal
- Comércio Exterior
- Contabilidade
- Almoxarifado/Estoque

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_CalculoItem_Entrada_Importacao` - Cálculo de itens de importação

**Localização:** `strema/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens do documento
- `ABD01` - Tipos de documento (PCD)
- `ABM01` - Cadastro de itens
- `ABM0101` - Configurações do item por empresa
- `ABM12` - Dados fiscais do item

**Entidades Envolvidas:**
- `Eaa0103` - Item do documento
- `Eaa01` - Documento fiscal
- `Abd01` - Tipo de documento (PCD)
- `Abm01` - Item
- `Abm0101` - Configuração empresa-item

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento a ser calculado |

## 🔄 Fluxo do Processo

1. **Inicialização e Validações**
   - Obtém item do documento (EAA0103)
   - Carrega documento pai (EAA01) e central (ABB01)
   - Recupera dados da PCD, entidade e empresa
   - Valida endereço principal da entidade

2. **Carregamento de Dados Complementares**
   - Endereços da entidade e empresa
   - Configurações do item (fiscal, comercial, valores)
   - Dados tributários (CFOP, NCM, CSTs)
   - Campos livres (JSON) de diversas entidades

3. **Processamento por Modalidade de Importação**
   - **PCD 114:** Cálculo simples baseado em quantidade × valor unitário
   - **PCD 314:** Cálculo complexo com conversão monetária e tributos específicos

4. **Cálculos Tributários**
   - Imposto de Importação (II)
   - IPI, PIS, COFINS, ICMS
   - Conversões de quantidades e unidades
   - Cálculo de custo de aquisição

## ⚠️ Regras de Negócio

### Validações Críticas
- Item deve existir no documento
- Endereço principal da entidade é obrigatório
- Configuração fiscal do item deve existir
- Para PCD 314, cotação monetária é obrigatória

### Tratamento por Tipo de Documento (PCD)
- **PCD 114:** Cálculo direto sem conversão monetária
- **PCD 314:** Cálculo com conversão para Real e tributos específicos

### Cálculos Tributários
- **II:** Base de cálculo no valor CIF, zerado quando alíquota zero
- **IPI:** Base = CIF + II, com isenção para alíquota zero
- **PIS/COFINS:** Tratamento diferenciado para operações com exterior (UF "EX")
- **ICMS:** Cálculo complexo com ajuste de base de cálculo e tratamento para CST 51

### Conversões Monetárias (PCD 314)
- Utiliza data de cotação específica ou última cotação disponível
- Converte valores FOB, frete e seguro para Real
- Valida existência de cotação para a data informada

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
- **Tratamento de Erros:** Validações com `ValidacaoException` e `interromper()` para dados inconsistentes
- **Performance:** Utiliza carregamento lazy de entidades relacionadas
- **Flexibilidade:** Campos livres (JSON) permitem extensibilidade sem alteração de schema

### Especificidades por PCD
- **PCD 114:** Foco em importações simplificadas
- **PCD 314:** Processamento completo com conversão monetária e todos os tributos

### Campos Livres (JSON) Utilizados
- `fob_imp_real`: Valor FOB convertido para Real
- `vlr_frete_real`, `vlr_seguro_real`: Despesas convertidas
- `cif_imp`: Valor CIF da importação
- `custo_aquisicao`: Custo total de aquisição
- `vlr_unit_sem`, `qtde_doc`: Dados básicos do item

### Tratamento de Tributos
- Ajuste automático de base de cálculo do ICMS
- Isenção de IPI quando alíquota zero no NCM
- Zeramento de ICMS para CST terminado em "51"
- Cálculo de PIS/COFINS com alíquotas específicas por UF