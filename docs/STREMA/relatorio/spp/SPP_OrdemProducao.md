# SPP_OrdemProducao

📖 **Descrição**  
Relatório de Ordem de Produção que apresenta o detalhamento de ordens de produção (OP) com informações de produto, componentes, quantidade, tipo, cliente, lote, data e dados complementares de transporte/etiqueta/prioridade para impressão e acompanhamento do processo produtivo.

---

🎯 **Finalidade**  
Permitir a geração da Ordem de Produção para controle e conferência do processo produtivo, possibilitando:
- Listagem de OPs conforme filtros de status, número, tipo, entidade, itens, lote e período de emissão
- Detalhamento de componentes por OP e produto principal
- Visualização de informações logísticas e de prioridade definidas no JSON da OP
- Emissão do documento em formato PDF para impressão ou registro interno

---

👥 **Público-Alvo**
- Produção
- Planejamento
- Logística
- Comercial
- Controle de Qualidade
- Faturamento

---

📊 **Dados e Fontes**

### Tabelas Principais
- **BAB01** – Ordem de Produção
- **ABB01** – Documento (central da OP)
- **AAB10** – Usuário/Operador
- **AAH01** – Tipo de documento
- **ABE01** – Entidade (Cliente / Representante)
- **ABE02** – Vínculo de Representante
- **ABP20** – BOM (lista de materiais / componentes)
- **ABM01** – Produto
- **BAB0101** – Itens da OP (componentes)
- **AAM06** – Unidade de Medida

### Entidades Envolvidas
- **Bab01** – Ordem de Produção
- **Abb01** – Documento
- **Abe01** – Entidade (Cliente / Representante)
- **Abm01** – Produto
- **Aam06** – Unidade de Medida
- **Abp20** – BOM (BOM e revisão)
- **Aab10** – Usuário

---

⚙️ **Parâmetros do Relatório**

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|---|---|---|---|---|
| status | Integer | Sim (default 0) | Status da OP | 0 (default), 1, etc. |
| numIni | Integer | Não | Número inicial do documento | Numérico |
| numFim | Integer | Não | Número final do documento | Numérico |
| tipo | Lista (Long) | Não | Tipos de OP | IDs de tipo (aah01id) |
| entidade | Lista (Long) | Não | Entidades (clientes) | IDs de entidade (abe01id) |
| itens | Lista (Long) | Não | Itens/componentes | IDs de produto (abm01id) |
| lotes | Lista (String) | Não | Lotes separados por “;” | Texto separado por “;” |
| dataEmissao | Intervalo de Datas | Não | Período de emissão | Data inicial e final |

**Observação:**
- `status` é definido como 0 por padrão no filtro inicial.
- Os filtros `numIni`, `numFim` e `lotes` só são aplicados quando informados (não zero / não vazios).
- `dataEmissao` é aplicada apenas se ambos os limites (inicial e final) estiverem preenchidos.

---

📋 **Campos do Relatório**

| Campo | Descrição | Tipo | Origem / Regra |
|---|---|---|---|
| OP | Número do documento | Integer | `abb01num` |
| Data | Data de emissão | Date | `abb01data` |
| Data Entrega | Data estimada/definida | Date | `bab01dtE` |
| Representante | Nome do representante | String | `rep0.abe01na` |
| Código Cliente | Código da entidade | String | `ent.abe01codigo` |
| Cliente | Nome do cliente | String | `ent.abe01nome` |
| Qtde | Quantidade | BigDecimal | `bab01qt` |
| Produto | Descrição do produto principal | String | `prod.abm01descr` |
| Código Produto | Código do produto principal | String | `prod.abm01codigo` |
| BOM (Revisão) | Número da BOM / Revisão | String | `abp20bomNumRev`, `abp20bomDtRev` |
| Componente | Código do componente | String | `comp.abm01codigo` |
| Descrição Componente | Descrição do componente | String | `comp.abm01descr` |
| Unidade | Unidade de medida | String | `aam06codigo` |
| Qtde Prevista | Quantidade prevista | BigDecimal | `bab0101qtP` |
| Qtde Atual | Quantidade atual | BigDecimal | `bab0101qtA` |
| Observação | Observação da OP | String | `bab01obs` |
| Detalhe Produto | Detalhe do produto | String | `bab01detProd` |
| Carga | Tipo de carga (traduzido) | String | JSON `bab01json ->> 'carga'` |
| Transporte | Tipo de transporte (traduzido) | String | JSON `bab01json ->> 'transporte'` |
| Etiqueta | Tipo de etiqueta (traduzido) | String | JSON `bab01json ->> 'etiqueta'` |
| Outros Itens | Indica se há outros itens | String | JSON `bab01json ->> 'outros_itens'` |
| Prioridade | Prioridade da OP (traduzido) | String | JSON `bab01json ->> 'prioridade'` |
| Operador | Usuário responsável | String | `aab10user` |

