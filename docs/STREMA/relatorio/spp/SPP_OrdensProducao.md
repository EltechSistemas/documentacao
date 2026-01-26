# SPP_OrdensProducao

📖 **Descrição**  
Relatório de Ordens de Produção que apresenta dados de produção (acabamento, romaneio/pesagem, tinturaria única e tinturaria ajustada), com agrupamento de itens por ordem e detalhamento de processos. Possui múltiplos layouts (designs) e sub-relatórios conforme o tipo de relatório selecionado.

---

🎯 **Finalidade**  
Permitir a emissão de relatórios de ordens de produção para diferentes etapas do processo produtivo, oferecendo visualizações específicas conforme o design escolhido e possibilitando agrupamento de itens e processos por ordem.

---

👥 **Público-Alvo**
- Produção
- Controle de Qualidade
- Planejamento
- Expedição
- Engenharia de Processos

---

📊 **Dados e Fontes**

### Tabelas Principais
- **BAB01** – Ordem de Produção
- **ABB01** – Documento (ordem)
- **AAH01** – Tipo de documento
- **ABP20** – Componente da ordem
- **ABM01** – Produto
- **ABP2001** – Processo do componente
- **ABP10** – Processo
- **ABP20011** – Item do processo
- **AAM06** – Unidade de medida
- **BAB0101** – Itens da ordem
- **ABE01** – (Não utilizado diretamente no SQL principal, mas usado no relatório)
- **ABE05** – (Não utilizado diretamente neste relatório)

### Entidades Envolvidas
- **Bab01** – Ordem de Produção
- **Abb01** – Documento
- **Aah01** – Tipo de Documento
- **Abm01** – Produto
- **Aam06** – Unidade de Medida

---

⚙️ **Parâmetros do Relatório**

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|---|---|---|---|---|
| numeroInicial | Integer | Não | Número inicial da ordem | Numérico |
| numeroFinal | Integer | Não | Número final da ordem | Numérico |
| tipoDoc | Lista (Long) | Não | Tipos de documento | IDs (aah01id) |
| desingRelatorio | Integer | Não (default 0) | Define o layout do relatório | 0, 1, 2 ou 3 |
| numAjustada1 | Integer | Não | Ordem ajustada (tinturaria ajustada) | Numérico |
| numAjustada2 | Integer | Não | Ordem ajustada (tinturaria ajustada) | Numérico |
| numAjustada3 | Integer | Não | Ordem ajustada (tinturaria ajustada) | Numérico |
| numAjustada4 | Integer | Não | Ordem ajustada (tinturaria ajustada) | Numérico |

---

📋 **Campos do Relatório**

| Campo | Descrição | Tipo | Origem / Regra |
|---|---|---|---|
| abb01num | Número da ordem | Integer | `abb01num` |
| abb01data | Data da ordem | Date | `abb01data` |
| prodCodigo | Código do produto | String | `produto.abm01codigo` |
| prodNa | Nome do produto | String | `produto.abm01na` |
| niquelina | Niquelina (tinturaria) | Integer | `bab01json ->> 'niquelina'` |
| turbo | Turbo (tinturaria) | Integer | `bab01json ->> 'turbo'` |
| kg | Peso líquido (kg) | BigDecimal | `bab01qt * produto.abm01pesoLiq` |
| controle | Controle (parte do detalhe) | String | Substring do `bab01detProd` |
| normasObs | Observações de normas | String | Substring do `bab01detProd` |
| bab01qt | Quantidade da ordem | BigDecimal | `bab01qt` |
| compDescr | Descrição do componente | String | `componente.abm01descr` |
| aam06codigo | Unidade de medida | String | `aam06codigo` |
| bab0101det | Detalhamento do processo | String | `bab0101det` |
| bab0101qtp | Quantidade do processo | BigDecimal | `SUM(bab0101qtp)` |
| bab0101seq | Sequência do item | Integer | `bab0101seq` |
| ITENS | Descrição do item no relatório | String | `abm01descr` |
| QTDE | Quantidade do item no relatório | BigDecimal | `bab0101qtp` |
| UNIDADE | Unidade do item no relatório | String | `aam06codigo` |
| PROCESSO | Código do processo (map) | String | Mapeado por `codProc` |
| NOMEPROCESSO | Nome do processo | String | `bab0101det` |
| SEQ | Sequência do processo no relatório | Integer | Incremental por ordem |
| key | Chave para subrelatório | Long | `bab01id` |

