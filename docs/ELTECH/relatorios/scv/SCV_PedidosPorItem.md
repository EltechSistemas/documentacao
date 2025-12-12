# SCV - Pedidos por Item (Eltech)

## 📖 Descrição
Relatório que consolida informações de pedidos agrupados por item, permitindo análise de quantidades solicitadas versus quantidades atendidas, com filtros flexíveis para diferentes tipos de pedidos e situações.

## 🎯 Finalidade
Fornecer uma visão consolidada dos pedidos por item, facilitando o acompanhamento de atendimento, identificação de saldos pendentes e análise de movimentação de produtos.

## 👥 Público-Alvo
- Comercial/Vendas
- Compras
- Planejamento
- Estoques
- Gerência

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Abb01` - Central de documento
- `Eaa0103` - Itens do documento fiscal
- `Eaa01032` - Itens atendidos do SCV
- `Abm01` - Produtos
- `Aam01` - Classes de produtos
- `Abe01` - Entidades (clientes/fornecedores)
- `Aam06` - Unidades de medida (comercial e uso)

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valor Padrão |
|-----------|------|-------------|-----------|--------------|
| tipos | List<Long> | Não | Tipos de documento a filtrar | - |
| numeroInicial | Integer | Não | Número inicial do documento | 0 |
| numeroFinal | Integer | Não | Número final do documento | 0 |
| entidades | List<Long> | Não | IDs das entidades (clientes/fornecedores) | - |
| emissao | LocalDate[] | Não | Período de emissão dos documentos | - |
| pedEntSai | Integer | Sim | Tipo de movimento (0=Compra, 1=Venda) | 1 |
| impressao | Integer | Sim | Formato de saída (0=PDF, 1=XLSX) | 0 |
| entrega | LocalDate[] | Não | Período de entrega dos itens | - |
| numeroCliente | String | Não | Número do pedido do cliente | - |
| isDesconsiderarSaldoZerado | Boolean | Sim | Desconsiderar itens com saldo zero | true |
| classes | List<Long> | Não | Classes de produtos | - |
| itens | List<Long> | Não | IDs dos itens específicos | - |
| status1 | Boolean | Sim | Incluir status 0 (pendente) | true |
| status2 | Boolean | Sim | Incluir status 1 (parcial) | true |
| status3 | Boolean | Sim | Incluir status 2 (atendido) | false |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Definição dos valores padrão dos filtros
- Carregamento dos parâmetros fornecidos pelo usuário
- Validação e formatação dos dados de entrada

### 2. **Busca de Documentos**
- Consulta ao banco de dados com múltiplos filtros aplicáveis
- Junção de tabelas relacionadas (documentos, itens, entidades, produtos)
- Aplicação automática de filtros de segurança (where padrão)

### 3. **Processamento de Dados**
- Agrupamento de itens por código de produto
- Soma de quantidades comercializadas por item
- Remoção de duplicidades na consolidação
- Cálculo de saldos (quantidade solicitada vs. atendida)

### 4. **Filtragem Final**
- Opção de desconsiderar itens com saldo zero
- Validação de diferenças entre quantidades
- Remoção de registros conforme critérios definidos

### 5. **Geração de Saída**
- Formatação dos dados para o formato escolhido (PDF ou XLSX)
- Adição de parâmetros de cabeçalho (empresa, título, datas)
- Criação do arquivo para download

## ⚠️ Regras de Negócio

### Filtros de Documentos
- **Documentos cancelados são excluídos automaticamente**
- Apenas documentos com classificação 0 são considerados
- Filtros de segurança são aplicados automaticamente
- Períodos de data são inclusivos (between)

### Tipos de Movimento
- **0 - Pedidos de Compra**: Movimento de entrada (esmov = 0)
- **1 - Pedidos de Venda**: Movimento de saída (esmov = 1)

### Status de Atendimento
- **0 - Pendente**: Nenhum item atendido
- **1 - Parcial**: Parte dos itens atendidos
- **2 - Atendido**: Todos os itens atendidos
- Filtro múltiplo permite combinação de status

### Consolidação por Item
- Itens com mesmo código são agrupados
- Quantidades comercializadas são somadas
- Primeira ocorrência mantém outros dados (entidade, datas)
- Ordenação por número do documento, código da entidade e código do item

### Tratamento de Saldos
- Saldo = eaa0103qtcoml (solicitado) - eaa01032qtcoml (atendido)
- Opção de excluir itens com saldo zero
- Validação de diferença exata (compareTo BigDecimal.ZERO)

## 🔧 Métodos Principais

### `getNomeTarefa()`
Retorna o nome da tarefa para identificação no sistema.

### `criarValoresIniciais()`
Define os valores padrão para os filtros do relatório.

### `executar()`
Método principal que orquestra todo o processo do relatório.

### `buscarDocumentos(...)`
Executa a consulta SQL com todos os filtros aplicados e retorna os dados brutos.

## 📊 Estrutura de Saída

**Colunas do Relatório:**
- `abb01num` - Número do documento
- `abb01data` - Data de emissão
- `abb01serie` - Série do documento
- `abe01codigo` - Código da entidade
- `abe01na` - Nome/Apelido da entidade
- `abm01codigo` - Código do produto
- `abm01na` - Nome/Apelido do produto
- `eaa0103dtentrega` - Data de entrega do item
- `eaa01032qtComl` - Quantidade atendida (consolidada)
- `eaa0103qtcoml` - Quantidade solicitada
- `aam06descr_uso` - Descrição da unidade de uso

**Formato de Saída:**
- **PDF**: Relatório formatado para impressão/visualização
- **XLSX**: Planilha Excel para análise e manipulação

**Parâmetros do Cabeçalho:**
- Código e razão social da empresa
- Título do relatório (Pedidos de Compra/Venda)
- Período de emissão filtrado

## 🔧 Dependências

**Bibliotecas:**
- `br.com.multitec.utils` - Utilitários gerais
- `br.com.multitec.utils.collections.TableMap` - Estrutura de dados
- `sam.server.samdev.relatorio` - Classes base para relatórios
- `java.time` - Manipulação de datas
- `java.math` - Operações com BigDecimal

**Módulo:** SCV (Sistema de Controle de Vendas)

## 📝 Observações Técnicas

### Performance da Consulta
- Uso de LEFT JOIN para garantir todos os dados necessários
- Índices sugeridos nas colunas de filtro frequente
- Filtragem aplicada diretamente no banco para eficiência
- Paginação implícita por limitação de período

### Tratamento de Dados
- Conversão segura de parâmetros para evitar NullPointerException
- Uso de BigDecimal para precisão em cálculos monetários
- Formatação de datas no padrão brasileiro (dd/MM/yyyy)
- Remoção de registros nulos das listas de status

### Segurança
- Aplicação automática de where padrão por empresa/usuário
- Validação de permissões através da classe base
- Sanitização de parâmetros de entrada

### Algoritmo de Consolidação
- Loop while com controle manual do índice para remoção segura
- Comparação de código do produto para identificação de duplicatas
- Acumulação progressiva de quantidades
- Atualização in-place dos registros consolidados

---

**Última Alteração:** 28/11/2025 às 20:49  
**Autor:** Bruno  
**Tipo:** Relatório de Pedidos por Item  
**Versão:** 1.0