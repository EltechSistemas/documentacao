# SRF - Resumo Por Estado

## 📖 Descrição
Relatório resumido de documentos fiscais consolidados por estado/UF, com tratamento de devoluções e agrupamento automático para análise geográfica.

## 🎯 Finalidade
Fornecer visão consolidada das operações fiscais por estado, permitindo análise rápida de faturamento ou recebimento com totais agrupados por UF e ajustes para devoluções.

## 👥 Público-Alvo
- Departamento Fiscal
- Controladoria
- Gestão Comercial
- Diretoria

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades
- `AAG02` - Estados/UF
- `AAG0201` - Municípios
- `AAH01` - Tipos de documento
- `AAC10` - Empresas

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Abe01` - Clientes/Fornecedores
- `Aag02` - Estado
- `Aac10` - Empresa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| resumoOperacao | Integer | Sim | Tipo de operação | 0=Recebimento, 1=Faturamento |
| impressao | Integer | Sim | Formato saída | 0=PDF, 1=XLSX |
| numeroInicial | Integer | Sim | Número documento inicial | 000000001 |
| numeroFinal | Integer | Sim | Número documento final | 999999999 |
| entidade | List<Long> | Não | Entidades específicas | IDs entidades |
| tipo | List<Long> | Não | Tipos documento | IDs tipos documento |
| empresas | List<Long> | Não | Empresas específicas | IDs empresas |
| dataEmissao | LocalDate[] | Não | Período emissão | Data inicial e final |
| dataEntSai | LocalDate[] | Não | Período entrada/saída | Data inicial e final |
| estados | List<Long> | Não | Estados específicos | IDs estados |
| pcd | List<Long> | Não | Critérios PCD | IDs critérios |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| aag02uf | Estado/UF | String |
| aag02nome | Nome do estado | String |
| eaa0103qtComl | Soma quantidade comercial | BigDecimal |
| eaa0103unit | Soma preço unitário | BigDecimal |
| eaa0103total | Soma total item | BigDecimal |
| eaa0103totDoc | Soma total documento | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Define operação (faturamento/recebimento)
   - Configura período no cabeçalho
   - Processa agrupamentos de entidades

2. **Busca de Agrupamentos**
   - Identifica códigos de entidade com 2 caracteres
   - Expande para incluir entidades filhas
   - Prepara lista completa de IDs

3. **Consulta de Dados Detalhados**
   - Busca documentos com filtros aplicados
   - Processa devoluções para cada item
   - Ajusta valores com base nas devoluções

4. **Consolidação por Estado**
   - Agrupa dados por UF
   - Soma valores para cada estado
   - Cria estrutura resumida final

5. **Geração de Saída**
   - Formata dados consolidados
   - Exporta para PDF ou XLSX
   - Configura cabeçalho com período

## ⚠️ Regras de Negócio

### Agrupamento de Entidades
- Códigos com 2 caracteres considerados como agrupadores
- Inclui automaticamente entidades com código iniciando pelo agrupador
- Expande lista de IDs para entidades filhas

### Filtros de Operação
- **Recebimento:** `eaa01esMov = Eaa01.ESMOV_ENTRADA`
- **Faturamento:** `eaa01esMov = Eaa01.ESMOV_SAIDA`

### Tratamento de Devoluções
- Busca itens com relacionamento `EAA01033`
- Subtrai quantidades e valores devolvidos
- Mantém apenas valores líquidos no relatório

### Consolidação
- Agrupamento automático por estado
- Soma de todos os valores por UF
- Manutenção da ordenação por estado

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Relatório formatado | `gerarPDF()` |
| XLSX | Planilha para análise | `gerarXLSX()` |

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários e datas
- `jasperreports` - Geração de relatórios

**Configurações:**
- Templates Jasper para formatação
- Parâmetros JSON para campos adicionais

## 📝 Observações Técnicas

- Processamento em duas etapas: detalhado → consolidado
- Expansão automática de agrupamentos de entidades
- Tratamento complexo de devoluções antes da consolidação
- Algoritmo de agrupamento manual por estado na memória
- Manutenção da ordenação original durante a consolidação
- SQL com múltiplos joins para dados geográficos