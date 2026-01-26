# SCV_PedidoVenda.md

📖 **Descrição**  
Relatório de Pedido de Venda (SCV) que reúne informações de pedidos e itens, incluindo dados do cliente, endereço, frete, impostos, valores, status, representante e itens do pedido. Permite também envio automático por e-mail e marcação de impressão no sistema.

---

🎯 **Finalidade**  
Permitir a geração e impressão do Pedido de Venda ou Pedido de Expedição (ou ambos), com detalhamento completo de itens e valores, além de possibilitar o envio por e-mail e controle de impressão do documento.

---

👥 **Público-Alvo**
- Comercial
- Faturamento
- Expedição
- Financeiro
- Atendimento ao Cliente

---

📊 **Dados e Fontes**

### Tabelas Principais
- **EAA01** – Pedido de Venda
- **ABB01** – Documento
- **ABB10** – Operador (usuário)
- **ABE01** – Entidade (Cliente / Representante / Despacho)
- **EAA0101** – Endereços do pedido
- **EAA0102** – Informações de despacho/frete
- **EAA0103** – Itens do pedido
- **ABM01** – Produto
- **AAM01** – Classe do item
- **AAM06** – Unidade de medida
- **ABE30** – Centro de Produção (CP)
- **EAA01 JSON** – Informações de modelo e impostos
- **EAA0103 JSON** – Impostos por item
- **ABE0104** – Emails do cliente
- **AAC10** – Empresa ativa
- **AAC1002** – Inscrição Estadual da empresa
- **ABE05** – Vínculo de representante com usuário
- **BCC02 / BCC0201** – Saldos (para cálculo de total líquido)
- **BAB01 / ABP20** – Produção (para cálculo de total líquido)

### Entidades Envolvidas
- **Eaa01** – Pedido de Venda
- **Abb01** – Documento
- **Abe01** – Entidade
- **Abm01** – Produto
- **Aam06** – Unidade de Medida
- **Aab10** – Usuário
- **Aab1008** – E-mail do usuário (para envio)
- **Aac10** – Empresa
- **Aac1002** – IE da empresa
- **Abe0104** – Emails do cliente

---

⚙️ **Parâmetros do Relatório**

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|---|---|---|---|---|
| pedido | Integer | Sim (default 2) | Tipo de relatório | 1 = Pedido de Venda / 2 = Pedido de Expedição / 3 = Ambos |
| numIni | Integer | Não | Número inicial do pedido | Numérico |
| numFim | Integer | Não | Número final do pedido | Numérico |
| tipo | Lista (Long) | Não | Tipo de documento | IDs (aah01id) |
| entidade | Lista (Long) | Não | Cliente | IDs (abe01id) |
| representante | Lista (Long) | Não | Representante | IDs (abe01id) |
| classeItens | Lista (Long) | Não | Classe do item | IDs (aam01id) |
| dataEmissao | Intervalo de Datas | Não | Período de emissão | Data inicial e final |
| dataEntrega | Intervalo de Datas | Não | Período de entrega | Data inicial e final |
| enviaEmail | Boolean | Não | Envia e-mail ao cliente | true / false |
| tipoSCV2002 | Long | Não | Filtro específico (tela SCV200289) | ID do tipo |

**Observação:**
- O filtro de **representante** é preenchido automaticamente com base no usuário logado caso não seja informado.
- `pedido` define qual relatório Jasper será utilizado.

---

📋 **Campos do Relatório**

| Campo | Descrição | Tipo | Origem / Regra |
|---|---|---|---|
| eaa01id | ID do pedido | Long | `eaa01id` |
| Pedido | Número do pedido | Integer | `abb01num` |
| Data | Data de emissão | Date | `abb01data` |
| Cliente | Nome do cliente | String | `ent.abe01nome` |
| Código Cliente | Código do cliente | String | `eaa0102codigo` |
| Endereço | Endereço completo do cliente | String | `eaa0101endereco`, `eaa0101complem`, `aag0201nome`, `eaa0101bairro`, `aag02uf`, `eaa0101cep` |
| Contato | Telefone/Email do cliente | String | `eaa0101ddd`, `eaa0101fone`, `eaa0101email` |
| CP | Centro de Produção | String | `abd01descr` |
| Item | Código do item | String | `abm01codigo` |
| Nome Item | Nome do item | String | `abm01na` |
| Descrição | Descrição do item | String | `abm01descr` |
| Qtde Comercial | Quantidade comercial | BigDecimal | `eaa0103qtComl` |
| Unidade | Unidade de medida | String | `aam06codigo` |
| Unitário | Valor unitário | BigDecimal | `eaa0103unit` |
| Total Itens | Total do pedido | BigDecimal | `eaa01totItens` |
| Total Documento | Total do documento | BigDecimal | `eaa01totDoc` |
| ICM (item) | Valor do ICM do item | BigDecimal | `eaa0103json ->> 'icm_icm'` |
| ST (item) | Valor do ST do item | BigDecimal | `eaa0103json ->> 'st_icm'` |
| IPI (item) | Valor do IPI do item | BigDecimal | `eaa0103json ->> 'ipi_ipi'` |
| ICMS Aliq | Aliquota de ICMS | BigDecimal | `eaa0103json ->> 'icm_aliq'` |
| IPI Aliq | Aliquota de IPI | BigDecimal | `eaa0103json ->> 'ipi_aliq'` |
| Total Item | Total do item | BigDecimal | `eaa0103total` |
| Total IPI | Total IPI do pedido | BigDecimal | `eaa01json ->> 'ipi_ipi'` |
| Total ICMS | Total ICMS do pedido | BigDecimal | `eaa01json ->> 'icm_icm'` |
| Total ST | Total ST do pedido | BigDecimal | `eaa01json ->> 'st_icm'` |
| Frete Dest | Valor do frete (destinatário) | BigDecimal | `eaa01json ->> 'vlr_frete_dest'` |
| Modelo Etiqueta | Modelo da etiqueta (traduzido) | String | `eaa01json ->> 'modelo'` com tradução |
| CP (nome) | Nome do CP | String | `abe30nome` |
| Despacho | Nome do despacho com CIF/FOB | String | `despachoAbe01nome` |
| Observações Gerais | Observações do pedido | String | `eaa01obsGerais` |
| Observações Internas | Observações internas | String | `eaa01obsUsoint` |
| Representante | Nome do representante | String | `representante` (fallback NEUZA SANCHES) |
| Email Representante | E-mail do representante | String | `emailRep` |
| totalLiqItem | Total líquido do item (calculado) | BigDecimal | Calculado via `buscarViewItens()` |
| eaa0101emailFinanc | Email financeiro | String | `buscarEnderecoFinanc(eaa0101id)` |

