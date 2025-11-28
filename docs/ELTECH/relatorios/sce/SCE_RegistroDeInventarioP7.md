# SCE - Registro de Inventário Modelo P7

## 📖 Descrição
Relatório de registro de inventário (Modelo P7) que detalha itens por grupo e grau, incluindo totais, médias e unidades de medida. Permite geração de termo de abertura e encerramento do livro.

## 🎯 Finalidade
Fornecer visão detalhada do inventário, permitindo controle administrativo e fiscal, com consolidação por grupos, graus e totais.

## 👥 Público-Alvo
- Controladoria
- Departamento Fiscal
- Auditoria Interna
- Gestão de Estoque
- Diretoria Administrativa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-----------------|
| grupos | List<Long> | Não | IDs de grupos de inventário | Lista de IDs |
| livroNum | Integer | Sim | Número do livro | Inteiro |
| livroPag | Integer | Sim | Número da página inicial | Inteiro |
| impressao | Integer | Sim | Tipo de impressão | 0=Página, 1=Folha |
| imprimir | Integer | Sim | Formato de impressão | 0=Livro, 1=Termo abertura, 2=Termo encerramento |
| resumo | Integer | Não | Nivel de resumo dos grupos | 0=Nenhum, 1=Grau1, 2=Grau2 |
| totUniMed | Boolean | Não | Totalizar unidades de medida | true/false |
| totQtd | Boolean | Não | Totalizar quantidade | true/false |
| inventario | Long | Sim | ID do inventário | Inteiro |
| rascunho | Boolean | Não | Indica rascunho | true/false |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abm40codigo | Código do grupo/registro | String |
| abm40descr | Descrição do grupo | String |
| bcb11qtde | Quantidade do item | BigDecimal |
| bcb11total | Total do item | BigDecimal |
| descTotComp | Total do grupo | BigDecimal |
| totGrau1 / totGrau2 | Totais dos graus | BigDecimal |
| mediaGrau1 / mediaGrau2 | Média por grau | BigDecimal |
| totComp | Total geral do grupo | BigDecimal |
| descrUniMed | Unidade de medida | String |
| qtdUniMed | Quantidade por unidade | BigDecimal |
| totUniMed | Total por unidade | BigDecimal |

## 🔄 Fluxo do Processo
1. Carrega parâmetros e ano de referência.
2. Inicializa totais e listas de dados.
3. Busca os itens por grupo e inventário.
4. Agrupa por grau1, grau2 e grupo, calculando totais e médias.
5. Gera linhas de resumo, totais e unidades de medida.
6. Se `imprimir != 0`, gera termo de abertura ou encerramento.
7. Monta `TableMapDataSource` para JasperReports.
8. Gera PDF com o relatório e informações adicionais.

## ⚠️ Regras de Negócio
- Agrupamento por códigos de 2 e 4 caracteres (grau1 e grau2).
- Totalização condicional por unidade de medida.
- Cálculo de médias apenas se quantidade > 0.
- Termos de abertura e encerramento seguem regras legais (Lei 6374/89, Convênio 57/95).
- Inclusão de resumo por grupos se parâmetro `resumo` for diferente de 0.

## 🎨 Saídas Disponíveis
| Formato | Descrição |
|---------|-----------|
| PDF | Relatório pronto para impressão |

## 🔧 Dependências
- `sam.server.samdev.relatorio.RelatorioBase` — Classe base para relatórios
- `sam.server.samdev.relatorio.TableMapDataSource` — Fonte de dados para JasperReports
- `br.com.multitec.utils.collections.TableMap` — Estrutura de dados
- Entidades: `Abm40`, `Abm01`, `Bcb10`, `Bcb11`, `Aac10`, `Aag02`, `Aag0201`
- JasperReports (`JasperReport`, `JasperPrint`)
- Utilitários para datas e manipulação de mapas (`MDate`, `Utils`)
