# SCF_ExtratoFinanceiro

## 📖 Descrição
Relatório de extrato financeiro para o módulo Linhasita, que consolida informações de documentos financeiros (contas a receber e a pagar), calculando totais, juros, multas, descontos e formas de pagamento por entidade.

## 🎯 Finalidade
Gerar extratos financeiros detalhados por cliente/fornecedor, com opções de filtragem por período, portador, operação, classe de documento e status (a receber/recebidas, a pagar/pagas).

## 👥 Público-Alvo
- Departamento Financeiro
- Crédito e Cobrança
- Controladoria
- Gestão de Contas a Pagar/Receber

## 📊 Dados e Fontes

**Tabelas Principais:**
- `daa01` – Documentos financeiros
- `abb01` – Documentos de origem (central)
- `abe01` – Entidades (clientes/fornecedores)
- `abe0101` – Endereços das entidades
- `abe0104` – Contatos das entidades
- `abe02` – Representantes
- `abf15` – Portadores
- `abf16` – Operações
- `abf20` – Plano financeiro
- `abf40` – Formas de pagamento
- `dab10` – Baixas de documentos
- `dab1002` – Detalhes de baixa
- `aah01` – Tipos de documento
- `aba2001` – Observações de entidade

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| entidade | List<Long> | Não | IDs das entidades |
| portadorInicial | String | Não | Código inicial do portador |
| portadorFinal | String | Não | Código final do portador |
| operacaoInicial | String | Não | Código inicial da operação |
| operacaoFinal | String | Não | Código final da operação |
| receber | LocalDate[] | Não | Período de documentos a receber |
| recebidas | LocalDate[] | Não | Período de documentos recebidos |
| pagar | LocalDate[] | Não | Período de documentos a pagar |
| pagas | LocalDate[] | Não | Período de documentos pagos |
| vencimento | Integer | Sim | Tipo de vencimento (0=Nominal, 1=Real) |
| classe | Integer | Sim | Classe do documento (0=Real, 1=Previsão) |
| ordem | Integer | Sim | Ordenação (0=Vencimento, 1=Número documento) |
| forma | Boolean | Sim | Incluir formas de pagamento |
| imprimir | Integer | Sim | Formato de saída (0=XLSX, 1=PDF) |
| considerarTransmutado | Boolean | Sim | Considerar documentos transmutados |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Define nome da tarefa
- Configura filtros padrão

### 2. **Coleta de Parâmetros**
- Obtém todos os parâmetros do relatório
- Busca empresa ativa

### 3. **Processamento por Entidade**
- Para cada entidade selecionada:
  - Busca dados básicos e representantes
  - Extrai informações de crédito do JSON
  - Busca documentos financeiros com filtros aplicados
  - Calcula totais, juros, multas e descontos
  - Compõe endereços e contatos
  - Se solicitado, busca formas de pagamento

### 4. **Consolidação**
- Agrega totais por categoria (receber/recebidas/pagar/pagas)
- Adiciona linhas de totalização por entidade
- Prepara estruturas de dados para sub-relatórios

### 5. **Geração do Relatório**
- Define estrutura de dados principal e sub-datasources
- Seleciona formato de saída (PDF ou XLSX)
- Gera arquivo final

## ⚠️ Regras de Negócio

### Filtros de Documentos
- Filtragem por período (emissão ou pagamento)
- Seleção por portador e operação (intervalo de códigos)
- Classe do documento (real ou previsão)
- Status (aberto/quitado)

### Cálculos Financeiros
- **Juros/Multa:** Calculados com base em dias de atraso
- **Descontos:** Considerados conforme data limite
- **Dias de Atraso:** Calculados entre data atual/pagamento e vencimento
- **Valores Negativos:** Tratados como positivos para consolidação

### Estrutura do Relatório
- Agrupamento por entidade
- Sub-relatórios para formas de pagamento e detalhes
- Totais consolidados no final de cada entidade

### Representantes
- Até 5 representantes possíveis (rep0 a rep4)
- Prioridade: rep0 → rep1 → rep2 → rep3 → rep4

## 🔧 Métodos Principais

### `executar()`
Método principal que coordena todo o processo de geração do relatório.

### `buscarEntidade(List<Long> enti)`
Busca informações básicas das entidades, incluindo representantes.

### `buscarDocumentos(...)`
Busca documentos financeiros com todos os filtros aplicados.

### `comporEnderecos(TableMap entidade)`
Complementa a entidade com endereços principal, de cobrança e entrega.

### `comporContato(TableMap entidade)`
Adiciona informações de contato (email, telefone).

### `comporFormaPagamento(...)`
Busca formas de pagamento utilizadas nos documentos.

### `somarFormas(...)`
Calcula totais conciliados e não conciliados por entidade.

## 📊 Estrutura de Saída

**Campos Principais por Documento:**
- Dados da entidade (código, nome, representante, etc.)
- Dados do documento (número, série, parcela, valor, etc.)
- Cálculos (dias de atraso, juros/multa, desconto)
- Informações de endereço e contato

**Totais por Entidade:**
- Totais a receber/recebidos/a pagar/pagos
- Juros/multa e descontos por categoria

**Formas de Pagamento (se solicitado):**
- Código e descrição da forma
- Data da baixa
- Valor
- Totais conciliados e não conciliados

**Formatos de Saída:**
- PDF (com sub-relatórios)
- XLSX (com sub-relatórios)

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils.collections` – TableMap
- `sam.server.samdev.relatorio` – Classes base para relatórios
- `java.time` – Manipulação de datas

## 📝 Observações Técnicas

### Consultas SQL Complexas
- Múltiplas junções com tabelas relacionadas
- Agregações com `GROUP BY` e funções de agregação
- Subconsultas para dados mais recentes
- Parâmetros dinâmicos com `criarParametroSql`

### Manipulação de JSON
- Extração de dados de crédito do campo `abe01json`
- Extração de juros/multa/desconto do campo `daa01json`
- Campos específicos: `lim_cred`, `dt_fix_cred`, `obs_cred`, etc.

### Performance
- Processamento em lote por entidade
- Consultas otimizadas com filtros apropriados
- Uso de índices esperados nas tabelas principais

### Sub-relatórios
- Estrutura de `TableMapDataSource` com relacionamentos por chave
- Arquivos de template separados para cada sub-relatório

---

**Última Alteração:** 11/12/2025 às 17:04  
**Autor:** Nagyla  
**Tipo:** Relatório Financeiro  
**Versão:** 1.0  
**Módulo:** Linhasita  
**Código Fonte:** [SCF_ExtratoFinanceiro.groovy]