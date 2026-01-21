# SRF_DashboardServlet_FaturamentoTotalComMeta

## 📄 Descrição
O **SRF_DashboardServlet_FaturamentoTotalComMeta** é um componente de dashboard responsável por apresentar o **faturamento líquido mensal** comparado com a **meta definida**, considerando um período móvel dos **últimos 12 meses**, incluindo o mês atual.

O dashboard consolida dados reais de faturamento a partir dos pedidos fiscais e os compara com as metas cadastradas, exibindo essas informações de forma visual no frontend.

---

## 🎯 Finalidade
- Exibir a evolução mensal do faturamento líquido
- Comparar faturamento realizado versus metas definidas
- Apoiar análise gerencial e tomada de decisão
- Monitorar desempenho comercial ao longo do tempo

---

## 👥 Público-Alvo
- Diretoria
- Gestão Comercial
- Controladoria
- Planejamento Estratégico

---

## 📊 Dados e Fontes
- **Metas de Faturamento:** tabela `aba2001` (via JSON)
- **Pedidos / Faturamento:**
    - `eaa01`, `eaa0103`
    - `abb01`, `abb10`
    - `abe01`
    - `aaj15`, `aaj03`
- **Datas:** baseadas na data atual do sistema (`LocalDate.now()`)

Os dados são obtidos por meio de consultas SQL customizadas utilizando `TableMap`.

---

## 🔢 Parâmetros do Processo
Este dashboard **não recebe parâmetros externos**.  
O período é calculado automaticamente com base na data atual:

- Período analisado: **últimos 12 meses**
- Data inicial: primeiro dia do mês de 11 meses atrás
- Data final: último dia do mês atual

---

## 📤 Saídas do Processo
- Componente visual de dashboard
- Retorno em formato JSON (`UiDto`)
- Variáveis enviadas ao frontend:
    - `vlrFaturado` — lista de faturamento mensal
    - `vlrMeta` — lista de metas mensais
    - `meses` — rótulos dos meses (ex.: "Janeiro 2026")

---

## 🔄 Fluxo do Processo
1. Carrega o template HTML do componente do dashboard
2. Determina o período dos últimos 12 meses
3. Busca as metas de faturamento por mês
4. Busca os pedidos e calcula o faturamento líquido mensal
5. Consolida valores mês a mês:
    - Faturamento realizado
    - Meta definida
6. Trata meses sem movimentação com valor zero
7. Substitui variáveis no script do frontend
8. Retorna o componente renderizado via JSON

---

## 📜 Regras de Negócio
- O faturamento líquido considera:
    - Entradas fiscais válidas
    - CFOPs específicos (`124`, `125`, tipo 1)
    - Exclusão de documentos cancelados
    - Ajustes por devoluções (situação `09`)
- Metas são calculadas como:
    - `mercado_livre + meta_faturamento`
- Meses sem faturamento ou meta são exibidos com valor **zero**
- O dashboard sempre exibe **12 meses**, mesmo sem dados

---

## ⚠️ Inconsistências
- Caso não exista meta cadastrada para um mês, o valor será considerado `0`
- Caso não existam pedidos no período, o faturamento será `0`
- Dependência direta da integridade dos dados fiscais e de metas
- Datas inválidas ou ausentes no JSON de metas podem gerar lacunas

---

## 🔗 Dependências
- Infraestrutura de Dashboards SAM
- Framework Spring (`ResponseEntity`, `MediaType`)
- Utilitários `multitec`
- Banco de dados fiscal e comercial
- Template HTML:
    - `SRF_DashboardRecurso_FaturamentoTotalComMeta.html`

---

## 🛠️ Observações Técnicas
- Estende `ServletBase`
- Tipo de dashboard: **Componente**
- Cache configurado para **5 minutos**
- Utiliza `StringSubstitutor` para injeção dinâmica de dados no script
- Mapeamento de meses é feito manualmente para exibição em português
- Código preparado para atualização automática conforme avanço do tempo
