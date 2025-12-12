# SCE - Saldo por Local (LCB)

## 📖 Descrição
Relatório de saldo de estoque agrupado por localização, desenvolvido especificamente para a LCB, permitindo visualizar a quantidade de itens em cada local com filtros por itens, classes, status, lotes, séries e validade.

## 🎯 Finalidade
Gerar relatório de saldo de estoque por local para controle de inventário, identificação de locais sem saldo e análise da distribuição de produtos no armazém.

## 👥 Público-Alvo
- Departamento de Estoque
- Almoxarifado
- Logística
- Controle de Inventário
- Gerência de Operações

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abm15` - Locais de estoque
- `Bcc02` - Saldos de estoque
- `Bcc0201` - Detalhes de saldos (lotes/séries)
- `Abm01` - Itens cadastrais
- `Aam01` - Classes de itens
- `Aam04` - Status de estoque
- `Aam06` - Unidades de medida
- `Abm0101` - Configuração de itens

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| itens | List<Long> | Não | Filtro por IDs dos itens |
| classes | List<Long> | Não | Filtro por IDs das classes |
| status | List<Long> | Não | Filtro por IDs dos status |
| local | List<String> | Não | Filtro por nome do local (início do nome) |
| lote | String | Não | Filtro por número de lote (separado por ";") |
| serie | String | Não | Filtro por número de série (separado por ";") |
| validade | LocalDate[] | Não | Intervalo de validade [início, fim] |
| saldoItem | Boolean | Não | Opção de exibir saldo por item |
| exibirSaldo | Boolean | Não | Opção de exibir saldo |
| localSemSaldo | Boolean | Não | Opção de exibir apenas locais sem saldo |
| imprimir | Integer | Sim | Formato de saída (0=PDF, 1=XLSX) |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Carregamento dos filtros padrão
- Busca da lista de locais disponíveis
- Configuração da empresa ativa

### 2. **Processamento dos Filtros**
- Conversão de parâmetros de entrada
- Tratamento de listas de lotes e séries (separados por ";")
- Validação de intervalos de datas

### 3. **Busca de Dados**
- **Opção normal**: Busca de saldos por local com filtros aplicados
- **Opção locais sem saldo**: Busca de locais que não possuem nenhum item em estoque

### 4. **Geração do Relatório**
- Formato PDF ou XLSX conforme seleção
- Adição de parâmetros da empresa
- Formatação dos dados conforme opções selecionadas

## ⚠️ Regras de Negócio

### Filtros de Busca
- **Local**: Filtro por início do nome do local (LIKE 'local%')
- **Lotes/Séries**: Múltiplos valores separados por ponto e vírgula
- **Validade**: Intervalo inclusivo (BETWEEN)
- **Locais sem saldo**: Exibe apenas locais que não possuem itens em estoque

### Formatação de Dados
- **Quantidade em UA**: Cálculo de quantidade em unidade auxiliar (bcc0201qt / abm0101fcua)
- **Nome do local**: Truncado em 6 caracteres para agrupamento
- **Ordenação**: Por nome do local, código do item e lote

### Parâmetros do Relatório
- **SALDOITEM**: Controla exibição de saldo por item
- **EXIBIRSALDO**: Controla exibição de coluna de saldo
- **EMPRESA**: Nome da empresa ativa no cabeçalho

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra a geração do relatório.

### `buscarDados()`
Busca os saldos de estoque por local com aplicação de filtros.

### `buscarIdsLocais()`
Busca IDs dos locais que possuem saldo em estoque.

### `buscarLocaisSemSaldo()`
Busca locais que não possuem nenhum item em estoque.

### `buscarLocais()`
Busca lista de locais disponíveis para preenchimento do filtro.

## 📊 Estrutura de Saída

**Campos do Relatório:**
- `abm15nome` - Nome do local (truncado em 6 caracteres)
- `abm01codigo` - Código do item
- `abm01descr` - Descrição do item
- `bcc0201lote` - Número do lote
- `bcc0201serie` - Número de série
- `bcc0201qt` - Quantidade em estoque
- `bcc0201validade` - Data de validade
- `aam06codigo` - Unidade de medida de uso
- `fcua` - Quantidade em unidade auxiliar (quando aplicável)

**Formato de Saída:**
- **PDF**: Relatório formatado para impressão/visualização
- **XLSX**: Planilha Excel para análise e manipulação

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários (Utils, TableMap)
- `sam.server.samdev.relatorio` - Base para relatórios
- `sam.server.samdev.utils` - Utilitários do SAM (Parametro)
- `java.time` - Manipulação de datas
- `java.util.stream` - Processamento de collections

**Módulo:** SCE (Sistema de Controle de Estoque)

## 📝 Observações Técnicas

### Performance
- Consultas otimizadas com índices apropriados
- Filtros aplicados no banco de dados para reduzir transferência
- Agrupamento por local otimizado para grandes volumes

### Formato de Locais
- Nomes de locais truncados em 6 caracteres para agrupamento
- Suporte a múltiplos locais em filtro
- Locais inativos (com data de inativação) são excluídos

### Filtros Especiais
- **Lotes/Séries**: Suporte a múltiplos valores com separador ";"
- **Local**: Busca por início do nome (otimizado com índice)
- **Validade**: Intervalo flexível com tratamento de nulos

### Segurança
- Aplicação de where padrão do sistema para controle de acesso
- Validação de parâmetros de entrada
- Tratamento de valores nulos e vazios

---

**Última Alteração:** 10/12/2025 às 14:44  
**Autor:** NAGYLA  
**Tipo:** Relatório de Saldo por Local  
**Versão:** 1.0 (Customizado para LCB)