# SCV_F8_SCV2002

📖 **Descrição**  
Fórmula responsável por gerar uma lista paginada de documentos de SCV (Sistema de Controle de Vendas) de saída, com base em filtros e critérios de busca definidos pelo usuário. A fórmula recupera informações detalhadas sobre documentos, como tipo, número, status, data, valor, e dados da entidade e representante.

🎯 **Finalidade**  
Facilitar a consulta e exibição dos documentos de SCV de saída para controle e acompanhamento, permitindo a busca detalhada e a aplicação de filtros personalizados.

👥 **Público-Alvo**
- Departamento Comercial
- Departamento Financeiro
- Equipe de Vendas
- Suporte Operacional

📊 **Dados e Fontes**  
**Tabelas Principais:**
- `Eaa01` - Documentos SCV
- `Abb01` - Cabeçalhos de documentos
- `Aah01` - Tipos de documentos
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe0101` - Relacionamento de entidades
- `Abe30` - Condições de pagamento
- `Eaa0107` - Detalhes de documentos

**Entidades Envolvidas:**
- `Eaa01` - Dados do documento SCV
- `Abb01` - Cabeçalho do documento
- `Aah01` - Tipo de documento
- `Abe01` - Entidade do documento
- `Eaa0107` - Informações adicionais sobre documentos

⚙️ **Parâmetros da Fórmula**

| Parâmetro       | Tipo    | Obrigatório | Descrição                              |
|-----------------|---------|-------------|----------------------------------------|
| requisicao      | Objeto  | Sim         | Objeto contendo os filtros e parâmetros de busca |
| colunas         | Lista   | Sim         | Lista de colunas a serem exibidas na resposta do F8 |
| parametros      | Lista   | Sim         | Parâmetros utilizados nas consultas SQL |
| whereFiltros    | String  | Sim         | Filtros adicionais para limitar os resultados da busca |
| whereBusca      | String  | Sim         | Filtros para a busca personalizada |
| dados           | Lista   | Sim         | Dados obtidos da consulta ao banco para serem retornados no F8 |

🔄 **Fluxo do Processo**
1. **Preparação da Consulta SQL**
    - A fórmula recebe a requisição do F8, que contém filtros e parâmetros de busca.
    - A consulta SQL é montada utilizando os filtros fornecidos e a cláusula `WHERE`, considerando apenas documentos de SCV de saída.
2. **Execução da Busca**
    - A consulta SQL é executada no banco de dados para recuperar os documentos SCV.
    - A resposta é formatada para atender aos critérios definidos na fórmula, incluindo a ordenação por data e número de documento.
3. **Formatação da Resposta**
    - As colunas especificadas são formatadas e preenchidas com os dados obtidos da consulta.
    - A resposta do F8 é construída com as colunas e os dados recuperados.

⚠️ **Regras de Negócio**
- **Filtros de Documento SCV:**
    - Apenas documentos de SCV de saída (`eaa01clasDoc = 0` e `eaa01esMov = 1`) são considerados na consulta.
- **Ordenação dos Resultados:**
    - Os resultados são ordenados pela data do documento (`abb01data DESC`) e pelo número do documento (`abb01num DESC`).
- **Filtros de Busca e Filtro Personalizado:**
    - Filtros adicionais fornecidos pelo usuário via interface de busca são aplicados à consulta.
    - Filtros específicos são concatenados à consulta SQL com a cláusula `AND`.

🔧 **Métodos Principais**
- `obterTipoFormula()`  
  Retorna o tipo de fórmula `F8` para a execução do F8.
- `executar()`  
  Método principal que processa a requisição, monta a consulta SQL e retorna os dados formatados para o F8.

🔧 **Dependências**  
**Bibliotecas:**
- `br.com.multitec.utils.collections.TableMap` - Manipulação de dados e mapeamento de tabelas.
- `sam.server.samdev.utils.*` - Utilitários para manipulação de filtros e parâmetros de busca.
- `sam.dicdados.FormulaTipo` - Definição do tipo de fórmula a ser utilizada.

**Entidades:**
- `FormulaBase` - Classe base para fórmulas do sistema.
- `ColunaF8` - Definição de colunas que serão exibidas na resposta do F8.
- `RespostaDoF8` - Objeto que encapsula a resposta a ser retornada pelo F8.

📝 **Observações Técnicas**
- A fórmula permite realizar buscas detalhadas utilizando filtros como código do tipo de documento, número, data, status, entidade, entre outros.
- A busca é paginada para garantir que grandes volumes de dados não sobrecarreguem o sistema.
- O SQL gerado é dinâmico, permitindo a inclusão de filtros adicionais à consulta.

