# SRF - Resumo Por Item

## 📖 Descrição
Relatório resumido de documentos fiscais consolidados por item, com flexibilidade para seleção de campos personalizados e tratamento de devoluções.

## 🎯 Finalidade
Fornecer visão consolidada das operações fiscais por item, permitindo análise personalizada com campos flexíveis e ajustes automáticos para devoluções.

## 👥 Público-Alvo
- Departamento Fiscal
- Controladoria
- Gestão de Estoque
- Comercial

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA0103` - Itens de documentos fiscais
- `EAA01` - Documentos fiscais
- `ABB01` - Cabeçalho de documentos
- `ABM01` - Itens/Cadastro de produtos
- `AAC10` - Empresas
- `AAM06` - Unidades de medida
- `ABE01` - Entidades
- `ABM0102` - Critérios de itens

**Entidades Envolvidas:**
- `Eaa0103` - Itens de documentos
- `Abm01` - Itens/produtos
- `Eaa01` - Documentos fiscais
- `Aac10` - Empresa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| resumoOperacao | Integer | Sim | Tipo de operação | 0=Entrada, 1=Saída |
| impressao | Integer | Sim | Formato saída | 0=PDF, 1=XLSX |
| numeroInicial | Integer | Sim | Número documento inicial | 000000001 |
| numeroFinal | Integer | Sim | Número documento final | 999999999 |
| entidade | List<Long> | Não | Entidades específicas | IDs entidades |
| tipo | List<Long> | Não | Tipos documento | IDs tipos documento |
| itens | List<Long> | Não | Itens específicos | IDs itens |
| empresas | List<Long> | Não | Empresas específicas | IDs empresas |
| dataEmissao | LocalDate[] | Não | Período emissão | Data inicial e final |
| dataEntSai | LocalDate[] | Não | Período entrada/saída | Data inicial e final |
| criterios | List<Long> | Não | Critérios de item | IDs critérios |
| cfop | List<Long> | Não | CFOPs específicos | IDs CFOPs |
| pcd | List<Long> | Não | Critérios PCD | IDs critérios PCD |

## ⚙️ Configuração
**Campos Personalizáveis:**
- 6 campos configuráveis (fixos ou livres)
- Campos fixos: Qtde Uso, Qtde Comercial, Preço Unitário, Total Item, Total Documento, Total Financeiro
- Campos livres: Qualquer campo do JSON do item
- Validação contra duplicação de campos

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abm01codigo | Código do item | String |
| abm01descr | Descrição do item | String |
| aam06codigo | Unidade de medida | String |
| eaa0103qtComl | Quantidade comercial | BigDecimal |
| eaa0103unit | Preço unitário | BigDecimal |
| eaa0103total | Total item | BigDecimal |
| nomeCampo[1-6] | Nome campo personalizado | String |
| valor[1-6] | Valor campo personalizado | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Valida campos personalizados (sem duplicatas)
   - Define operação (entrada/saída)
   - Processa agrupamentos de entidades e itens

2. **Busca de Agrupamentos**
   - Identifica códigos com 2 caracteres como agrupadores
   - Expande para incluir entidades e itens filhas
   - Prepara listas completas de IDs

3. **Consulta de Dados Principais**
   - Busca itens consolidados com filtros aplicados
   - Agrupa por item com totais
   - Aplica critérios de operação

4. **Processamento de Campos Personalizados**
   - Busca campos JSON para cada item
   - Calcula totais dos campos livres
   - Aplica ajustes de devoluções

5. **Tratamento de Devoluções**
   - Identifica itens com devoluções
   - Subtrai valores devolvidos dos totais
   - Ajusta campos personalizados

6. **Geração de Saída**
   - Formata dados com campos personalizados
   - Exporta para PDF ou XLSX
   - Configura totais por campo

## ⚠️ Regras de Negócio

### Agrupamento
- Códigos com 2 caracteres considerados como agrupadores
- Expansão automática para entidades e itens filhas
- Agrupamento por tipo e código de item

### Campos Personalizados
- Máximo 6 campos por relatório
- Combinação de campos fixos e livres
- Validação contra duplicação
- Campos livres somados do JSON dos itens

### Devoluções
- Subtrai quantidades e valores devolvidos
- Ajusta campos personalizados quando aplicável
- Considera apenas devoluções com quantidades > 0

### Filtros
- Apenas documentos fiscais (`CLASDOC_SRF`)
- Exclui documentos cancelados
- Filtros por período de emissão ou entrada/saída

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
- Parâmetros JSON para campos personalizados
- Tabela `AAH02` para descrição de campos

## 📝 Observações Técnicas

- Flexibilidade total na seleção de campos
- Processamento complexo de agrupamentos
- Duas consultas SQL: dados principais + campos personalizados
- Algoritmo de validação de campos contra duplicação
- Suporte a múltiplos critérios de filtragem
- Tratamento automático de devoluções