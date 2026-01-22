# SPP_ListaOndeUsaComponentes

## 📖 Descrição
Classe `SPP_ListaOndeUsaComponentes` do sistema Eltech, responsável por gerar relatórios do tipo **“Onde Usa” Componentes**, mostrando em quais produtos ou fórmulas um determinado componente é utilizado.

Este relatório permite identificar relações entre componentes e produtos finais, auxiliando em planejamento de produção, estoque e rastreabilidade de componentes.

---

## 🎯 Finalidade
- Listar todos os componentes dentro de um intervalo específico (`itemIni` a `itemFim`);
- Mostrar para cada componente em quais fórmulas/produtos ele é utilizado;
- Detalhar quantidade, unidade de medida, perda, sequência na fórmula;
- Permitir filtragem por tipo de componente (M, P, S);
- Exportar o relatório em **PDF** ou **XLSX**.

---

## 👥 Público-Alvo
- Produção e Planejamento de Manufatura (SPP)
- Engenharia de Produto e Processos
- Controle de Estoque e Logística

---

## ⚙️ Configuração
**Classe Principal:** `SPP_ListaOndeUsaComponentes`  
**Pacote:** `eltech.relatorios.spp`

**Dependências:**
- `sam.server.samdev.relatorio` – Framework de relatórios
- `multitec.utils` – Utilitários para manipulação de mapas e filtros
- Entidades SAM: `TableMap`, `Parametro`, `Aam06`

---

## 📊 Dados e Fontes
**Tabelas principais consultadas:**
- `ABM01` – Itens/componentes
- `AAM06` – Unidade de medida
- `ABP20` – Fórmulas ou produtos finais
- `ABP2001` / `ABP20011` – Componentes da fórmula, sequência, quantidade e perda

---

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| itemIni | String | Não | Código inicial do componente |
| itemFim | String | Não | Código final do componente |
| mps | List<Integer> | Não | Tipo do componente (0 = M, 1 = P, outro = S) |
| impressao | Integer | Não | Define saída: 0 = PDF (default), 1 = XLSX |

---

## 📋 Saídas do Processo

| Campo | Descrição |
|-------|-----------|
| codComponente | Código do componente |
| naComponente | Nome do componente |
| MpsComponente | Tipo do componente (M, P ou S) |
| umuComponente | Unidade de medida do componente |
| formula | Código da fórmula ou produto final que utiliza o componente |
| sequencia | Sequência do componente na fórmula |
| quantidade | Quantidade do componente na fórmula |
| perda | Percentual de perda do componente na fórmula |
| codPrincipal | Código do item principal/produto final |
| naPrincipal | Nome do item principal/produto final |
| umuPrincipal | Unidade de medida do item principal |
| MpsPrincipal | Tipo do item principal (M, P ou S) |

**Saída final:** PDF ou XLSX do relatório, contendo todos os dados filtrados.

---

## 🔄 Fluxo do Processo

1. **Criação de valores iniciais (`criarValoresIniciais`)**
   - Define filtro padrão `impressao = 0` (PDF)

2. **Execução do relatório (`executar`)**
   - Coleta filtros do usuário:
      - Código inicial e final do componente
      - Lista de tipos de componentes (M/P/S)
      - Tipo de impressão (PDF/XLSX)
   - Busca dados detalhados via `buscarDadosRelatorio()`
   - Adiciona parâmetro `empresa` no relatório (`AAC10`)
   - Gera saída:
      - `impressao = 1` → XLSX
      - `impressao != 1` → PDF

3. **Busca de dados (`buscarDadosRelatorio`)**
   - Monta SQL dinâmico para buscar todos os componentes dentro do intervalo e filtros de tipo
   - Faz joins entre:
      - Componente (`ABM01`)
      - Produto/fórmula (`ABP20`)
      - Sequência e quantidade na fórmula (`ABP2001`, `ABP20011`)
      - Unidade de medida (`AAM06`)
   - Ordena resultados por tipo, componente, produto/fórmula e sequência
   - Retorna lista de `TableMap` pronta para relatório

---

## ⚠️ Regras de Negócio

- Componente é filtrado apenas se `itemIni` e `itemFim` estiverem definidos
- Lista de tipos `mps` permite filtrar por:
   - `0` → M (Matéria-prima)
   - `1` → P (Produto intermediário)
   - outro → S (Semiacabado ou produto final)
- Se `mps` contém `-1` ou está vazio, não filtra por tipo
- Relatório gera **PDF** por padrão, ou **XLSX** se `impressao = 1`
- Ordenação final do relatório:
   - Tipo do componente → Código do componente → Tipo do produto final → Código do produto final → Fórmula

---

## 🔄 Métodos Principais

### `executar()`
- Coleta filtros
- Chama `buscarDadosRelatorio()`
- Define parâmetro `empresa`
- Gera relatório PDF ou XLSX

### `buscarDadosRelatorio(codItemIni, codItemFin, mps)`
- Monta SQL dinâmico com filtros opcionais
- Realiza joins entre componentes, fórmulas, sequência, unidades
- Retorna lista de `TableMap` com todos os campos calculados

### `criarValoresIniciais()`
- Define valor padrão do filtro de impressão (`impressao = 0`)

---

## 💡 Observações Técnicas
- Suporte a filtros dinâmicos de intervalo e tipo
- Usa `TableMap` para armazenamento de resultados
- Formata tipo de componente com CASE (`M`, `P`, `S`)
- Permite exportação para PDF ou XLSX diretamente do SAM Framework