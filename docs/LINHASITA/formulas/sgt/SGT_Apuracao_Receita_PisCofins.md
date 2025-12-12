# SGT - Apuração de Receita PIS/COFINS

## 📖 Descrição
Fórmula para apuração de receitas tributáveis e não tributáveis, cálculo da Contribuição Previdenciária sobre a Receita Bruta (P100) e consolidação de dados para escrituração fiscal (PIS/COFINS).

## 🎯 Finalidade
Automatizar a apuração de receitas e o cálculo da contribuição previdenciária sobre a receita bruta, considerando matriz e filiais, classificações de receita, atividades econômicas e ajustes por devoluções.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Auditoria Fiscal

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Aac10` - Empresas (matriz e filiais)
- `Edb10` - Período de apuração
- `Edb1001` - Empresas com receita no período
- `Edb10011` - Atividades com contribuição previdenciária
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal
- `Abg02` - Atividades econômicas (CPRB)
- `Abc10` - Contas contábeis
- `Abb01` - Lançamentos contábeis
- `Aah01` - Tipos de documento

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb10 | Edb10 | Sim | Período de apuração (mês/ano) |
| campos JSON | String | Sim | Campos da tabela P100 para extração dos valores (vlRecTotEst, vlRecAtivEstab, vlExc, aliqCont) |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Seleção do alinhamento contábil (`0044`)
- Busca da matriz e filiais relacionadas
- Extração dos campos JSON da P100

### 2. **Apuração da Receita**
- Classificação da receita em:
  - Tributável (classe 1)
  - Não tributável (classe 2)
  - Exportação (classe 3)
  - Cumulativo (classe 4)
- Consolidação por empresa (matriz + filiais)

### 3. **Cálculo da Contribuição Previdenciária (P100)**
- Agrupamento por atividade econômica (`Abg02`)
- Soma das receitas totais e receitas com atividade
- Dedução de devoluções
- Cálculo da base de cálculo e contribuição

### 4. **Consolidação dos Resultados**
- Preenchimento dos campos `Edb10`:
  - `edb10rbTrib` - Receita tributável
  - `edb10rbNaoTrib` - Receita não tributável
  - `edb10rbExp` - Receita de exportação
  - `edb10rbCumul` - Receita cumulativa
  - `edb10rbTotal` - Receita total com atividade
  - `edb10rbAtivCP` - Receita sujeita à contribuição
  - `edb10rbAtivSemCP` - Receita sem contribuição

## ⚠️ Regras de Negócio

### Classificação da Receita
- **Classe 1**: Receita tributável
- **Classe 2**: Receita não tributável
- **Classe 3**: Receita de exportação
- **Classe 4**: Receita cumulativa

### Cálculo da Contribuição Previdenciária
- **Agrupamento**: Por atividade econômica (`abg02id`)
- **Base de cálculo**: Receita total da atividade menos exclusões
- **Alíquota**: Específica por atividade
- **Fórmula**: `(RB - Exc) * Alíquota / 100`

### Tratamento de Devoluções
- Consideradas apenas para atividades com contribuição
- Deduzidas do campo `vlExc` (exclusões)
- Não afetam receitas totais, apenas a base de cálculo

### Empresas e Grupos
- **Matriz**: Empresa principal do grupo
- **Filial**: Empresas vinculadas à matriz
- **Grupo Contábil**: Vinculação por empresa e tabela (`EA`)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração.

### `selecionarGCPorEmpresa(Long aac10id, String tabela)`
Retorna o ID do grupo contábil da empresa para uma tabela específica.

### `buscarMatrizEFiliais()`
Busca a matriz e todas as filiais relacionadas à empresa ativa.

### `buscarReceita(Integer mes, Integer ano, Long gcEA)`
Busca receitas classificadas por grupo contábil no período.

### `buscarReceitaP100(Integer mes, Integer ano, String cpoVlRecTotEst, String cpoVlRecAtivEstab, String cpoVlExc, String cpoAliqCont, Long gc)`
Busca receitas para cálculo da P100 (ativações + contribuição).

### `buscarDevolucoesP100(Integer mes, Integer ano, String cpoVlRecTotEst, String cpoVlRecAtivEstab, String cpoVlExc, String cpoAliqCont, Long gc)`
Busca devoluções para ajuste do cálculo da P100.

## 📊 Estrutura de Saída

**Edb10 Atualizado:**
- Campos de receita classificada (`rbTrib`, `rbNaoTrib`, `rbExp`, `rbCumul`)
- Campos consolidados da P100 (`rbTotal`, `rbAtivCP`, `rbAtivSemCP`)
- Lista de empresas com receita (`Edb1001`)
- Detalhamento por atividade (`Edb10011`)

**Edb10011 (Detalhe por Atividade):**
- `ativ` - Atividade econômica (`Abg02`)
- `rb` - Receita bruta da atividade
- `exc` - Exclusões/devoluções
- `aliq` - Alíquota da contribuição
- `cprb` - Contribuição calculada
- `cta` - Conta contábil (`Abc10`)

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas (implícita)

**Módulo:** SGT (Sistema de Gestão Tributária)

## 📝 Observações Técnicas

### Campos JSON da P100
- `vlRecTotEst` - Receita total do estabelecimento
- `vlRecAtivEstab` - Receita com atividade do estabelecimento
- `vlExc` - Exclusões da base de cálculo
- `aliqCont` - Alíquota da contribuição

### Tratamento de Devoluções
- Devoluções são subtraídas das exclusões (`vlExc`)
- Não reduzem a receita bruta total
- Apenas atividades com `abg02id` são ajustadas

### Performance
- Consultas otimizadas com agrupamento no banco
- Uso de `TableMap` para manipulação eficiente de dados
- Processamento em lote para matriz + filiais

### Validações
- Apenas documentos não cancelados (`eaa01cancData IS NULL`)
- Apenas documentos de entrada (`eaa01esMov = 1`) ou devolução (`tipocod = 4`)
- Filtro por grupo contábil (`gc`)

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0