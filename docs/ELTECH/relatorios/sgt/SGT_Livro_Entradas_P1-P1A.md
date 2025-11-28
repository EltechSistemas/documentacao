# SGT - Livro de Entradas P1/P1A

## 📖 Descrição
Relatório do livro de entradas fiscais (Modelos P1 e P1A), detalhando notas fiscais de entrada, demonstrativos por estado, resumo por CFOP e alíquota de ICMS. Permite geração de termo de abertura e encerramento do livro e suporta rascunho para pré-visualização.

## 🎯 Finalidade
Fornecer visão detalhada das entradas fiscais da empresa, permitindo controle administrativo e fiscal, com consolidação por estado, CFOP e alíquota de ICMS.

## 👥 Público-Alvo
- Controladoria
- Departamento Fiscal
- Auditoria Interna
- Gestão Financeira
- Diretoria Administrativa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-----------------|
| empresaId | Long | Sim | ID da empresa | Inteiro |
| dataIni | LocalDate | Sim | Data inicial do período | yyyy-MM-dd |
| dataFim | LocalDate | Sim | Data final do período | yyyy-MM-dd |
| livroNum | Integer | Não | Número do livro | Inteiro |
| livroPag | Integer | Não | Número da página inicial | Inteiro |
| modelo | Integer | Sim | Modelo do livro | 0=P1, 1=P1A |
| imprimir | Integer | Sim | Tipo de impressão | 0=Livro, 1=Termo abertura, 2=Termo encerramento |
| rascunho | Boolean | Não | Indica rascunho | true/false |
| comIPI | Boolean | Não | Inclui valores de IPI | true/false |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| obs | Observações da nota fiscal | String |
| aliqIcms | Alíquota de ICMS | BigDecimal |
| bcIcms | Base de cálculo do ICMS | BigDecimal |
| icms | Valor do ICMS | BigDecimal |
| isentasIcms | Valor de ICMS isento | BigDecimal |
| outrasIcms | Outras situações de ICMS | BigDecimal |
| bcIpi | Base de cálculo do IPI | BigDecimal |
| ipi | Valor do IPI | BigDecimal |
| isentasIpi | Valor de IPI isento | BigDecimal |
| outrasIpi | Outras situações de IPI | BigDecimal |
| vlrGia | Valor GIA | BigDecimal |
| icmsST | ICMS Substituição Tributária | BigDecimal |
| estado | UF da nota fiscal | String |
| cfop | Código Fiscal de Operações e Prestações | String |

## 🔄 Fluxo do Processo
1. Carrega parâmetros do relatório e define período e modelo.
2. Inicializa listas e totais.
3. Busca notas fiscais de entrada conforme filtros.
4. Agrupa dados por estado, CFOP e alíquota de ICMS.
5. Calcula totais, subtotais e valores de IPI se aplicável.
6. Se `imprimir != 0`, gera termo de abertura ou encerramento.
7. Monta `TableMapDataSource` para JasperReports.
8. Gera PDF final do relatório.
9. Atualiza dados do livro no sistema caso não seja rascunho.

## ⚠️ Regras de Negócio
- Agrupamento por estado, CFOP e alíquota de ICMS.
- Inclusão de IPI é opcional (`comIPI`).
- Termos de abertura e encerramento seguem regras legais fiscais.
- Rascunho não grava dados do livro no sistema.
- Campos de notas canceladas têm valores zerados.
- Número do livro é sequencial, evitando duplicidade.

## 🎨 Saídas Disponíveis
| Formato | Descrição |
|---------|-----------|
| PDF | Relatório pronto para impressão |