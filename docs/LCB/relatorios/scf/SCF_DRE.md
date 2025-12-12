# SCF - Demonstrativo de Resultados do Exercício (DRE) - LCB

## 📖 Descrição
Relatório financeiro que gera o Demonstrativo de Resultados do Exercício (DRE) para a empresa LCB, com duas modalidades: sintético (resumido por grupos) e analítico (detalhado por documento). O relatório consolida receitas, custos, despesas e resultados do período.

## 🎯 Finalidade
Fornecer uma visão completa do desempenho financeiro da empresa, permitindo análise de margens, rentabilidade e composição de resultados. Atende tanto a necessidades gerenciais (visão sintética) quanto contábeis (visão analítica).

## 👥 Público-Alvo
- Diretoria/Administração
- Controladoria
- Contabilidade
- Gerência Financeira
- Analistas de Custos

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Daa01` - Documentos financeiros
- `Daa0101` - Departamentos do documento
- `Daa01011` - Naturezas financeiras do departamento
- `Abf10` - Naturezas financeiras
- `Abf11` - Agrupamentos de natureza financeira
- `Abf1101` - Relacionamento natureza-agrupamento
- `Dab10` - Lançamentos contábeis
- `Dab1001` - Departamentos do lançamento
- `Dab10011` - Naturezas do lançamento
- `Abb01` - Central de documento
- `Abe01` - Entidades (clientes/fornecedores)
- `Aac10` - Empresa
- `Bcc01` - Movimentações de estoque (informações adicionais)
- `Abm01` - Produtos
- `Aam01` - Classes de produtos

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valor Padrão |
|-----------|------|-------------|-----------|--------------|
| emissao | LocalDate[] | Sim | Período de emissão/lançamento | Mês atual |
| detalhamento | Integer | Sim | Tipo de detalhamento (0=Sintético, 1=Analítico) | 0 |
| impressao | Integer | Sim | Formato de saída (0=PDF, 1=XLSX) | 0 |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Definição dos valores padrão dos filtros (mês atual)
- Carregamento dos parâmetros fornecidos pelo usuário
- Obtenção dos dados da empresa ativa
- Formatação do período para exibição no relatório

### 2. **Processamento Sintético (detalhamento = 0)**
#### **Coleta de Dados**
- **Vendas**: Busca receitas por linha de produto (IDs específicos)
- **Custos**: Busca custos diretos por linha de produto (IDs correspondentes)
- **Margem**: Obtém estrutura para cálculo de margem bruta
- **Outras Entradas**: Receitas não operacionais
- **Despesas Operacionais**: Divididas em:
  - Custo Fixo (infraestrutura, manutenção)
  - Departamento Pessoal (salários, encargos)
  - Despesas Bancárias (tarifas, juros)
  - Impostos (tributos diretos)
  - ARS Itatiba (despesas específicas)
- **Informações Adicionais**: Dados de estoque por classe de produto
- **Ativo Imobilizado**: Depreciações e amortizações
- **Distribuição de Lucros**: Provisões e distribuições

#### **Cálculos e Consolidação**
- **Totais por Grupo**: Soma de valores por agrupamento
- **Percentuais**: Cálculo de participação sobre vendas ou margem
- **Margem Bruta**: Vendas - Custos Diretos
- **Resultados Intermediários**:
  - Lucro Líquido Operacional: Margem Bruta - Despesas Operacionais
  - Lucro Líquido: Lucro Operacional + Outras Entradas + Ativo Imobilizado
- **Consolidação de Fontes**: Unificação de dados de `Dab10` (lançados) e `Daa01` (não pagos)

#### **Estrutura de Sub-relatórios**
- 13 sub-relatórios interligados via TableMapDataSource
- Cada seção do DRE como sub-relatório independente
- Chave de relacionamento (`key`) para vínculo principal

### 3. **Processamento Analítico (detalhamento = 1)**
#### **Dados Detalhados**
- Mesmas categorias do sintético, mas com detalhamento por documento
- Inclusão de: número documento, data, situação (quitado), entidade
- Consolidação de múltiplas fontes em listas únicas
- Ordenação por agrupamento → número documento → entidade

#### **Estrutura de Sub-relatórios**
- 7 sub-relatórios analíticos
- Layout otimizado para visualização detalhada
- Manutenção da estrutura hierárquica

### 4. **Geração de Saída**
- **PDF**: Relatório formatado para impressão/visualização
- **XLSX**: Planilha Excel para análise e manipulação
- Mesma estrutura de dados para ambos os formatos

## ⚠️ Regras de Negócio

### Categorias Financeiras (IDs Fixos)
#### **Vendas (Abf10 IDs)**
- 45118536, 45118558, 48621447, 45118556, 45118571, 45118538,
- 45118540, 48312987, 45118567, 45118565, 45118575, 45118560,
- 45118569, 45118542, 45118550, 45118563, 50242950, 56159621,
- 63491575, 67170852, 73575997

#### **Custos (Abf10 IDs Correspondentes)**
- 45118606, 45118631, 48621476, 45118629, 45118643,
- 45118611, 45118613, 48313032, 45118639, 45118637, 45118647,
- 45118633, 45118641, 45118616, 45118623, 45118636, 50242996, 56159623,
- 63491577, 67170690, 73575999

#### **Agrupamentos de Despesas (Abf11 IDs)**
- **Outras Entradas**: 45724656
- **Custo Fixo**: 45671519, 45671525, 45671528, 45714999, 45671530, 45671542, 45671532, 45671534, 45671536, 45671538, 45671540, 53634052, 53633915, 79473971
- **Depto Pessoal**: 45671546, 45671548, 45671550, 45671552, 45671554, 68684602, 73779171, 75611157
- **Despesas Bancárias**: 45671560, 45671558
- **Impostos**: 45671571
- **ARS Itatiba**: 47514382
- **Ativo Imobilizado**: 45671568
- **Distribuição Lucros**: 45805292

#### **Informações Adicionais (Bcc01 PLE)**
- 44322655, 44322656, 44514002

### Tratamento de Valores
- **Movimentação de Entrada (dab10mov = 1)**: Valores multiplicados por -1
- **Documentos RP (daa01rp = 1)**: Valores multiplicados por -1
- **Arredondamentos**: 2 casas decimais para valores monetários
- **Percentuais**: Calculados sobre total de vendas ou margem, conforme categoria

### Consolidação de Fontes
- **Dab10**: Documentos já lançados contabilmente
- **Daa01**: Documentos financeiros não pagos (em aberto)
- **Lógica de Junção**: Quando existe `daa01dtLcto`, usa essa data; senão, usa `abb01data`
- **Validação de IDs**: Garantia de que todos os registros da segunda fonte sejam incluídos

### Ordenação
- **Sintético**: Por nome do agrupamento/natureza
- **Analítico**: Por nome do agrupamento → número do documento → nome da entidade
- **Custom Comparators**: Implementação específica para ordenação múltipla

## 🔧 Métodos Principais

### `executar()`
Método principal que direciona para processamento sintético ou analítico.

### Métodos de Busca Sintética
- `buscarDadosVendas()`, `buscarDadosCustos()`, `buscarDadosMargem()`
- `buscarDadosOutrasEntradas()`, `buscarDadosCustoFixo()`, `buscarDadosDeptoPessoal()`
- `buscarDadosDespeBancarias()`, `buscarDadosImpostos()`, `buscarDadosArsItatiba()`
- `buscarDadosInfoAdd()`, `buscarDadosAtivoImob()`, `buscarDistribLucro()`

### Métodos de Busca Analítica
- `buscarDados...Analitico()`: Versões detalhadas de cada categoria
- Incluem dados documentais: número, data, situação, entidade

### Métodos de Cálculo de Totais
- `buscarTotalVendas()`, `buscarTotalCustos()`, `buscarTotalInfoAdd()`
- Retornam somatórios para cálculo de percentuais

## 📊 Estrutura do DRE

### **Seção 1: Receitas Operacionais**
- Vendas por linha de produto (com % sobre total)
- Subtotal: Total de Vendas

### **Seção 2: Custos Operacionais**
- Custos diretos por linha de produto (com % sobre total)
- Subtotal: Total de Custos

### **Seção 3: Margem Bruta**
- Margem por linha de produto (Vendas - Custos)
- Percentual sobre margem total
- Subtotal: Margem Bruta Total

### **Seção 4: Outras Receitas**
- Receitas não operacionais
- Subtotal: Total Outras Entradas

### **Seção 5: Despesas Operacionais**
#### 5.1 Custo Fixo
#### 5.2 Departamento Pessoal
#### 5.3 Despesas Bancárias
#### 5.4 Impostos
#### 5.5 ARS Itatiba
- Subtotal: Total Despesas Operacionais

### **Seção 6: Ativo Imobilizado**
- Depreciações e amortizações

### **Seção 7: Resultados**
- Lucro Líquido Operacional (Margem - Despesas)
- Distribuição de Lucros
- Lucro Líquido do Exercício

### **Seção 8: Informações Adicionais**
- Movimentações de estoque por classe de produto

## 🔧 Dependências

**Bibliotecas:**
- `br.com.multitec.utils` - Utilitários de data e coleções
- `br.com.multitec.utils.collections.TableMap` - Estrutura de dados
- `sam.server.samdev.relatorio` - Classes base para relatórios
- `sam.core.variaveis.MDate` - Manipulação de datas
- `java.time` - API de datas moderna
- `java.util` - Coleções e comparators

**Módulo:** SCF (Sistema Contábil Financeiro)

## 📝 Observações Técnicas

### Performance de Consultas
- Consultas otimizadas com filtros por período
- Uso de subqueries para consolidação
- Agrupamentos (GROUP BY) no banco para reduzir processamento
- Índices sugeridos em: `daa01dtLcto`, `abb01data`, `abf10id`, `abf11id`

### Tratamento de Dados
- Conversão segura de tipos (getBigDecimal_Zero)
- Validação de nulls antes de cálculos
- Arredondamento consistente em 2 casas decimais
- Formatação de datas no padrão brasileiro (dd/MM/yyyy)

### Arquitetura de Sub-relatórios
- **TableMapDataSource**: Estrutura hierárquica de dados
- **Chaves de Relacionamento**: Campo `key` para vínculo entre níveis
- **SubDataSources Múltiplos**: Até 13 sub-relatórios no modo sintético
- **Templates Independentes**: Cada seção com layout específico

### Flexibilidade do Relatório
- **Duas Modalidades**: Sintético (gerencial) e Analítico (contábil)
- **Dois Formatos**: PDF (visualização) e XLSX (análise)
- **Período Customizável**: Qualquer intervalo de datas
- **Empresa Ativa**: Dados específicos da empresa do usuário logado

### Validações e Tratamento de Erros
- Verificação de dados vazios/nulos
- Cálculos de percentuais com proteção contra divisão por zero
- Consolidação segura de múltiplas fontes de dados
- Ordenação robusta com comparators customizados

---

**Última Alteração:** 28/11/2025 às 08:50  
**Autor:** Bruno  
**Tipo:** Relatório Financeiro (DRE)  
**Versão:** 1.0