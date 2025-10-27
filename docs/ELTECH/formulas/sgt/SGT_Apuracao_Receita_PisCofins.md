# SGT - Apuração de Receita PIS/COFINS

## 📖 Descrição
Fórmula para apuração de receita e cálculo da Contribuição Previdenciária sobre Receita Bruta (PIS/COFINS), considerando diferentes classificações de receita e atividades econômicas.

## 🎯 Finalidade
Realizar a apuração completa da receita bruta para fins de PIS/COFINS, classificando as receitas por tipo (tributada, não-tributada, exportação, cumulativa) e calculando a base de cálculo da contribuição previdenciária.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria
- Gestores Tributários

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais de entrada/saída
- `Eaa0103` - Itens de documentos fiscais
- `Abb01` - Cabeçalho de documentos
- `Aah01` - Tipos de documento
- `Abb10` - Operações fiscais
- `Aac10` - Empresas
- `Abg02` - Atividades econômicas
- `Abc10` - Plano de contas contábil
- `Edb10` - Período de apuração
- `Edb1001` - Receita por empresa
- `Edb10011` - Detalhe por atividade

**Entidades Envolvidas:**
- `Edb10` - Período de apuração
- `Edb1001` - Receita por empresa
- `Edb10011` - Detalhe por atividade econômica
- `Aac10` - Empresa
- `Abg02` - Atividade econômica
- `Abc10` - Conta contábil

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb10 | Edb10 | Sim | Período de apuração com mês/ano |
| alinApur | String | Sim | Código do alinhamento (0044) |

**Campos do Alinhamento:**
- `vlRecTotEst` - Valor total da receita do estabelecimento
- `vlRecAtivEstab` - Valor da receita da atividade do estabelecimento
- `vlExc` - Valor das exclusões
- `aliqCont` - Alíquota da contribuição

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Seleciona alinhamento de apuração (0044)
- Busca matriz e filiais da empresa ativa
- Obtém campos do alinhamento para cálculo

### 2. **Apuração da Receita por Classificação**
- Busca receitas do período por empresa
- Classifica em 4 categorias:
  - **1 - Tributada**: Receitas sujeitas à tributação
  - **2 - Não-Tributada**: Receitas isentas ou não-tributadas
  - **3 - Exportação**: Receitas de exportação
  - **4 - Cumulativa**: Receitas do regime cumulativo

### 3. **Cálculo da Contribuição Previdenciária (P100)**
- Processa receitas por atividade econômica
- Aplica alíquotas específicas por atividade
- Considera devoluções e exclusões
- Calcula base de cálculo e contribuição

### 4. **Consolidação dos Resultados**
- Totaliza receitas por categoria
- Calcula totais para o período
- Preenche entidades do modelo (Edb10, Edb1001, Edb10011)

## ⚠️ Regras de Negócio

### Classificação de Receitas
- **Classe 1**: Receitas tributadas
- **Classe 2**: Receitas não-tributadas  
- **Classe 3**: Receitas de exportação
- **Classe 4**: Receitas do regime cumulativo

### Cálculos Financeiros
- **Base de Cálculo**: Receita da atividade - Exclusões
- **Contribuição**: Base de cálculo × Alíquota
- **Receita Total**: Soma de todas as receitas do estabelecimento
- **Receita com CP**: Receitas com contribuição previdenciária
- **Receita sem CP**: Receitas sem contribuição previdenciária

### Filtros de Período
- Considera documentos do mês/ano informado
- Apenas documentos não cancelados
- Apenas documentos com indicador EFD Contribuinte = 1
- Operações fiscais do tipo 1 (entrada/saída)

### Tratamento de Devoluções
- Considera operações fiscais do tipo 4 (devolução)
- Ajusta valores de exclusões nas atividades
- Mantém demais dados da atividade original

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração.

### `selecionarGCPorEmpresa(Long aac10id, String tabela)`
Busca o grupo de empresas para a tabela especificada.

### `buscarMatrizEFiliais()`
Retorna lista da matriz e todas suas filiais.

### `buscarReceita(Integer mes, Integer ano, Long gcEA)`
Busca receitas classificadas por tipo.

### `buscarReceitaP100(Integer mes, Integer ano, String... campos)`
Busca receitas para cálculo da contribuição P100.

### `buscarDevolucoesP100(Integer mes, Integer ano, String... campos)`
Busca devoluções para ajuste do cálculo P100.

## 📊 Estrutura de Saída

**Edb10 (Período):**
- `edb10rbTrib` - Receita tributada
- `edb10rbNaoTrib` - Receita não-tributada
- `edb10rbExp` - Receita de exportação
- `edb10rbCumul` - Receita cumulativa
- `edb10rbTotal` - Receita total
- `edb10rbAtivCP` - Receita com contribuição previdenciária
- `edb10rbAtivSemCP` - Receita sem contribuição previdenciária

**Edb1001 (Por Empresa):**
- `edb1001empresa` - Empresa
- `edb1001rb` - Receita bruta da empresa

**Edb10011 (Por Atividade):**
- `edb10011ativ` - Atividade econômica
- `edb10011rb` - Receita bruta
- `edb10011exc` - Exclusões
- `edb10011aliq` - Alíquota
- `edb10011cprb` - Contribuição previdenciária
- `edb10011cta` - Conta contábil

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários e datas
- `multiorm` - Criteria e fields
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema

**Entidades:**
- `FormulaBase` - Classe base para fórmulas
- `TableMap` - Estrutura de dados para resultados
- `Parametro` - Parâmetros de consulta

## 📝 Observações Técnicas

- Utiliza JSON fields para campos dinâmicos (`eaa0103json`)
- Processa matriz e filiais consolidamente
- Suporte a múltiplas atividades econômicas por empresa
- Cálculos com precisão decimal (2 casas)
- Consultas SQL otimizadas com agregações
- Tratamento específico para devoluções
- Alíquotas variáveis por atividade econômica

---

**Última Alteração:** 05/09 às 16:00  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração  
**Versão:** 1.0