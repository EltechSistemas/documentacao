# SRF_CalculoItem_Entrada_Cte.md

## 📖 Descrição
Fórmula especializada para cálculo de itens em documentos fiscais de entrada com foco em operações de Conhecimento de Transporte Eletrônico (CT-e), incluindo tratamento completo de tributos e ajustes fiscais.

## 🎯 Finalidade
Calcular valores tributários, conversões de unidades, custos de aquisição e tratamentos fiscais específicos para operações de entrada via CT-e, garantindo conformidade fiscal e precisão nos cálculos.

## 👥 Público-Alvo
- Departamento Fiscal
- Logística/Transportes
- Contabilidade
- Almoxarifado/Estoque

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_CalculoItem_Entrada_Cte` - Cálculo de itens de entrada via CT-e

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
   - Obtém item do documento (EAA0103) e documento pai (EAA01)
   - Carrega dados da central (ABB01) e PCD
   - Valida endereço principal da entidade
   - Verifica se documento é de entrada

2. **Carregamento de Dados Complementares**
   - Endereços da entidade e empresa
   - Configurações do item (fiscal, comercial, valores)
   - Dados tributários (CFOP, NCM, CSTs)
   - Tabela de preços e fatores de conversão

3. **Cálculos Principais**
   - Valores do item e totais
   - Conversões de quantidades e unidades
   - Tributos (ICMS, IPI, PIS, COFINS)
   - Ajustes de CFOP automáticos
   - Cálculo de custo de aquisição

4. **Tratamentos Específicos CT-e**
   - Diferencial de alíquota para operações interestaduais
   - Ajustes fiscais específicos para transporte
   - Observações fiscais automáticas

## ⚠️ Regras de Negócio

### Validações Críticas
- Item deve existir no documento
- Documento deve ser de entrada (PCD com abd01es ≠ 1)
- Endereço principal da entidade é obrigatório
- Configuração fiscal do item deve existir
- Fator de conversão de unidade é obrigatório

### Ajustes Automáticos de CFOP
- **Mercadoria para Revenda (00):** CFOP 1/2102 ou 1/2403 com IVA
- **Matéria-Prima/Embalagem (01/02):** CFOP 1/2101 ou 1/2401 com IVA
- **Material de Uso e Consumo (07):** CFOP 1/2556 com ajustes específicos
- **Operações com ST:** CFOP 1/2403 com tratamentos diferenciados

### Cálculos Tributários
- **ICMS:** Tratamento completo por CST (00, 10, 20, 30, 40, 50, 51, 60, 61, 70, 90)
- **IPI:** Cálculo com base no NCM, com observações fiscais específicas
- **PIS/COFINS:** Tratamento por CST com bases de cálculo ajustadas
- **Diferencial de Alíquota:** Aplicado para operações interestaduais de uso/consumo

### Conversões e Ajustes
- Conversão de quantidade comercial para estoque
- Cálculo automático de valores unitários da tabela de preços
- Ajustes de redução de base de cálculo
- Tratamento de IVA para substituição tributária

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
- **Validações:** Utiliza `ValidacaoException` para dados inconsistentes
- **Flexibilidade:** Campos livres (JSON) permitem extensibilidade
- **Performance:** Carregamento otimizado de entidades relacionadas

### Tratamentos Específicos
- **Simples Nacional:** Ajustes automáticos de CST e alíquotas
- **Operações com ST:** Cálculo completo de IVA e base de cálculo
- **Observações Fiscais:** Geração automática para IPI e diferencial de alíquota
- **CFOP Dinâmico:** Ajuste automático baseado no tipo de item e operação

### Campos Livres (JSON) Utilizados
- `unit_tabela`: Valor unitário da tabela de preços
- `ipi_obs`: Observação fiscal de IPI
- `custo_simples`: Percentual de custo para Simples Nacional
- `custo_aquisicao`: Custo total de aquisição
- `bc_difaliq`, `valor_difal`: Dados do diferencial de alíquota