# SCF_DRE

## 📖 Descrição
Classe `SCF_DRE` do sistema **Foco**, responsável por gerar o relatório de **Demonstração do Resultado do Exercício (DRE)**.

O relatório consolida vendas, custos, despesas, impostos, margem e resultados financeiros, permitindo visão detalhada ou resumida do desempenho financeiro da empresa em um período definido.

---

## 🎯 Finalidade
- Gerar DRE consolidado para a empresa ativa;
- Permitir detalhamento de dados por categorias: vendas, custos, despesas operacionais, impostos, ativo imobilizado e informações adicionais;
- Calcular margens e percentuais sobre vendas;
- Exportar relatório em **PDF** ou **XLSX**.

---

## 👥 Público-Alvo
- Financeiro e Contabilidade;
- Controladoria e Planejamento Financeiro;
- Diretores e Gerentes para tomada de decisão estratégica.

---

## ⚙️ Configuração
**Classe Principal:** `SCF_DRE`  
**Pacote:** `foco.relatorios.scf`

**Dependências:**
- `br.com.multitec.utils.DateUtils` – Utilitários de datas;
- `br.com.multitec.utils.Utils` – Manipulação de mapas;
- `sam.server.samdev.relatorio` – Framework de relatórios;
- `sam.model.entities.aa.Aac10` – Entidade de empresa ativa;
- `TableMap` e `TableMapDataSource` – Estruturas para organizar dados do relatório;

---

## 📊 Dados e Fontes
O relatório é construído a partir de múltiplas consultas a banco de dados:

| Categoria | Origem / Tabelas |
|-----------|----------------|
| Vendas | `dadosVendas` |
| Custos | `dadosCustos` |
| Margem de Lucro Bruta | Calculada: vendas - custos |
| Outras Entradas | `dadosOutrasEntradas`, `dadosOutrasEntradas2` |
| Despesas Operacionais | `dadosCustoFixo`, `dadosDeptoPessoal`, `dadosRefeitorio`, `dadosDespeBancarias`, `dadosImpostos` |
| Ativo Imobilizado | `buscarDadosAtivoImob`, `buscarDadosAtivoImob2` |
| Resultados Finais | Lucro líquido operacional, distribuição de lucros, lucro líquido |
| Informações Adicionais | `dadosInfoAdd` |

---

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| emissao | Intervalo de datas | Não | Período de emissão do relatório (início/fim do mês padrão) |
| impressao | Integer | Não | 0 = PDF (default), 1 = XLSX |
| detalhamento | Integer | Não | 0 = Detalhado, 1 = Resumido |

---

## 📋 Saídas do Relatório

| Campo | Descrição |
|-------|-----------|
| key | Identificador para vincular sub-relatórios |
| %venda | Percentual da categoria em relação ao total (vendas ou margem) |
| valorMargem | Margem de lucro por categoria |
| total | Valor agregado da categoria |
| totalDespesas | Soma de todas as despesas operacionais |
| lucroLiqOpe | Lucro líquido operacional |
| distribLucro | Distribuição de lucros |
| lucroLiq | Lucro líquido final (operacional + outras entradas + ativo imobilizado) |

**Sub-relatórios:**  
O relatório principal (`dsPrincipal`) possui 13 sub-relatórios (StreamSub1 a StreamSub13) com dados detalhados de cada categoria:

1. `dadosVendas`
2. `dadosCustos`
3. `dadosMargem`
4. `dadosOutrasEntradas`
5. `dadosCustoFixo`
6. `dadosDeptoPessoal`
7. `dadosRefeitorio`
8. `dadosDespeBancarias`
9. `dadosImpostos`
10. `dadosTotalDespesas`
11. `dadosAtivoImob`
12. `resultado`
13. `dadosInfoAdd`

---

## 🔄 Fluxo do Processo

1. **Criação de valores iniciais (`criarValoresIniciais`)**
    - Define filtros padrão:
        - `emissao` → intervalo do mês atual
        - `impressao = 0` (PDF)
        - `detalhamento = 0` (detalhado)

2. **Execução do relatório (`executar`)**
    - Coleta filtros do usuário (`emissao`, `detalhamento`, `impressao`)
    - Obtém empresa ativa (`Aac10`) e adiciona parâmetro `PERIODO` para exibição no relatório
    - Inicializa lista principal de dados (`dados`)
    - Se detalhamento = 0:
        - Busca dados detalhados de todas as categorias
        - Calcula totais e percentuais (% sobre vendas ou margem)
        - Consolida dados duplicados de segunda consulta para cada categoria
        - Ordena dados por nome ou identificador (`abf11nome`, `abf11id`)

3. **Cálculos principais**
    - **Margem de Lucro Bruta:** vendas - custos
    - **Despesas:** consolidação de custos fixos, pessoal, refeitório, despesas bancárias, impostos
    - **Total Despesas:** soma de todas as despesas operacionais
    - **Ativo Imobilizado:** soma das duas fontes (`ativoImob`, `ativoImob2`)
    - **Resultados:**
        - Lucro líquido operacional
        - Distribuição de lucros
        - Lucro líquido final (incluindo outras entradas e ativo imobilizado)
    - **Informações adicionais:** percentuais calculados sobre total

4. **Geração do relatório**
    - Cria `TableMapDataSource` principal
    - Adiciona todos os sub-relatórios
    - Define parâmetros de streams de cada sub-relatório (`StreamSub1` a `StreamSub13`)
    - Saída:
        - `impressao = 1` → XLSX
        - caso contrário → PDF

---

## ⚠️ Regras de Negócio

- Percentual de cada categoria é calculado sobre o total de vendas ou margem;
- Dados de segunda consulta (duplicados) são consolidados para evitar contagem dupla;
- Ordenação de dados é realizada por nome (`abf11nome`) ou ID (`abf11id`) quando necessário;
- Relatório suporta detalhamento ou versão resumida;
- Sub-relatórios permitem modularização de dados e melhor performance na geração de PDF/XLSX.

---

## 💡 Observações Técnicas

- Estrutura modular: dados separados por categoria para facilitar manutenção e validação;
- Uso extensivo de `TableMap` para representar registros;
- Percentuais e totais são calculados dinamicamente;
- Integração com SAM Framework para geração de PDF/XLSX e streams de sub-relatórios;
- Possui tratamento de dados nulos e consolidação de valores entre consultas.  
