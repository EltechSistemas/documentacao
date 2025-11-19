# SRF_FaturamentoXPrecoMedio - Relatório de Faturamento x Preço Médio

## 📖 Descrição
Relatório analítico e sintético de faturamento versus preço médio de vendas, com múltiplos layouts e agrupamentos para análise comercial e estratégica de precificação.

## 🎯 Finalidade
Fornecer insights sobre o desempenho de vendas através da análise do faturamento e preço médio por grupo de itens, permitindo identificar tendências de mercado e eficácia da estratégia de precificação.

## 👥 Público-Alvo
- Departamento Comercial
- Gestores de Vendas
- Diretoria Executiva
- Planejamento Estratégico
- Controladoria

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| empresas | List<Long> | Não | Empresas específicas | IDs empresas |
| emissao | LocalDate[] | Sim | Período de emissão | Data inicial e final |
| layout | Integer | Sim | Tipo de layout | 0=Sintético, 1=Analítico |
| sintetico | Integer | Sim | Nível de detalhe | 0=Padrão, 1=Nacional, 2=Por Cor, 3=Exportação |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| grupoItem | Código do grupo de item | String |
| nomeGrupo | Nome do grupo de item | String |
| qtComlTotal | Quantidade comercial total | BigDecimal |
| total | Valor total líquido | BigDecimal |
| media | Preço médio | BigDecimal |
| grupoNat | Grupo natural/cor | String |
| abm01codigo | Código do item | String |
| abm01na | Nome do item | String |
| abb01num | Número do documento | Integer |
| valorCotacao | Valor da cotação | BigDecimal |
| mediaDolar | Média em dólar | BigDecimal |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Define valores padrão (mês atual, layout sintético)
- Obtém lista de empresas selecionadas
- Formata período para exibição

### 2. **Seleção do Layout**
- **Layout 0 (Sintético)**: Dados consolidados por grupo
  - Sintético 0: Agrupamento padrão
  - Sintético 1: Separação NAT/COR
  - Sintético 2: Detalhamento por cor
  - Sintético 3: Exportação com cotação
- **Layout 1 (Analítico)**: Dados detalhados por documento

### 3. **Busca de Dados**
- Consulta grupos de itens ativos
- Executa query específica conforme layout selecionado
- Calcula médias e totais

### 4. **Processamento de Dados**
- Combina dados de faturamento com grupos de itens
- Para exportação, busca e calcula cotações de dólar
- Ordena resultados por nome do grupo

### 5. **Geração do Relatório**
- Seleciona template JasperReports conforme layout
- Gera PDF com dados processados
- Retorna arquivo para download

## ⚠️ Regras de Negócio

### Filtros de Dados:
- **Documentos de venda**: clasDoc = 1, esMov = 1
- **Operações comerciais**: tipocod = 1
- **Itens válidos**: tipo in (0, 1)
- **Exclusões**: Operações 112 e 115 (exceto exportação)

### Cálculo de Valores:
- **Total líquido**: totDoc - ICMS - PIS - COFINS
- **Preço médio**: Total líquido / Quantidade
- **Código grupo**: 5 dígitos (tipo 0) ou 8 dígitos (tipo 1)

### Agrupamentos Específicos:
- **NAT/COR**: Identifica pelo nome do item contendo "NAT"
- **Exportação**: Inclui apenas operações 112 e 115
- **Cotação**: Média das cotações no período

### Validações:
- Divisão por zero protegida em cálculos de média
- Tratamento de exceções em conversão para dólar
- Ordenação consistente por tipo e código

## 🎨 Saídas Disponíveis

| Layout | Sintético | Template | Descrição |
|--------|-----------|----------|-----------|
| 0 | 0 | `SRF_FaturamentoXPrecoMedio_Sintetico` | Agrupamento padrão |
| 0 | 1 | `SRF_FaturamentoXPrecoMedio_Sintetico_S1` | Separação NAT/COR |
| 0 | 2 | `SRF_FaturamentoXPrecoMedio_Sintetico_S2` | Detalhamento por cor |
| 0 | 3 | `SRF_FaturamentoXPrecoMedio_Sintetico_S3` | Exportação com cotação |
| 1 | - | `SRF_FaturamentoXPrecoMedio_Analitico` | Detalhado por documento |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios PDF
- `br.com.multitec.utils` - Utilitários e datas

**Entidades:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens de documentos
- `Abm01` - Itens cadastrais
- `Aac10` - Empresas
- `Abb10` - Operações comerciales

## 📝 Observações Técnicas

### Consultas SQL:
- Múltiplas variações conforme layout selecionado
- Uso de substring para agrupamento por código
- Extração de campos JSON para valores fiscais
- Agregação com GROUP BY dinâmico

### Performance:
- Consultas otimizadas com filtros aplicados no banco
- Processamento em memória apenas para combinação final
- Paginação tratada pelo JasperReports

### Tratamento de Dados:
- Formatação personalizada do período
- Concatenação de nomes de empresas
- Cálculo de médias com proteção contra divisão por zero
- Ordenação alfabética por nome do grupo

### Cálculo de Cotação:
- Busca cotações históricas para datas de emissão
- Calcula média ponderada das cotações
- Converte valores para dólar quando aplicável