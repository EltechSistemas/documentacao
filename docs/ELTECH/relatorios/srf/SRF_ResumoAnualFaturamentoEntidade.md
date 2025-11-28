# SRF – Resumo Anual Faturamento por Entidade

## 📖 Descrição
Relatório anual consolidado de faturamento por entidade (cliente), trazendo valores brutos e líquidos por mês, com tratamento de devoluções, agrupamento de entidades por código e filtros corporativos para análise gerencial.

## 🎯 Finalidade
Fornecer visão anual do faturamento bruto e líquido de cada entidade, mês a mês, permitindo à gestão analisar evolução, sazonalidade e impacto de devoluções por cliente.

## 👥 Público-Alvo
- Controladoria
- Departamento Fiscal
- Gestão Comercial
- Diretoria / Gerência Administrativa

## 📊 Dados e Fontes

### Tabelas Principais
- `EAA01` — Documentos fiscais
- `EAA0103` — Itens de documentos
- `EAA01033` — Relacionamento de devoluções
- `ABB01` — Cabeçalho do documento
- `ABB10` — Operações comerciais
- `ABE01` — Entidades (clientes, representantes)
- `AAC10` — Empresas
- `AAJ15` — CFOP

### Entidades Envolvidas
- `Abe01` — Clientes e representantes
- `Aac10` — Empresa
- `Eaa01 / Eaa0103` — Documentos e itens de faturamento
- `Eaa01033` — Itens de devolução vinculados

---

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| faturamento | Integer | Não | Tipo de operação (não utilizado diretamente) | 0 / 1 |
| valorRelatorio | Integer | Sim | Define se relatório soma quantidade ou valor | 0=Quantidade, 1=Total documento |
| anoRef | String | Sim | Ano de referência do relatório | AAAA |
| dtIni | String | Automático | Mês/ano inicial | "MM/yyyy" |
| dtFim | String | Automático | Mês/ano final | "MM/yyyy" |
| impressao | Integer | Sim | Formato de saída | 0=PDF, 1=XLSX |
| entidade | List<Long> | Não | IDs de entidades (clientes) | Lista de IDs |
| tipoDoc | List<Long> | Não | Tipos de documento | Lista de IDs |
| numero | Integer | Não | Número de documento | Inteiro |
| pcd | List<Long> | Não | Critérios PCD | Lista |
| representantes | List<Long> | Não | Representantes vinculados | Lista |
| empresas | List<Long> | Não | Empresas | Lista |
| liquido | Boolean | Não | Define se mostra apenas o valor líquido | true/false |

---

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| codcli | Código da entidade (cliente) | String |
| nomecli | Nome da entidade | String |
| Janeiro…Dezembro | Valor bruto e líquido do mês | BigDecimal |
| totalbruto | Soma anual do valor bruto | BigDecimal |
| totalliquido | Soma anual do valor líquido | BigDecimal |

---

## 🔄 Fluxo do Processo

### 1. Carregamento de Parâmetros e Ano de Referência
- Prepara datas de início e fim do ano.
- Obtém nome(s) das empresas para exibição.
- Define se relatório usa quantidade ou valor.

### 2. Processamento de Agrupamentos de Entidades
- Identifica códigos com 2 dígitos (agrupadores).
- Expande para incluir entidades com início igual ao grupo.
- Retorna IDs finais para inclusão no filtro do SQL.

### 3. Busca de Faturamento
- Aplica filtros de período, empresas, clientes e representantes.
- Somente documentos de saída válidos entram no cálculo:
    - SRF (`eaa01clasDoc = 1`)
    - Saída (`eaa01esMov = 1`)
    - Emissão própria
    - Não cancelados
    - `abb10tipoCod IN (1,6)`
- CFOP terminando em **124** recebe tratamento especial.

### 4. Tratamento de Devoluções
- Identifica devoluções através de `EAA01033`.
- Soma valores devolvidos por mês.
- Subtrai do faturamento bruto para obter o líquido.

### 5. Consolidação Mensal por Entidade
- Cria acumuladores `TableMap` por cliente.
- Preenche os meses: Janeiro, Fevereiro, ..., Dezembro.
- Calcula totais anuais bruto e líquido.
- Mantém ordenação por código da entidade.

### 6. Geração da Saída
- Se `impressao = 1`: gera XLSX.
- Caso contrário: gera PDF.
- Envia dataset consolidado ao Jasper.

---

## ⚠️ Regras de Negócio

### Agrupamento de Entidades
- Códigos com **2 caracteres** são agrupadores.
- Entidades com código iniciando nesses 2 caracteres são incluídas.
- Usuario não precisa informar todas as entidades individualmente.

### Seleção de Valores (valorRelatorio)
- **0** → Soma quantidade comercial (`eaa0103qtComl`)
- **1** → Soma total do item (`eaa0103total`)

### Filtros de Faturamento
Aceita apenas documentos com:
- `eaa01clasDoc = 1`
- `eaa01esMov = 1`
- `eaa01emissao = 1`
- `abb10tipoCod IN (1,6)`
- Não cancelados
- Itens CFOP `%124` sempre incluídos

### Devoluções
- `aaj03codigo = '09'`
- `abb10tipocod = 4`
- Somente devoluções dentro do período
- Vinculação item a item via `EAA01033`

### Consolidação
- Agrupamento final por cliente.
- Soma de valores mensais e totais.
- Mantém meses como chave textual via `verificaMes()`.

---

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Relatório para impressão | `gerarPDF()` |
| XLSX | Planilha analítica com meses | `gerarXLSX()` |

---

## 🔧 Dependências

### Bibliotecas
- `multitec.utils` — Datas e utilitários
- `TableMap` — Estrutura de dados
- `jasperreports` — Engine de relatórios

### Configurações
- Template Jasper dedicado
- Parâmetros de header (ano, empresa, período)

---

## 📝 Observações Técnicas

- Relatório **sempre cobre janeiro a dezembro** do ano informado.
- Preenchimento de mês feito por `verificaMes()`.
- Cálculo e agrupamento feitos **em memória** (Java/Groovy).
- SQL separado para faturamento e devoluções.
- Usa funções SQL como `string_agg`, `extract(month)`, `like any`.
- CFOP `%124` possui lógica especial no somatório.
- Exportação depende de `impressao`:
    - `0` → PDF
    - `1` → XLSX

---
