# SCE_ControleEstoque.md
# SCE - Controle de Estoque

## 📖 Descrição
Relatório de controle de estoque que apresenta saldos, movimentações de entrada e saída, valores e informações detalhadas por item, lote e série. Suporta exportação em PDF e XLSX.

## 🎯 Finalidade
- Controlar saldos de estoque por item, lote e série
- Apresentar movimentações de entrada e saída
- Calcular valores totais em estoque
- Filtrar por itens, processos, unitizadores e locais de estoque
- Exportar resultados em PDF ou XLSX

## 👥 Público-Alvo
- Almoxarifado
- Controle de estoque
- Financeiro
- Auditoria

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Bcc01` - Movimentações de estoque
- `Bcc02` - Saldos de estoque
- `Bcc0201` - Detalhes de saldo (lote/série)
- `Abm01` - Itens/Produtos
- `Abm15` - Locais de estoque (controle 0)
- `Aam01` - Classes de itens
- `Aam04` - Status de estoque
- `Abb01` - Centrais de documento
- `Abm20` - Tipos de lançamento
- `Aba01` - Parâmetros do sistema

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| itens | List<Long> | Não | IDs dos itens para filtrar |
| local | List<Long> | Não | IDs dos locais de estoque (controle 0) |
| proc | String | Não | Processos/lotes (separados por ;) |
| uniti | String | Não | Unitizadores/séries (separados por ;) |
| data | LocalDate[] | Não | Período para filtro |
| imprimir | Integer | Sim | Formato de saída (0=PDF, 1=XLSX) |

## 🔄 Fluxo do Processo

### 1. **Recebimento e Processamento dos Parâmetros**
- Leitura e validação dos filtros
- Separação de strings (processos e unitizadores)
- Obtenção da data atual do estoque do parâmetro do sistema

### 2. **Busca de Dados Principais**
- **Saldo dos itens:** Quantidades totais em estoque
- **Lotes e séries:** Listas únicas baseadas nos filtros
- **Itens de entrada:** Detalhes das entradas com valores unitários

### 3. **Cálculos e Processamento**
- Cálculo de saídas por item/lote/série
- Cálculo de entradas por item/lote/série
- Cálculo do saldo atual (entradas - saídas)
- Remoção de registros com saldo zero
- Cálculo de valores totais (quantidade × valor unitário)

### 4. **Preparação da Estrutura de Dados**
- Criação de data sources principais e secundários
- Associação de sub-relatórios
- Carregamento de templates

### 5. **Geração do Relatório**
- Renderização em PDF ou XLSX conforme parâmetro
- Inclusão de sub-relatórios para detalhamento

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração do relatório.

**Fluxo:**
1. Processa parâmetros de entrada
2. Busca data atual do estoque do sistema
3. Obtém saldos dos itens
4. Busca lotes e séries
5. Obtém itens de entrada com detalhes
6. Calcula movimentações e saldos
7. Prepara estrutura de data sources
8. Gera relatório no formato solicitado

### `buscarEstoque()`
Busca saldos de estoque agrupados por item.

**Parâmetros:**
- `idItens`: Filtro por IDs de itens
- `lotes`: Filtro por lotes
- `series`: Filtro por séries
- `ctrl0`: Filtro por locais de estoque

**Retorno:** `List<TableMap>` com saldos agregados

### `buscarItensEntrada()**
Busca detalhes das entradas de estoque.

**Parâmetros:**
- `idItens`: Filtro por IDs de itens
- `procs`: Filtro por processos/lotes
- `units`: Filtro por unitizadores/séries
- `ctrl0`: Filtro por locais de estoque

**Retorno:** `List<TableMap>` com detalhes das entradas

### `buscarLctoSaida()` e `buscarLctoEntrada()`
Calculam totais de saídas e entradas por item/lote/série.

### `buscarConteudoParametro()`
Busca parâmetros do sistema (ex: data atual do estoque).

## 📊 Estrutura de Dados

### **Data Source Principal (Dados Empresa)**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| aac10rs | String | Razão social da empresa |
| key | Integer | Chave para associação (valor fixo: 1) |

### **Sub-Data Source 1 (Itens de Entrada)**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| abm01id | Long | ID do item |
| abm01na | String | Nome alternativo do item |
| abm01livre | String | Código livre do item |
| abm01descr | String | Descrição do item |
| numEntrada | Integer | Número do documento de entrada |
| bcc01qt | BigDecimal | Quantidade (após cálculo de saldo) |
| unit | BigDecimal | Valor unitário |
| abm15nome | String | Nome do local de estoque |
| bcc01lote | String | Lote |
| bcc01serie | String | Série |
| vlr | BigDecimal | Valor total (quantidade × valor unitário) |
| key | Integer | Chave para associação |

### **Sub-Data Source 2 (Saldos dos Itens)**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| abm01id | Long | ID do item |
| abm01livre | String | Código livre do item |
| abm01na | String | Nome alternativo do item |
| aam01descr | String | Descrição da classe do item |
| qtd_estoque | BigDecimal | Quantidade total em estoque |
| key | Integer | Chave para associação |

## ⚠️ Regras de Negócio

### **Filtros de Status**
- Apenas registros com status 44322667 (ativo) são considerados
- Exclusão de tipos de lançamento específicos ('406', '405', '602')

### **Cálculo de Saldo**
Saldo Atual = Total Entradas - Total Saídas

- Registros com saldo zero são removidos da listagem
- Valores unitários são obtidos do campo JSON `vlr_unit`

### **Formatação de Parâmetros**
- Processos: separados por `;`
- Unitizadores: separados por `;`
- Listas vazias ou nulas são tratadas como "todos"

### **Data do Estoque**
- Obtida do parâmetro do sistema `BC_DATAATUAL`
- Usada como referência para cálculos

### **Formato de Saída**
- **0 (PDF)**: Relatório formatado para impressão
- **1 (XLSX)**: Planilha Excel para análise

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e ColumnType
- `multitec.utils` - Utils, DateUtils, TableMap
- `sam.dicdados` - Parametros
- `sam.model` - Entidades do sistema
- `sam.server.samdev.relatorio` - Base para relatórios
- `sam.server.samdev.utils` - Parametro

**Arquivos:**
- `SCE_ControleEstoque.jrxml` - Template principal
- `SCE_ControleEstoque_S1.jrxml` - Sub-relatório 1 (itens entrada)
- `SCE_ControleEstoque_S2.jrxml` - Sub-relatório 2 (saldos)
