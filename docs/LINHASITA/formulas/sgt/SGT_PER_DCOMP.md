# SGT_PER_DCOMP – Exportação de Dados de Apuração de IPI

## 📖 Descrição
Fórmula para exportação de dados de IPI (Imposto sobre Produtos Industrializados) conforme layout exigido pela Secretaria da Receita Federal (SRF). Gera registros R11, R12, R13 e R21 para fins de apuração e escrituração fiscal.

## 🎯 Finalidade
Exportar dados de IPI de entradas, saídas, notas fiscais e livro de apuração em formato textual padronizado para integração com sistemas fiscais externos.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` – Documentos fiscais
- `Eaa0102` – Dados complementares do documento
- `Eaa0103` – Itens do documento fiscal
- `Abb01` – Documentos centralizadores
- `Aaj15` – CFOP (Código Fiscal de Operações e Prestações)
- `Aac10` – Empresa
- `Edb01` – Período de apuração
- `Aac1001` – Grupo centralizador por empresa/tabela
- `Aag02` / `Aag0201` – UF/Município

## ⚙️ Parâmetros da Fórmula

| Parâmetro   | Tipo             | Obrigatório | Descrição                                      |
|-------------|------------------|-------------|------------------------------------------------|
| whereTipo   | ClientCriterion  | Não         | Critério de filtro por tipo de documento       |
| whereData   | ClientCriterion  | Sim         | Intervalo de datas (inicial e final)           |
| whereEnt    | ClientCriterion  | Não         | Critério por entidade (cliente/fornecedor)     |
| whereNum    | ClientCriterion  | Não         | Critério por número do documento               |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação e parse das datas inicial e final
- Obtenção da empresa ativa e matriz (se houver)
- Inicialização do arquivo de texto de saída

### 2. **Geração do Registro R11 – Apuração do IPI (Entradas)**
- Busca dados de entradas (movimento = 0)
- Filtra CFOP 1604 (isento de registro)
- Formata e imprime linhas no formato R11

### 3. **Geração do Registro R12 – Apuração do IPI (Saídas)**
- Busca dados de saídas (movimento = 1)
- Formata e imprime linhas no formato R12

### 4. **Geração do Registro R13 – Notas Fiscais de Entrada/Aquisição**
- Busca documentos de entrada com IPI > 0
- Obtém CFOP do primeiro item
- Ignora CFOP 1604
- Formata e imprime linhas no formato R13

### 5. **Geração do Registro R21 – Livro de Apuração do IPI**
- Busca grupo centralizador “EE”
- Obtém período de apuração (Edb01)
- Calcula créditos e débitos nacionais e do exterior
- Aplica estornos e outros valores do JSON
- Define situação da apuração (1 = sem movimento, 2 = com movimento)
- Formata e imprime linha no formato R21

### 6. **Finalização**
- Arquivo de texto é retornado no parâmetro `dadosArquivo`

## ⚠️ Regras de Negócio

### Filtros de Movimentação
- **Movimento 0**: Entradas
- **Movimento 1**: Saídas
- **Movimento 3**: Notas fiscais de entrada com IPI

### CFOP
- CFOP 1604 é ignorado nos registros R11 e R13
- CFOPs iniciados com 1, 2, 3 são considerados entradas
- CFOPs iniciados com 5, 6, 7 são considerados saídas

### Cálculos Financeiros
- Valores monetários são multiplicados por 100 (conversão para inteiro)
- Campos JSON (`ipi_ipi`, `ipi_bc`, `ipi_isento`, `ipi_outras`) são somados por CFOP
- Créditos e débitos são calculados separadamente para operações nacionais e do exterior

### Validações
- Apenas documentos não cancelados são considerados
- A data do documento é usada conforme o tipo de movimento (entrada/saída)
- Município com UF = “EX” indica operação com exterior

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração do arquivo.

### `buscarDados(ClientCriterion whereTipo, ClientCriterion whereData, ClientCriterion whereNum, ClientCriterion whereEnt, Integer mov)`
Busca dados agregados de IPI por CFOP conforme o movimento.

### `buscarDocumentos(ClientCriterion whereTipo, ClientCriterion whereData, ClientCriterion whereNum, ClientCriterion whereEnt, Integer mov)`
Busca documentos fiscais completos para o registro R13.

### `buscarCfopItemDocumento(Long eaa01id)`
Retorna o CFOP do primeiro item de um documento.

### `buscarGrupoCentralizadorPorEmpresaTabela(Long aac10id, String tabela)`
Busca ID do grupo centralizador pela tabela (ex: “EE”).

### `buscarApuracaoPorPeriodo(Long edb01tipo, Integer edb01ano, Integer edb01mes)`
Busca período de apuração pelo tipo, ano e mês.

### `buscarSomaDebOuCredIPI(LocalDate dataIni, LocalDate dataFin, Boolean isEntrada)`
Soma valores de IPI para entradas ou saídas no período.

### `buscarSomaDebOuCredIPIExterior(LocalDate dataIni, LocalDate dataFin, Boolean isEntrada)`
Soma valores de IPI para operações com o exterior.

## 📊 Estrutura de Saída

**Arquivo de Texto (TextFile):**
- Linhas no formato fixo, com campos alinhados e preenchidos
- Registros R11, R12, R13 e R21
- Retornado no parâmetro `dadosArquivo`

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Criteria e consultas ao banco
- `multitec.utils` – Utilitários de texto, data e arquivo
- `sam.dicdados` – Tipos de fórmula
- `sam.model` – Entidades do sistema
- `java.time` – Manipulação de datas

**Módulo:** SGT (Sistema de Gestão Tributária)

## 📝 Observações Técnicas

### Formato de Datas
- Entrada: `yyyyMMdd`
- Saída: `ddMMyyyy`

### Campos Monetários
- Convertidos para inteiro (×100) no arquivo de saída
- Valores nulos são tratados como zero

### Consultas SQL
- Uso de `jGet()` para extrair valores JSON
- Junções complexas com múltiplas tabelas
- Filtros dinâmicos via `ClientCriterion`

### Tratamento de Exterior
- Operações com UF = “EX” são consideradas exterior
- Valores de exterior são subtraídos dos totais nacionais

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Exportação Fiscal  
**Versão:** 1.0  