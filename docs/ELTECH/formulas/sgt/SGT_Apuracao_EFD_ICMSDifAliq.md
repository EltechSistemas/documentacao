# SGT_Apuracao_EFD_ICMSDifAliq.md

## 📖 Descrição
Sistema de apuração automática do ICMS Diferencial de Alíquota (DIFAL) e Fundo de Combate à Pobreza (FCP) para a EFD-ICMS/IPI no SGT (Sistema de Gestão Tributária).

## 🎯 Finalidade
Realizar a apuração mensal automática dos valores de ICMS DIFAL e FCP por estado, gerando os registros contábeis necessários para a escrituração fiscal digital.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria
- Gestão Tributária

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SGT_Apuracao_EFD_ICMSDifAliq` - Apuração de ICMS DIFAL/FCP

**Localização:** `eltech/formulas/sgt/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EDB01` - Apurações fiscais
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos fiscais
- `EAA01035` - Ajustes fiscais
- `ABB01` - Documentos fiscais
- `AAG02` - Estados/UFs
- `AAC10` - Empresas

**Entidades Envolvidas:**
- `Edb01` - Apuração fiscal
- `Eaa01` - Documento fiscal
- `Eaa0103` - Item de documento fiscal
- `Eaa01035` - Ajuste fiscal
- `Aac10` - Empresa
- `Aag02` - Estado/UF

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Período de apuração (ano/mês) |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| edb01 | Apuração fiscal gerada | Edb01 |
| edb01json | Campos de apuração DIFAL/FCP | TableMap |

## 🔄 Fluxo do Processo

1. **Inicialização**
   - Obtém período de apuração (EDB01)
   - Define datas inicial e final do mês
   - Seleciona alinhamento "0031" para estrutura de campos

2. **Processamento por Estado**
   - Identifica estados envolvidos (UF da empresa ou todos)
   - Verifica se é primeira apuração para cada estado
   - Determina se estado é da empresa ou destino

3. **Apuração do DIFAL**
   - Calcula saldo credor anterior
   - Processa saídas com débito
   - Aplica ajustes de débito e crédito
   - Calcula saldo devedor e valor a recolher

4. **Apuração do FCP**
   - Repete estrutura do DIFAL para Fundo de Combate à Pobreza
   - Calcula valores específicos do FCP

5. **Persistência dos Resultados**
   - Armazena valores nos campos JSON da apuração
   - Persiste apenas apurações com valores relevantes
   - Aplica where padrão do sistema

## ⚠️ Regras de Negócio

### Campos de Apuração DIFAL

| Campo | Descrição |
|-------|-----------|
| sdoCredAntDifal | Saldo credor anterior - DIFAL |
| debDifal | Saídas com débito - DIFAL |
| ajuDebDifal | Ajustes de débito - DIFAL |
| credDifal | Créditos do ICMS - DIFAL |
| ajuCredDifal | Ajustes de crédito - DIFAL |
| sdoDevDifal | Saldo devedor antes das deduções - DIFAL |
| deducoesDifal | Deduções - DIFAL |
| vlrRecolDifal | Valor recolhido ou a recolher - DIFAL |
| sdoCredDifal | Saldo credor a transportar - DIFAL |
| extraDifal | Valores extra-apuração - DIFAL |

### Campos de Apuração FCP

| Campo | Descrição |
|-------|-----------|
| sdoCredAntFcp | Saldo credor anterior - FCP |
| debFcp | Saídas com débito - FCP |
| ajuDebFcp | Ajustes de débito - FCP |
| credFcp | Entradas com crédito - FCP |
| ajuCredFcp | Ajustes de crédito - FCP |
| sdoDevFcp | Saldo devedor antes das deduções - FCP |
| deducoesFcp | Deduções - FCP |
| vlrRecolFcp | Valor recolhido ou a recolher - FCP |
| sdoCredFcp | Saldo credor a transportar - FCP |
| extraFcp | Valores extra-apuração - FCP |

### Validações e Regras
- Estados "EX" (Exterior) são ignorados
- Primeira apuração considera valores do primeiro período
- Campos com valor zero não geram apuração
- Aplicação de where padrão em todas as consultas
- Documentos cancelados são excluídos dos cálculos

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Critérios e consultas
- `multitec.utils` - Utilitários e coleções
- `sam.dicdados` - Definição de tipos de fórmula

**Consultas:**
- Saldo credor anterior por estado
- Total de imposto DIFAL/FCP por operação
- Ajustes de débito e crédito
- Valores extra-apuração
- Modelos de documentos por registro

## 📝 Observações Técnicas

### Modelos de Documentos
- **Registro C100**: 01, 1B, 04, 55, 65
- **Registro D100**: 07, 08, 8B, 09, 10, 11, 26, 27, 57, 67, 63

### Códigos de Ajuste
- **Terceiro dígito**: 2 (ICMS), 3 (FCP)
- **Quarto dígito**: 
  - 0,1 (Débito)
  - 2,3 (Crédito/Dedução)
  - 5 (Extra-apuração)

### Tratamento de Estados
- **UF da Empresa**: Considera campo vl_icms_uf_rem
- **UF Destino**: Considera campo vl_icms_uf_dest
- **FCP**: Sempre utiliza campo vl_fcp_uf_dest

## 🔄 Métodos Principais

### `executar()`
Método principal que orquestra toda a apuração.

### `buscarTotalImpostoDifalFcp()`
Calcula totais de imposto para DIFAL e FCP.

### `buscarAjustesDebitosICMSDifalFcp()`
Busca ajustes de débito para ICMS e FCP.

### `buscarAjustesCreditosICMSDifalFcp()`
Busca ajustes de crédito para ICMS e FCP.

### `buscarSaldoCredorAnterior()`
Obtém saldo credor do período anterior.

## 💡 Estrutura de Cálculo

**Saldo Devedor =** (Saídas com Débito + Ajustes Débito) - (Saldo Credor Anterior + Créditos + Ajustes Crédito)

**Valor a Recolher =** Saldo Devedor - Deduções

**Saldo Credor =** (Saldo Credor Anterior + Créditos + Ajustes Crédito) - (Saídas com Débito + Ajustes Débito)

## ⚠️ Validações Fiscais
- Apenas documentos com indicador EFD ICMS = 1
- Exclusão de documentos cancelados
- Filtro por situação fiscal válida para EFD
- Consideração apenas de endereços principais
- Validação de modelos de documento permitidos