---

🔄 **Fluxo do Processo**

### 1. Inicialização
- Define valor padrão do filtro:
    - `desingRelatorio = 0`

### 2. Execução do Relatório
- Coleta filtros: número inicial/final, tipoDoc, design do relatório e números ajustados.
- Monta lista de ordens ajustadas (`listaAjustadas`) com até 4 entradas.
- Carrega sub-relatório `SPP_OrdensProducao_S3_subreport1`.

### 3. Seleção do Layout (desingRelatorio)

#### **Design 0 – Acabamento**
- Relatório: `SPP_OrdensProducao_S1`
- Busca dados de acabamento via `buscarDadosAcabamento()`
- Gera PDF com os dados retornados.

#### **Design 1 – Romaneio / Pesagem**
- Relatório: `SPP_OrdensProducao_S2`
- Busca dados via `buscarDadosRomaneioPesagem()`
- Gera PDF com os dados retornados.

#### **Design 2 – Tinturaria Única**
- Relatório: `SPP_OrdensProducao_S3`
- Busca dados cabeçalho via `buscarDadosTinturariaUnica()`
- Monta lista `itensRelatorio` com processos filtrados por `codProc`
- Cria datasource principal e subdatasource (cross table) por `key`
- Gera PDF com sub-relatório.

#### **Design 3 – Tinturaria Ajustada**
- Relatório: `SPP_OrdensProducao_S4`
- Busca dados via `buscarDadosTinturariaAjustada()`
- Busca itens via `buscarItensTinturariaAjustada()`
- Valida que todas ordens possuem o mesmo `turbo` (senão interrompe)
- Monta lista final `listTM` com dados e processos filtrados por `codProc`
- Define parâmetro `TURBO`
- Gera PDF com `listTM`.

---

⚠️ **Regras de Negócio**

### Validações
- `numeroInicial` e `numeroFinal` são aplicados apenas se preenchidos.
- `tipoDoc` é aplicado apenas se informado.
- Em `desingRelatorio = 3`, não é permitido unificar ordens com `turbo` diferente (interrompe).
- `det` (detalhamento do processo) deve existir para ser incluído no relatório.

### Filtros
- Aplica `obterWherePadrao("bab01", "where")` em todas as consultas.
- Filtra processos específicos:
    - `abp10descr = 'PRODUÇÃO GERAL'`
    - `abp20011det = 'SEPARAÇÃO E DISTRIBUIÇÃO PRODUTO INTERMEDIÁRIO'`

### Processamento de Dados
- Em Tinturaria Única e Ajustada, os itens são filtrados por processos pré-definidos (`codProc`).
- Os dados são agrupados por ordem (`abb01num`) e por processo (`bab0101seq`).

---

🎨 **Saídas Disponíveis**

| Formato | Descrição | Método |
|---|---|---|
| PDF | Ordens de Produção conforme layout selecionado | `gerarPDF(nomeRelatorio, dsPrincipal)` ou `gerarPDF(nomeRelatorio, dados)` |

---

🔧 **Dependências**

### Bibliotecas
- `br.com.multitec.utils` – Utilitários
- `sam.server.samdev.relatorio` – Infraestrutura de relatórios
- `sam.server.samdev.utils` – Parâmetros
- `sam.model.entities` – Entidades do sistema

---

📝 **Observações Técnicas**
- Usa SQL nativo com filtros dinâmicos para performance.
- Utiliza JSON para capturar campos como `turbo` e `niquelina`.
- Usa `TableMapDataSource` para sub-relatórios (cross table).
- O relatório possui 4 designs diferentes, cada um com sua consulta e estrutura.
