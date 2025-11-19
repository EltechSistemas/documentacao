# CAS_ImportarFCIEab01011 - Importação de Ficha de Conteúdo de Importação

## 📖 Descrição
Classe responsável pela importação de dados de Ficha de Conteúdo de Importação (FCI) a partir de arquivos Excel (XLSX), realizando a inserção em lote na tabela `eab01011` do sistema.

## 🎯 Finalidade
Automatizar o processo de importação de dados de FCI, permitindo o carregamento em massa de informações de cálculo, itens, quantidades e valores através de planilhas Excel.

## 👥 Público-Alvo
- Departamento Fiscal
- Import/Export
- Contabilidade
- Administradores do sistema

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| arquivo | MultipartFile | Sim | Arquivo Excel (.xlsx) com dados FCI |

## 📋 Estrutura de Dados

### Tabela Destino: `eab01011`

| Campo | Tipo | Descrição |
|-------|------|-----------|
| eab01011id | Long | ID único do registro (sequencial) |
| eab01011calc | Long | Código do cálculo FCI |
| eab01011item | Long | Código do item |
| eab01011qtdComp | BigDecimal | Quantidade do componente |
| eab01011qtde | BigDecimal | Quantidade |
| eab01011valor | BigDecimal | Valor |
| eab01011vmu | BigDecimal | Valor por unidade de medida |

### Estrutura do Arquivo Excel:

| Coluna | Tipo | Descrição | Exemplo |
|--------|------|-----------|---------|
| A | Long | Código do cálculo FCI | 1001 |
| B | Long | Código do item | 2050 |
| C | BigDecimal | Quantidade do componente | 2.5 |
| D | BigDecimal | Quantidade | 10.0 |
| E | BigDecimal | Valor | 150.75 |
| F | BigDecimal | Valor por unidade | 15.075 |

## 🔄 Fluxo do Processo

### 1. **Leitura do Arquivo**
- Recebe arquivo Excel via MultipartFile
- Abre workbook e seleciona primeira planilha
- Itera sobre todas as linhas a partir da linha 1 (cabeçalho)

### 2. **Validação e Processamento**
- Ignora linhas vazias na coluna A
- Valida existência do registro Eab0101
- Converte valores das células para tipos apropriados

### 3. **Construção do SQL**
- Monta instrução INSERT em lote
- Utiliza sequence para geração de IDs
- Prepara statement com parâmetros dinâmicos

### 4. **Execução em Lote**
- Preenche parâmetros do PreparedStatement
- Executa inserção em massa
- Confirma transação automaticamente

## ⚠️ Regras de Negócio

### Validações:
- **Linha vazia**: Ignora se célula A estiver vazia
- **Registro existente**: Verifica se Eab0101 existe antes de importar
- **Tipos de dados**: Converte automaticamente para BigDecimal/Long

### Estrutura do Arquivo:
- **Cabeçalho**: Linha 0 é ignorada (considerada cabeçalho)
- **Dados efetivos**: Processa a partir da linha 1
- **Formato numérico**: Todas as colunas de B a F devem ser numéricas

### Processamento em Lote:
- Construção incremental do SQL com múltiplos VALUES
- Uso de PreparedStatement para segurança e performance
- Commit automático ao final da execução

## 🎨 Saídas Geradas

| Tipo | Descrição | Resultado |
|------|-----------|-----------|
| Banco | Inserção em `eab01011` | Registros importados |
| Sistema | Transação | Commit automático |

## 🔧 Dependências

### Bibliotecas:
- `Apache POI` - Manipulação de arquivos Excel
- `Spring Multipart` - Upload de arquivos
- `MultiORM` - Acesso a dados e criteria

### Entidades:
- `Eab0101` - Entidade de validação
- `Eab01011` - Tabela de destino

### Recursos do Sistema:
- Sequence: `default_sequence` para geração de IDs
- Conexão ativa com banco de dados

## 📝 Observações Técnicas

### Performance:
- Processamento em lote para melhor performance
- Uso de PreparedStatement para evitar SQL injection
- Stream processing do arquivo Excel

### Tratamento de Erros:
- Validação de existência do registro pai
- Conversão segura de tipos de dados
- Continua processamento mesmo com linhas inválidas

### Segurança:
- PreparedStatement previne SQL injection
- Validação de tipos antes da inserção
- Controle de transações automático

### Limitações:
- Processa apenas primeira planilha do workbook
- Assume estrutura fixa de colunas
- Requer cabeçalho na primeira linha