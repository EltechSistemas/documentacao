# SAP - Ficha Patrimonial (Eltech)

## 📖 Descrição
Relatório para geração da ficha patrimonial de bens, contendo dados de imobilizado, depreciação e reclassificações, com agrupamento por código de bem.

## 🎯 Finalidade
Gerar um documento PDF estruturado com informações patrimoniais, organizadas por grupo de bens, para análise e controle contábil.

## 👥 Público-Alvo
- Departamento Contábil
- Patrimônio
- Auditoria Interna

## 📊 Dados e Fontes

**Tabelas Principais:**
- `abb20` - Cadastro de bens
- `ecb01` - Dados de imobilizado
- `abb01` - Centros de custo
- `abe01` - Entidades (responsáveis)
- `aah01` - Tipos de centro de custo
- `ecb0102` - Depreciações acumuladas
- `ecb0101` - Reclassificações
- `eca01` - Classificações contábeis
- `abb11` - Departamentos

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| clas | List<Long> | Não | IDs das classificações de bens para filtro |
| bens | List<Long> | Não | IDs dos bens para filtro |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação e captura dos parâmetros de entrada
- Carregamento dos dados principais da ficha patrimonial
- Definição dos grupos de bens baseado no código

### 2. **Agrupamento de Dados**
- Identificação de grupos baseados no código do bem (terminação "000")
- Criação de estruturas de dados para:
  - Dados principais do bem
  - Subdados de imobilizado
  - Subdados de depreciação
  - Subdados de reclassificação

### 3. **Busca de Dados Complementares**
- Consulta de dados de depreciação por grupo de bens
- Consulta de dados de reclassificação por grupo de bens
- Associação dos dados complementares aos grupos correspondentes

### 4. **Montagem da Estrutura do Relatório**
- Configuração de DataSources principais e subsidiários
- Associação de sub-relatórios para cada seção
- Adição de parâmetros comuns (ex: nome da empresa)

### 5. **Geração do PDF**
- Montagem final do documento
- Aplicação de templates e formatação
- Retorno do arquivo para download

## ⚠️ Regras de Negócio

### Agrupamento de Bens
- **Grupo**: Definido pelo código do bem terminado em "000"
- **Hierarquia**: Bens com mesmo prefixo pertencem ao mesmo grupo
- **Controle**: Sequência numérica automática por ordem de código

### Dados de Imobilizado
- **Informações básicas**: Código, nome, descrição, chapa, data de aquisição
- **Valores**: Valor de aquisição, valor atual, valor de baixa
- **Localização**: Centro de custo, responsável, tipo

### Depreciação
- **Agrupamento**: Por ano e mês
- **Cálculo**: Soma dos valores depreciados por período
- **Ordenação**: Cronológica (ano → mês)

### Reclassificação
- **Período**: Mês e ano da reclassificação
- **Classificação**: Código e nome da nova classificação contábil
- **Departamento**: Departamento associado

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração do relatório.

### `buscarDadosFicha(List<Long> clas, List<Long> bens)`
Busca os dados principais da ficha patrimonial, aplicando filtros por classificação e bens.

### `buscarDadosDeprec(List<Long> idsBensImob)`
Busca dados de depreciação acumulada para os bens informados.

### `buscarDadosReclassif(List<Long> idsBensImob)`
Busca dados de reclassificação para os bens informados.

### `agruparPorGrupo(Integer grupo, Long ecb01id, Map<Integer, List<Long>> grupoMap)`
Agrupa IDs de bens por grupo para consultas complementares.

## 📊 Estrutura de Saída

**Relatório Principal (PDF):**
- Seção 1: Dados principais do bem (código, nome, valores, localização)
- Seção 2: Histórico de depreciação (tabela por período)
- Seção 3: Histórico de reclassificações (tabela por período)

**Estrutura de Dados:**
- `grupo`: Identificador numérico do grupo
- `key`: Chave de relacionamento entre seções
- `abb20codigo`: Código do bem
- `abb20nome`: Nome do bem
- `ecb01vlraquis`: Valor de aquisição
- `ecb0102deprec`: Valor depreciado no período
- `eca01nome`: Nome da classificação contábil

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Acesso ao banco de dados
- `sam.server.samdev.relatorio` - Framework de relatórios
- `sam.server.samdev.utils` - Utilitários do sistema
- `sam.model` - Entidades do sistema

**Módulo:** Relatórios SAP

## 📝 Observações Técnicas

### Tratamento de Grupos
- Uso de window function para identificação de grupos
- Mapa de agrupamento para otimizar consultas complementares
- Chave única (`key`) para relacionar seções do relatório

### Sub-relatórios
- Três sub-relatórios independentes:
  - `SAP_FichaPatrimonial_S1`: Dados de imobilizado
  - `SAP_FichaPatrimonial_S2`: Dados de depreciação
  - `SAP_FichaPatrimonial_S3`: Dados de reclassificação
- Vinculação por chave comum (`key`)

### Performance
- Consultas otimizadas com filtros dinâmicos
- Agrupamento em memória para reduzir chamadas ao banco
- Uso de TableMap para estruturação flexível dos dados

### Formatação
- Conversão de meses para formato "00" (TO_CHAR)
- Ordenação cronológica em todas as seções temporais
- Nome da empresa como parâmetro global

---

**Última Alteração:** 09/12/2025 às 09:20  
**Autor:** Bruno  
**Tipo:** Relatório SAP  
**Versão:** 1.0