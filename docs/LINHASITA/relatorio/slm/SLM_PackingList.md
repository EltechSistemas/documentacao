# SLM – Packing List – Linhasita

## 📖 Descrição
Relatório de **Packing List** que apresenta o detalhamento de romaneios de expedição, agrupando itens por caixa, pedido de venda e cliente, com informações de endereço, pesos líquidos e brutos, quantidade de tubos e dados complementares para impressão e acompanhamento logístico.

## 🎯 Finalidade
Permitir a geração do Packing List para processos de expedição, possibilitando:
- Conferência de romaneios de carga
- Identificação de pedidos de venda vinculados
- Detalhamento de itens por caixa
- Apuração de peso líquido, peso bruto e quantidade
- Emissão de documento logístico padronizado para envio ao cliente

## 👥 Público-Alvo
- Logística
- Expedição
- Faturamento
- Comercial
- Controle de Produção

## 📊 Dados e Fontes

### Tabelas Principais
- BFA01 – Romaneio
- ABB01 – Documento (Romaneio / Pedido de Venda)
- ABE01 – Entidades
- ABE0101 – Endereços da Entidade
- AAG0201 – Municípios
- BFA0101 – Itens do Romaneio
- BFA01011 – Controle de Itens
- EAA0103 – Vínculo Produto
- ABM01 – Produto
- ABM0101 – Dados Complementares do Produto (JSON)
- ABM15 – Caixa / Volume
- ABM70 – Dados de Embalagem
- AAM06 – Unidade de Medida

### Entidades Envolvidas
- Bfa01 – Romaneio
- Abb01 – Documento
- Abe01 – Entidade
- Abm01 – Produto
- Abm15 – Caixa
- Abm70 – Embalagem
- Aam06 – Unidade de Medida

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|----------|------|-------------|-----------|-------------------|
| numRomaneioIni | Integer | Condicional | Número inicial do romaneio | Numérico |
| numRomaneioFim | Integer | Condicional | Número final do romaneio | Numérico |
| numPedVendaIni | Integer | Condicional | Número inicial do pedido de venda | Numérico |
| numPedVendaFim | Integer | Condicional | Número final do pedido de venda | Numérico |
| entidades | Lista (Long) | Não | Lista de entidades (clientes) | IDs de entidade |
| data | Date | Não | Data de emissão do packing list | Data |
| invoice | String | Não | Número da invoice | Texto |
| obs | String | Não | Observações adicionais | Texto |

> **Observação:**  
> É obrigatório informar **ao menos um filtro de romaneio ou pedido de venda**.

## 📋 Campos do Relatório

| Campo | Descrição | Tipo | Origem / Regra |
|------|-----------|------|----------------|
| Romaneio | Número do romaneio | Integer | ABB01 (docRom) |
| Pedido de Venda | Número do pedido de venda | Integer | ABB01 (docScv) |
| Cliente | Nome da entidade | String | ABE01.abe01nome |
| Endereço | Endereço completo do cliente | String | ABE0101 + AAG0201 |
| Caixa | Identificação da caixa | String | ABM15.abm15nome |
| Item Ref | Referência do item | String | ABM0101 JSON |
| Item Cor | Cor do item | String | ABM0101 JSON |
| Peso Líquido | Peso líquido total | BigDecimal | Cálculo por unidade |
| Peso Bruto | Peso bruto total | BigDecimal | Cálculo por unidade |
| Tubos | Quantidade de tubos | Integer | Regra por unidade |
| Página | Número da página | Integer | A cada 35 registros |

## 🔄 Fluxo do Processo

### Validação de Parâmetros
- Inicializa data padrão com a data atual
- Valida se há filtro de romaneio ou pedido de venda informado

### Preparação de Recursos
- Carrega imagens institucionais:
   - Logo Packing List
   - WhatsApp
   - Instagram
   - Website
- Define parâmetros visuais do relatório

### Definição de Cabeçalho
- Define data de emissão
- Define invoice (quando informada)
- Define observações adicionais

### Processamento dos Dados
- Executa consulta SQL com múltiplos joins
- Aplica filtros dinâmicos conforme parâmetros informados
- Considera empresa ativa para produtos
- Agrupa dados por romaneio, pedido, caixa e item

### Pós-processamento
- Exibe o nome da caixa apenas na primeira ocorrência de cada grupo (romaneio + caixa)
- Calcula paginação automática (35 registros por página)

## ⚠️ Regras de Negócio

### Validações
- Obrigatório informar romaneio ou pedido de venda
- Considera apenas registros válidos conforme `obterWherePadrao()`

### Cálculos
- Peso Líquido:
   - Unidade `TB`: quantidade × peso líquido do produto
   - Demais unidades: soma direta da quantidade
- Peso Bruto:
   - Unidade `TB`: quantidade × peso bruto do produto
   - Demais unidades: valor informado no JSON
- Tubos:
   - Unidade `TB`: quantidade utilizada
   - Demais unidades: valor informado no JSON

### Filtros
- Filtro por romaneio
- Filtro por pedido de venda
- Filtro por entidades (clientes)
- Filtros padrão do sistema

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|--------|-----------|--------|
| PDF | Packing List para impressão | gerarPDF() |

## 🔧 Dependências

### Bibliotecas
- multitec.utils – Utilitários gerais
- sam.server.samdev.relatorio – Infraestrutura de relatórios
- sam.server.samdev.utils – Parâmetros
- sam.model – Entidades do sistema

### Recursos
- Imagens institucionais carregadas dinamicamente por caminho de recurso

## 📝 Observações Técnicas
- Utiliza SQL nativo com agregações para performance
- Uso de campos JSON para dados de produto
- Controle manual de paginação (35 registros por página)
- Impressão preparada para layout logístico
- Relatório exclusivo para saída em PDF