---

🔄 **Fluxo do Processo**

### 1. Inicialização
- Define valores padrão do filtro:
    - `status = "0"`

### 2. Execução do Relatório
- Coleta parâmetros do filtro (com validações de valores vazios/zero).
- Monta caminho da logo do relatório (arquivo `Logo.png` no diretório do pacote).
- Obtém dados da empresa ativa (`AAC10`) e monta o endereço completo para exibição no relatório.
- Adiciona parâmetros do relatório (LOGO e ENDERECO).
- Chama `buscarDados(...)` para coletar dados da base.
- Gera PDF com os dados retornados.

### 3. Busca de Dados
- Monta cláusulas `WHERE` dinamicamente conforme os parâmetros.
- Executa SQL com múltiplos joins, incluindo leitura de campos JSON.
- Traduz valores de JSON (carga, transporte, etiqueta, outros_itens, prioridade) para texto legível.
- Ordena por número do documento e sequência do item.

---

⚠️ **Regras de Negócio**

### Validações
- `status` é obrigatório (mas possui valor padrão).
- `numIni`, `numFim`, `lotes` e `dataEmissao` são opcionais, porém aplicados somente quando preenchidos.
- `tipo`, `entidade` e `itens` são aplicados somente se houver pelo menos um elemento na lista.

### Regras de Filtro
- `whereStatus`: filtra OP por status (`bab01status`).
- `whereNumIni` e `whereNumFim`: filtram intervalo de número do documento.
- `whereTipo`: filtra por tipo de documento (`aah01id`).
- `whereEntidade`: filtra por cliente (`ent.abe01id`).
- `whereItens`: filtra por componente (`comp.abm01id`).
- `whereLotes`: filtra por lote (`bab01lote`).
- `whereEmissao`: filtra por data de emissão (`abb01data`).
- Sempre aplica filtros padrão do sistema via `obterWherePadrao("bab01", "where")`.

### Tradução de Valores JSON
Os campos do JSON `bab01json` são traduzidos conforme regras abaixo:

| Campo JSON | Valores | Resultado |
|---|---|---|
| carga | carga_lenta | CARGA LENTA |
|  | carga_padrao | CARGA PADRÃO |
|  | carga_rapida | CARGA RÁPIDA |
|  | depassivacao | DEPASSIVAÇÃO |
|  | meia_carga | MEIA CARGA |
|  | seguir_observacao | SEGUIR OBSERVAÇÃO |
|  | carga ultra rapida | CARGA ULTRA RÁPIDA |
|  | outro valor | mantém o valor original |
| transporte | correios | CORREIOS |
|  | entrega | ENTREGA |
|  | retira | RETIRA |
|  | transportadora | TRANSPORTADORA |
| etiqueta | personalizada | PERSONALIZADA |
|  | padrao | PADRÃO |
|  | nome_cliente | NOME DO CLIENTE |
|  | lacre_strema | LACRE STREMA |
|  | lacre_expower | LACRE EX-POWER |
| outros_itens | sim | SIM |
|  | nao | NÃO |
| prioridade | padrao | PADRÃO |
|  | baixa | BAIXA |
|  | alta | ALTA (URGENTE) |

---

🎨 **Saídas Disponíveis**

| Formato | Descrição | Método |
|---|---|---|
| PDF | Ordem de Produção em formato de impressão | `gerarPDF()` |

---

🔧 **Dependências**

### Bibliotecas
- `multitec.utils` – Utilitários gerais
- `sam.server.samdev.relatorio` – Infraestrutura de relatórios
- `sam.server.samdev.utils` – Parâmetros
- `sam.model.entities.aa` – Entidade `Aac10` (empresa)

### Recursos
- Logo do relatório (`Logo.png`) localizado no diretório do pacote:  
  `samdev/resources/strema/relatorios/spp/Logo.png`

---

📝 **Observações Técnicas**
- O relatório utiliza SQL nativo com múltiplos joins para melhor performance.
- Campos JSON são lidos diretamente no SQL e traduzidos para valores legíveis.
- O endereço da empresa ativa é montado dinamicamente a partir da entidade `AAC10`.
- O relatório é gerado exclusivamente em PDF.
- O filtro padrão do sistema (`obterWherePadrao`) é aplicado na consulta.

---