---

🔄 **Fluxo do Processo**

### 1. Inicialização
- Define valor padrão do filtro:
   - `pedido = 2` (Pedido de Expedição)

### 2. Execução do Relatório
- Coleta os filtros do usuário (número, tipo, cliente, representante, classe, datas, etc.).
- Se representante não for informado, busca representantes vinculados ao usuário logado.
- Monta caminhos das logos do relatório.
- Busca dados da empresa ativa (`AAC10`) e IE (`AAC1002`).
- Configura parâmetros do relatório (logo, endereço, contato, etc.).
- Busca dados do pedido e view de itens (para cálculo de total líquido).
- Para cada registro, preenche:
   - `eaa0101emailFinanc`
   - `totalLiqItem`
   - `key` (para subrelatórios)
- Define qual arquivo Jasper será usado (`SCV_PedidoVenda`, `SCV_PedidoExpedicao` ou `SCV_PedidoVendaExpedicao`).
- Cria `TableMapDataSource` principal e subdatasources.
- Se `enviaEmail = true`, envia e-mail com PDF anexo.
- Marca status de impressão do pedido.
- Gera PDF.

### 3. Busca de Dados
- Monta filtros dinâmicos conforme parâmetros informados.
- Executa SQL com múltiplos joins e leitura de JSON.
- Aplica filtros padrões do sistema (`getSamWhere().getWherePadrao`).
- Ordena por número do pedido, ID e código do item.

### 4. Envio de Email
- Carrega relatório Jasper e processa PDF.
- Busca emails do cliente (ABE0104 com código 9001).
- Valida email do usuário logado.
- Anexa PDF e envia com corpo padronizado.
- Adiciona assinatura caso exista.

### 5. Atualização de Status
- Marca `eaa01statusImpr = 2` para os pedidos processados.

---

⚠️ **Regras de Negócio**

### Validações
- `pedido` é obrigatório (possui default 2).
- `numIni` e `numFim` são aplicados apenas se preenchidos.
- `tipo`, `entidade`, `representante`, `classeItens` são aplicados apenas se existirem itens na lista.
- `dataEmissao` e `dataEntrega` são aplicados apenas se preenchidos.
- Se `enviaEmail = true` e não houver dados, interrompe com erro.
- Se o usuário não possuir e-mail cadastrado, interrompe com erro.
- Se não houver e-mail do cliente, interrompe com erro.

### Filtros
- Filtros padrão do sistema aplicados via `getSamWhere().getWherePadrao("where", Eaa01.class)`
- Exclui registros de usuário específico: `abb10id <> 46045860`
- Apenas documentos de classe 0 e movimento 1, e endereço principal.

### Cálculos
- `totalLiqItem` é calculado com base em uma view com saldo, vendas, compras e produção:
   - totalSaldo
   - totalVendido
   - totalCompra
   - totalProducao
   - totalLiqItem = saldo - vendido + compra + produção

### Tradução de Valores JSON
- `modelo` traduzido:
   - `nome_cliente` → **NOME DO CLIENTE**
   - `lacre_strema` → **LACRE STREMA**
   - `lacre_expower` → **LACRE EX-POWER**
   - `seguir_observacao` → **SEGUIR OBSERVADOR**
   - outros valores → mantêm o original

---

🎨 **Saídas Disponíveis**

| Formato | Descrição | Método |
|---|---|---|
| PDF | Pedido de Venda/Expedição em formato de impressão | `gerarPDF(nomeRelatorio, dsPrincipal, "pedido", true)` |

---

🔧 **Dependências**

### Bibliotecas
- `br.com.multiorm.criteria` – Critérios e joins (DAO)
- `br.com.multitec.utils` – Utilitários e e-mail
- `sam.server.samdev.relatorio` – Infraestrutura de relatórios
- `sam.server.samdev.utils` – Parâmetros
- `sam.model.entities` – Entidades do sistema
- `net.sf.jasperreports.engine` – JasperReports
- `javax.mail.util.ByteArrayDataSource` – Anexo PDF
- `java.text.Normalizer` – Normalização de texto
- `java.time.LocalDate` – Datas

### Recursos
- Logos do relatório:
   - `Logo.png`
   - `LogoEx-Power.png`  
     (localizados em `samdev/resources/strema/relatorios/scv/`)

---

📝 **Observações Técnicas**
- Usa critérios e SQL nativo para performance.
- Usa JSON em colunas `eaa01json` e `eaa0103json`.
- Usa subrelatórios com datasource por `key`.
- Envio de e-mail com PDF anexado e assinatura (CAS1010).
- Marca status de impressão no banco (campo `eaa01statusImpr`).
- Inclui fallback de representante quando não informado.