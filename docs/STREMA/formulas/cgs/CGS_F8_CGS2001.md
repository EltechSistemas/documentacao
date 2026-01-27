# CGS_F8_CGS2001

📖 **Descrição**  
Fórmula para realizar a busca de informações de entidades (clientes/fornecedores) no sistema, com base em filtros e parâmetros definidos pelo usuário na tela de consulta. A fórmula retorna os dados organizados por código da entidade e seus respectivos detalhes.

🎯 **Finalidade**  
Automatizar a consulta de entidades com base em filtros e parâmetros fornecidos, otimizando a busca e a apresentação dos dados para o usuário, com suporte a paginação e filtros avançados.

👥 **Público-Alvo**
- Departamento de Cadastro
- Equipe de Suporte ao Sistema
- Usuários que precisam consultar entidades no sistema

📊 **Dados e Fontes**  
**Tabelas Principais:**
- `abe01` - Entidades (clientes/fornecedores)
- `abe02` - Relacionamento entre entidades e representantes
- `rep` - Representantes

**Entidades Envolvidas:**
- `abe01` - Entidade (cliente/fornecedor)
- `abe02` - Relacionamento de entidades
- `rep` - Representante

⚙️ **Parâmetros da Fórmula**

| Parâmetro       | Tipo    | Obrigatório | Descrição                          |
|-----------------|---------|-------------|------------------------------------|
| requisicao      | Objeto  | Sim         | Objeto contendo os filtros e buscas |
| filtros         | Lista   | Não         | Filtros adicionais para consulta   |
| buscas          | Lista   | Não         | Filtros de busca baseados no campo de pesquisa |
| pagina          | Inteiro | Sim         | Número da página de resultados     |
| tamanhoDaPagina | Inteiro | Sim         | Quantidade de itens por página    |

🔄 **Fluxo do Processo**
1. **Recebimento de Parâmetros**  
   A fórmula começa recebendo um objeto `RequisicaoDoF8`, que contém filtros e parâmetros definidos pelo usuário.
2. **Construção de WHERE para Filtros**  
   A partir dos filtros fornecidos pelo usuário, a fórmula monta o bloco `WHERE` da consulta SQL.
3. **Construção de WHERE para Busca**  
   Similar aos filtros, mas usando os parâmetros de busca definidos no campo de pesquisa.
4. **Execução da Consulta SQL**  
   A fórmula monta uma consulta SQL que busca as entidades e seus dados associados. A consulta é construída com base nos filtros e parâmetros definidos, e a consulta é otimizada para o tamanho da página.
5. **Processamento dos Dados**  
   Os dados recuperados são processados e organizados para serem apresentados ao usuário.

⚠️ **Regras de Negócio**

- **Validações:**
    - A consulta só é executada se a requisição contiver parâmetros de filtro ou busca válidos.
    - A paginação é aplicada ao resultado, retornando apenas a quantidade de registros definidos para a página.

- **Formatação de Dados:**
    - Dados de tipo `String` são retornados diretamente como estão.
    - As colunas de dados incluem `abe01codigo`, `abe01na`, `abe01nome`, `abe01ni`, e `rep0` (representante).

🔧 **Métodos Principais**
- `obterTipoFormula()`  
  Retorna o tipo de fórmula `F8`, conforme esperado pelo sistema.

- `executar()`  
  Método principal que orquestra todo o processo de busca e formatação dos dados. Responsável por montar a consulta SQL, executar a busca e formatar a resposta.

🔧 **Dependências**  
**Bibliotecas:**
- `sam.dicdados` - Tipos de fórmula e atributos de entidades
- `sam.server.samdev` - Utilitários de consulta e manipulação de dados
- `java.util.stream.Collectors` - Utilitário para transformação e filtragem de listas

**Entidades:**
- `FormulaBase` - Classe base para fórmulas
- `Parametro` - Parâmetros de consulta ao banco

📝 **Observações Técnicas**
- O método de construção do SQL utiliza `stream()` e `Collectors` para gerar dinamicamente o `WHERE` conforme os filtros e buscas fornecidos.
- A resposta é processada em formato tabular com colunas e dados organizados para fácil leitura.
- Suporte à paginação, garantindo que apenas os dados da página solicitada sejam retornados.
