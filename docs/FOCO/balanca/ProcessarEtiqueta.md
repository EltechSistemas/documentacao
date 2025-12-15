# ProcessarEtiqueta.md
# Foco - Processar Etiqueta

## 📖 Descrição
Serviço responsável por processar e gerar etiquetas para pesagens (balança), associando lotes, séries e dados de vencimento, e retornando os dados para impressão.

## 🎯 Finalidade
- Gerar etiquetas a partir de dados de pesagem
- Associar etiquetas a lotes e séries
- Disparar impressão de etiquetas
- Atualizar status da pesagem no sistema

## 👥 Público-Alvo
- Operadores de balança
- Faturamento
- Controle de qualidade

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Bab01` - Pesagens
- `Bab0105` - Itens de pesagem/etiquetas
- `Abp20` - Produtos
- `Abm70` - Etiquetas geradas
- `Aam05` - Modelos de etiqueta

## ⚙️ Parâmetros do Serviço

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| bab01id | Long | Sim | ID da pesagem |
| abp20id | Long | Sim | ID do produto |
| abm01id | Long | Sim | ID do município |
| abb01id | Long | Sim | ID do centro de custo |
| diasParaVencimento | Long | Não | Dias a adicionar para vencimento |
| lote | String | Não | Número do lote |
| serie | String | Não | Número da série |
| peso | BigDecimal | Sim | Peso da pesagem |

## 🔄 Fluxo do Processo

### 1. **Recebimento e Validação dos Dados**
- Leitura do corpo da requisição HTTP
- Extração dos parâmetros necessários
- Validação dos dados obrigatórios

### 2. **Busca de Dados Complementares**
- Busca do modelo de etiqueta baseado no produto
- Busca de informações de lote/série do centro de custo

### 3. **Geração da Etiqueta**
- Criação do DTO para gravação da etiqueta
- Cálculo da data de vencimento (data atual + diasParaVencimento)
- Chamada ao serviço de gravação de etiqueta
- Obtenção do ID da etiqueta gerada (abm70id)

### 4. **Geração dos Dados para Impressão**
- Criação do DTO para impressão de etiquetas
- Chamada ao serviço de impressão para obter bytes da etiqueta

### 5. **Atualização da Pesagem**
- Criação do registro Bab0105 com dados da etiqueta
- Associação com a pesagem original (bab01id)
- Definição do status "A Concluir"

### 6. **Retorno da Resposta**
- Estruturação dos dados em ResponseDto
- Retorno com status 200 e conteúdo JSON

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração de etiquetas.

**Fluxo:**
1. Extrai parâmetros do corpo da requisição
2. Busca modelo de etiqueta (`buscarModelo`)
3. Busca informações de lote/série (`buscarLoteSerie`)
4. Prepara DTO para gravação (`CGSGravarEtiquetaDto`)
5. Grava etiqueta via `CGSService.gravarEtiqueta()`
6. Gera dados para impressão via `CGS7050Service.buscarDadosImpressaoEtiquetas()`
7. Cria registro `Bab0105` com status "A_CONCLUIR"
8. Retorna resposta com dados da etiqueta e bytes para impressão

### `buscarModelo(Long abp20id)`
Busca o ID do modelo de etiqueta associado ao produto.

**Parâmetros:**
- `abp20id`: ID do produto

**Retorno:** `Long` - ID do modelo de etiqueta (aam05id)

### `buscarLoteSerie(Long abb01id)`
Busca informações de lote e série do centro de custo.

**Parâmetros:**
- `abb01id`: ID do centro de custo

**Retorno:** `TableMap` com campos:
- `bab01lote`: Número do lote
- `bab01serie`: Número da série

## 📊 Estrutura de Dados

### **CGSGravarEtiquetaDto**
DTO para gravação de etiqueta:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| abb01id | Long | ID do centro de custo |
| abm01id | Long | ID do município |
| aam05id | Long | ID do modelo de etiqueta |
| vencimento | LocalDate | Data de vencimento |
| dataEmissao | LocalDate | Data de emissão |
| dataBase | LocalDate | Data base |
| peso | BigDecimal | Peso da pesagem |
| lote | String | Número do lote |
| serie | String | Número da série |
| observacao | String | Observações (opcional) |
| camposLivres | Object | Campos livres (opcional) |

### **CGS7050ImprimirDto**
DTO para impressão de etiquetas:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| aam05id | Long | ID do modelo de etiqueta |
| abm70ids | List<Long> | IDs das etiquetas a imprimir |

### **ResponseDto**
DTO de resposta do serviço:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| bab0105 | Bab0105 | Registro da etiqueta criada |
| bytes | byte[] | Bytes da etiqueta para impressão |

## ⚠️ Regras de Negócio

### **Transações**
- Inicia transação se não existir uma aberta
- Commita transação ao final do processo

### **Datas**
- Data de emissão: data atual
- Data base: data atual
- Vencimento: data atual + diasParaVencimento
- Se `diasParaVencimento` não informado, usa data atual

### **Status da Etiqueta**
- `Bab0105.STATUS_A_CONCLUIR`: Etiqueta gerada, aguardando conclusão

### **Validações**
- Parâmetros obrigatórios devem ser fornecidos
- Peso deve ser maior que zero (validação implícita)
- Produto deve existir e ter modelo de etiqueta associado

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - TableMap e utilitários
- `multitec.utils.jackson` - Serialização JSON
- `sam.dto.cgs` - DTOs do módulo CGS
- `sam.model` - Entidades do sistema
- `sam.server.cgs.service` - Serviços do CGS
- `sam.server.samdev.relatorio` - Base para servlets
- `java.time` - Manipulação de datas
- `org.springframework` - HTTP e media types

**Serviços:**
- `CGSService` - Serviço principal do CGS
- `CGS7050Service` - Serviço de impressão de etiquetas
