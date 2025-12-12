# SCE - Importar Itens de Lançamentos XLSX

## 📖 Descrição
Fórmula para importação de itens de lançamentos de estoque a partir de arquivo Excel (XLSX), permitindo carga em massa de movimentações de estoque com dados como tipo de item, código, quantidade, valor, lote, série, validade e fabricação.

## 🎯 Finalidade
Facilitar a importação de múltiplos itens de lançamentos de estoque através de planilha Excel, validando os dados e convertendo-os para o formato DTO do sistema.

## 👥 Público-Alvo
- Departamento de Estoque
- Almoxarifado
- Logística
- Controle de Inventário

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abm01` - Itens cadastrais
- `Aba20` - Repositório de avisos
- `Aba2001` - Itens do repositório de avisos

**Estrutura do Arquivo Excel (XLSX):**
| Coluna | Campo | Tipo | Descrição |
|--------|-------|------|-----------|
| A | Tipo do Item | Integer | Tipo do item (referência a abm01tipo) |
| B | Código do Item | String | Código do item (referência a abm01codigo) |
| C | Quantidade | BigDecimal | Quantidade a ser lançada |
| D | Valor | BigDecimal | Valor unitário do item |
| E | Lote | String | Número do lote (opcional) |
| F | Série | String | Número de série (opcional) |
| G | Validade | LocalDate | Data de validade (opcional) |
| H | Fabricação | LocalDate | Data de fabricação (opcional) |

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| arquivo | MultipartFile | Sim | Arquivo Excel (XLSX) com os dados dos itens |

## 🔄 Fluxo do Processo

### 1. **Leitura do Arquivo Excel**
- Abertura do arquivo XLSX como InputStream
- Criação do workbook Apache POI
- Seleção da primeira planilha (sheet)

### 2. **Processamento das Linhas**
- Iteração a partir da linha 2 (ignora cabeçalho)
- Para cada linha com dados na coluna A:
  - Leitura dos valores das células
  - Busca do item no sistema pelo tipo e código
  - Conversão dos dados para DTO
  - Coleta de erros em caso de exceções

### 3. **Validação e Conversão**
- Validação da existência do item no cadastro (Abm01)
- Conversão de tipos (numérico para BigDecimal, data para LocalDate)
- Tratamento de valores opcionais (lote, série, datas)

### 4. **Registro de Avisos**
- Busca do repositório "Avisos Itens Lctos" (Aba20)
- Criação de itens de aviso para cada mensagem de erro
- Persistência dos avisos no banco de dados

### 5. **Retorno dos Dados**
- Lista de itens convertidos para DTO
- DTOs prontos para processamento em lote

## ⚠️ Regras de Negócio

### Validações
- O item deve existir no cadastro (Abm01) com tipo e código correspondentes
- Campos obrigatórios: tipo, código, quantidade e valor
- Campos opcionais: lote, série, validade, fabricação
- Em caso de erro, o processamento continua com as demais linhas

### Formato do Arquivo
- Extensão: .xlsx (Excel)
- Formato de datas: conforme configuração do Excel
- Cabeçalho na primeira linha (ignorada no processamento)
- Suporte a caracteres especiais em strings

### Tratamento de Erros
- Erros por item são coletados em lista de mensagens
- Mensagens são registradas no repositório "Avisos Itens Lctos"
- Processamento continua mesmo com erros em linhas específicas
- Exceções são capturadas e registradas com identificação do item

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de importação.

### Processos Internos
- Leitura do arquivo Excel com Apache POI
- Conversão de células para tipos Java apropriados
- Busca de itens no banco de dados
- Criação de DTOs `ItemSCEDto`
- Registro de avisos em repositório específico

## 📊 Estrutura de Saída

**ItemSCEDto:**
- `abm01id` - ID do item encontrado
- `quantidade` - Quantidade do item
- `valor` - Valor unitário
- `lote` - Número do lote (opcional)
- `serie` - Número de série (opcional)
- `validade` - Data de validade (opcional)
- `fabricacao` - Data de fabricação (opcional)

**Parâmetros de Retorno:**
- `itensLctos`: Lista de `ItemSCEDto` com os itens importados

**Registro de Avisos:**
- Itens com erros são registrados no repositório "Avisos Itens Lctos" (Aba20)
- Cada aviso contém a mensagem de erro e índice do item

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `apache.poi` - Manipulação de arquivos Excel (XSSFWorkbook)
- `commons.io` - Utilitários de I/O (FileUtils)
- `springframework` - MultipartFile para upload
- `sam.dto.sce` - DTOs do módulo SCE
- `sam.model` - Entidades do sistema

**Módulo:** SCE (Sistema de Controle de Estoque)

## 📝 Observações Técnicas

### Tratamento de Arquivos
- Suporte apenas a formato XLSX (Excel 2007+)
- Uso de Apache POI para manipulação eficiente
- Stream processing para arquivos grandes

### Performance
- Processamento linha a linha para evitar carga de memória
- Transação única para persistência dos avisos
- Busca de itens em lote otimizada

### Segurança
- Validação de tipos de dados
- Tratamento de valores nulos
- Captura de exceções para evitar falha total

### Formato de Datas
- Conversão automática de datas do Excel para LocalDate
- Suporte a células vazias em datas opcionais
- Formato conforme configuração regional do Excel

---

**Última Alteração:** 11/12/2025 às 13:45  
**Autor:** Bruno  
**Tipo:** Fórmula de Importação de Itens de Lançamentos  
**Versão:** 1.0