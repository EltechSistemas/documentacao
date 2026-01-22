# SAP SLM – Resumo Packing List por Palete – Linhasita

## 📖 Descrição
Relatório de **Resumo de Packing List por Palete**, que apresenta a consolidação dos itens expedidos agrupados por palete e caixa, com detalhamento de romaneios, pedidos de venda, clientes, pesos líquidos, pesos brutos e quantidade de tubos.

## 🎯 Finalidade
Permitir a análise resumida da expedição por palete, possibilitando:
- Conferência logística por palete
- Visualização consolidada de romaneios e pedidos de venda
- Controle de volumes, pesos e quantidades
- Apoio à expedição, transporte e conferência de carga

## 👥 Público-Alvo
- Logística
- Expedição
- Transporte
- Controle de Produção
- Faturamento

## 📊 Dados e Fontes

### Tabelas Principais
- BFA01 – Romaneio
- ABB01 – Documento (Romaneio / Pedido de Venda)
- ABE01 – Entidades
- BFA0101 – Itens do Romaneio
- BFA01011 – Controle de Itens
- EAA0103 – Vínculo Produto
- ABM01 – Produto
- ABM0101 – Dados Complementares do Produto (JSON)
- ABM15 – Palete / Caixa
- ABM70 – Dados de Embalagem
- AAM06 – Unidade de Medida

### Entidades Envolvidas
- Aac10 – Empresa
- Aag0201 – Município
- Aag02 – UF
- Bfa01 – Romaneio
- Abb01 – Documento
- Abe01 – Entidade
- Abm01 – Produto
- Abm15 – Palete / Caixa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|----------|------|-------------|-----------|-------------------|
| numRomaneioIni | Integer | Condicional | Número inicial do romaneio | Numérico |
| numRomaneioFim | Integer | Condicional | Número final do romaneio | Numérico |
| numPedVendaIni | Integer | Condicional | Número inicial do pedido de venda | Numérico |
| numPedVendaFim | Integer | Condicional | Número final do pedido de venda | Numérico |
| entidades | Lista (Long) | Não | Lista de entidades (clientes) | IDs de entidade |

> **Observação:**  
> É obrigatório informar **ao menos um filtro de romaneio ou pedido de venda**.

## 📋 Campos do Relatório

| Campo | Descrição | Tipo | Origem / Regra |
|------|-----------|------|----------------|
| Romaneio | Número do romaneio | Integer | ABB01 (docRom) |
| Pedido de Venda | Número do pedido de venda | Integer | ABB01 (docScv) |
| Cliente | Nome da entidade | String | ABE01.abe01nome |
| Palete | Identificação do palete | String | ABM15 (ctrl2) |
| Caixa | Identificação da caixa | String | ABM15 (ctrl1) |
| Item Ref | Referência do item | String | ABM0101 JSON |
| Item Cor | Cor do item | String | ABM0101 JSON |
| Peso Líquido | Peso líquido total | BigDecimal | Regra por unidade |
| Peso Bruto | Peso bruto total | BigDecimal | Regra por unidade |
| Tubos | Quantidade de tubos | Integer | Regra por unidade |

## 🔄 Fluxo do Processo

### Validação de Parâmetros
- Valida se há filtro de romaneio ou pedido de venda informado

### Preparação de Recursos
- Carrega logotipo institucional da Linhasita
- Define parâmetros visuais do relatório

### Definição de Dados da Empresa
- Obtém empresa ativa
- Carrega endereço, contato e dados fiscais
- Define parâmetros de cabeçalho (razão social, CNPJ, endereço, UF)

### Processamento dos Dados
- Executa consulta SQL com agregações
- Aplica filtros dinâmicos conforme parâmetros informados
- Considera apenas dados da empresa ativa
- Agrupa informações por palete, caixa e item

### Pós-processamento
- Exibe o nome da caixa apenas na primeira ocorrência de cada grupo (romaneio + caixa)

## ⚠️ Regras de Negócio

### Validações
- Obrigatório informar romaneio ou pedido de venda
- Considera apenas registros válidos conforme `obterWherePadrao()`

### Cálculos
- Peso Líquido:
  - Unidade `TB`: quantidade × peso líquido do produto
  - Demais unidades: soma da quantidade
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
| PDF | Resumo do Packing List por Palete | gerarPDF() |

## 🔧 Dependências

### Bibliotecas
- sam.server.samdev.relatorio – Infraestrutura de relatórios
- sam.server.samdev.utils – Parâmetros
- sam.model – Entidades do sistema

### Recursos
- Logotipo institucional carregado dinamicamente

## 📝 Observações Técnicas
- Utiliza SQL nativo com agrupamentos e cálculos
- Uso de campos JSON para dados de produto
- Estrutura otimizada para conferência logística
- Relatório exclusivo para saída em PDF
