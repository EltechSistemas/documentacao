# SCV_PedidoVenda.md

## 📖 Descrição
Sistema de geração de relatórios de pedidos de venda para o SCV (Sistema de Controle de Vendas) da Strema, com funcionalidades de impressão e envio automático por e-mail.

## 🎯 Finalidade
Gerar relatórios detalhados de pedidos de venda com opções de visualização, exportação e distribuição automatizada para clientes, incluindo controle de estoque e informações fiscais.

## 👥 Público-Alvo
- Departamento Comercial
- Vendedores
- Expedição
- Atendimento ao Cliente
- Gestão Comercial

## ⚙️ Configuração
**Recursos Necessários:**
- Classe `SCV_PedidoVenda` - Relatório de pedidos de venda
- Arquivos de imagem (Logo.png, LogoEx-Power.png)
- Subrelatórios: SCV_PedidoVendaExpedicao_S1, SCV_PedidoVendaExpedicao_S2

**Localização:** `strema/relatorios/scv/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades/Clientes
- `EAA0103` - Itens do pedido
- `ABM01` - Materiais/Produtos
- `AAB10` - Usuários
- `ABE0104` - Classificação de entidades (e-mails)
- `AAC10` - Empresa

**Entidades Envolvidas:**
- `Eaa01` - Documento fiscal
- `Abb01` - Cabeçalho do documento
- `Abe01` - Entidade/Cliente
- `Eaa0103` - Item do pedido
- `Abm01` - Material/Produto
- `Aab10` - Usuário
- `Abe0104` - Classificação de entidade
- `Aac10` - Empresa

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| pedido | Integer | Sim | Tipo de pedido (1-Venda, 2-Expedição, 3-Venda/Expedição) |
| numIni | Integer | Não | Número inicial do pedido |
| numFim | Integer | Não | Número final do pedido |
| tipo | List<Long> | Não | Lista de tipos de documento |
| entidade | List<Long> | Não | Lista de entidades |
| representante | List<Long> | Não | Lista de representantes |
| classeItens | List<Long> | Não | Lista de classes de itens |
| dataEmissao | LocalDate[] | Não | Período de emissão |
| dataEntrega | LocalDate[] | Não | Período de entrega |
| enviaEmail | Boolean | Não | Enviar por e-mail automaticamente |
| tipoSCV2002 | Long | Não | Filtro específico da tela SCV2002 |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF | Relatório formatado | Arquivo |
| E-mail | Pedido enviado por e-mail | Mensagem |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define valores padrão para filtros
   - Carrega logos e recursos visuais
   - Compõe dados da empresa e inscrição estadual

2. **Processamento de Filtros**
   - Aplica filtros de número, tipo, entidade, representante
   - Processa filtros de data de emissão e entrega
   - Aplica filtro específico da tela SCV2002

3. **Busca de Dados**
   - Executa consulta SQL com múltiplos joins
   - Busca view de itens com cálculos de estoque
   - Calcula totais líquidos por item

4. **Geração de Relatório**
   - Seleciona template baseado no tipo de pedido
   - Configura subrelatórios para detalhamento
   - Gera PDF com estrutura complexa

5. **Envio de E-mail (Opcional)**
   - Envia pedido automaticamente por e-mail
   - Aplica formatação e assinatura

## ⚠️ Regras de Negócio

### Tipos de Pedido
- **1 - Venda**: Relatório padrão de pedido de venda
- **2 - Expedição**: Relatório focado em expedição
- **3 - Venda/Expedição**: Relatório combinado

### Filtros e Validações
- Filtro padrão por operação diferente de 46045860
- Apenas documentos de classificação 0 (venda)
- Apenas movimentações de saída (esMov = 1)
- Documentos não cancelados
- Endereços principais apenas

### Cálculos de Estoque
- **Total Saldo**: Quantidade em estoque
- **Total Vendido**: Quantidade em pedidos de venda
- **Total Compra**: Quantidade em pedidos de compra
- **Total Produção**: Quantidade em produção
- **Total Líquido**: Saldo - Vendido + Compra + Produção

## 🎨 Estrutura do Relatório

**Cabeçalho:**
- Logos da empresa
- Dados cadastrais da empresa (endereço, CNPJ, IE)
- Filtros aplicados

**Detalhes do Pedido:**
- Número, data, cliente
- Endereço de entrega completo
- Condição de pagamento
- Representante comercial

**Itens do Pedido:**
- Código, descrição, quantidade, unidade, unitário, total
- Tributos (ICMS, ST, IPI, alíquotas)
- Informações de frete

**Subrelatórios:**
- **S1**: Detalhes adicionais do pedido
- **S2**: Informações complementares

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Critérios e joins
- `multitec.utils` - Utilitários e e-mail
- `jasperreports` - Geração de relatórios
- `javax.mail` - Envio de e-mail

**Serviços:**
- `CAS1010Service` - Processamento de assinaturas de e-mail

**Consultas:**
- Busca de pedidos com múltiplos relacionamentos
- View de itens com cálculos de estoque
- E-mails de destino por classificação
- Configuração de e-mail do usuário

## 📝 Observações Técnicas

- **Recursos Visuais**: Logos carregadas dinamicamente do filesystem
- **Formatação de Dados**: CEP formatado (XXXXX-XXX), endereços concatenados
- **Subrelatórios**: Estrutura complexa com múltiplos níveis de detalhe
- **Cálculos em Tempo Real**: Totais líquidos calculados dinamicamente
- **Segurança**: Aplicação de where padrão em todas as consultas

## 🔄 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo do relatório.

### `buscarDados()`
Executa consulta principal com todos os filtros aplicados.

### `buscarViewItens()`
Cria view complexa com cálculos de estoque.

### `buscarValorLiqTotal()`
Calcula total líquido por item baseado na view.

### `enviarEmail()`
Processa envio automático do pedido por e-mail.

### `buscarEmailDestino()`
Obtém lista de e-mails de destino baseado na classificação.

## 💡 View de Itens
A view calcula:
- **Saldo em estoque**
- **Quantidade vendida** (pedidos de venda não atendidos)
- **Quantidade comprada** (pedidos de compra não atendidos)
- **Quantidade em produção** (ordens de produção pendentes)
- **Saldo líquido** = Saldo - Vendido + Comprado + Produção

## ⚠️ Validações de E-mail
- Usuário deve ter e-mail cadastrado (AAB1008)
- Entidade deve ter contato classificado como "9001"
- Assinatura opcional do usuário
- Tratamento de caracteres especiais no corpo
- Validação de existência de destinatários

## 💰 Template de E-mail
**Assunto**: "PEDIDO N° [número] STREMA BATERIAS – APROVADO"

**Corpo**: