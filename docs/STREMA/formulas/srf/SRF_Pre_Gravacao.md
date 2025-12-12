# SRF - Pré-Gravação (Strema)

## 📖 Descrição
Fórmula executada antes da gravação de um documento, responsável por criar/atualizar registros de estoque por item com base em regras específicas de recebimento e tipo de documento.

## 🎯 Finalidade
Processar itens de documentos de recebimento (tipos 202/203) para criar ou atualizar lotes e séries no estoque, garantindo rastreabilidade e controle de inventário.

## 👥 Público-Alvo
- Departamento de Estoque
- Recebimento
- Logística

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal
- `Eaa01038` - Estoque por item
- `Abb01` - Central de documento
- `Abd01` - PCD (Processo/Condição de Documento)
- `Aah01` - Tipo de documento
- `Abb11` - Departamento (Strema)
- `Aam04` - Status (Disponível)
- `Abm15` - Depósito

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal sendo processado |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção do documento fiscal (Eaa01)
- Carregamento da central de documento (Abb01)
- Identificação do tipo de documento (Aah01)
- Carregamento do PCD (Abd01)
- Leitura dos campos livres (JSON) do documento

### 2. **Validação de Contexto**
- Verifica se é um movimento de recebimento (`eaa01esMov == 0`)
- Confirma se o PCD é "314" (recebimento específico)

### 3. **Processamento por Item**
- Para cada item do documento (Eaa0103):
  - **Caso 1:** Item já possui estoque registrado (Eaa01038)
    - Atualiza lote e série nos registros existentes
  - **Caso 2:** Item não possui estoque
    - Cria novo registro Eaa01038 com:
      - Departamento fixo (Strema - ID 67660)
      - Status fixo (Disponível - ID 4224)
      - Depósito fixo (ID 67479)
      - Lote = número do documento
      - Série = invoice do JSON (se existir)
      - Quantidade = quantidade de uso do item

### 4. **Controle de Gravação**
- Define flag `gravar = 1` para permitir gravação
- Retorna flag via parâmetro de saída

## ⚠️ Regras de Negócio

### Contexto de Execução
- Executa apenas para **recebimentos** (`eaa01esMov == 0`)
- Aplica apenas para **PCD 314** (Processo específico de recebimento)
- Atua sobre **tipos de documento 202 ou 203**

### Gestão de Estoque
- **Itens com estoque existente:** Apenas atualiza lote e série
- **Itens sem estoque:** Cria registro completo com:
  - Departamento: Strema (fixo)
  - Status: Disponível (fixo)
  - Depósito: Configurado (fixo)
- **Campos dinâmicos:**
  - Lote: Número do documento (abb01num)
  - Série: Invoice do JSON (se disponível)

### Validações
- Verifica existência de registros Eaa01038 antes de criar
- Utiliza dados do JSON apenas se existirem
- Mantém estrutura hierárquica (Eaa01 → Eaa0103 → Eaa01038)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de pré-gravação.

### `criarEstoqueItem()`
Processa todos os itens do documento para criar/atualizar estoque:
- Identifica itens com estoque existente
- Cria novos registros quando necessário
- Aplica regras específicas de recebimento

### `obterTipoFormula()`
Retorna o tipo da fórmula: `SCV_SRF_PRE_GRAVACAO`

## 📊 Estrutura de Saída

**Parâmetros de Saída:**
- `gravar` - Integer (0-Não gravar, 1-Gravar)

**Alterações no Modelo:**
- Atualiza `eaa01038lote` e `eaa01038serie` nos itens existentes
- Adiciona novos registros `Eaa01038` aos itens sem estoque

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Acesso ao banco de dados
- `sam.model` - Entidades do sistema
- `sam.server.samdev.formula` - Base para fórmulas
- `sam.dicdados` - Tipos de fórmula

**Módulo:** Strema

## 📝 Observações Técnicas

### Tratamento de JSON
- Campos livres do Eaa01 armazenados em `TableMap`
- Campo "invoice" utilizado para série do estoque
- Compatível com estrutura JSON existente

### IDs Fixos
- Departamento Strema: 67660
- Status Disponível: 4224
- Depósito: 67479
*(Nota: Considerar parametrização futura)*

### Performance
- Processamento por lote (todos os itens do documento)
- Acesso otimizado via `getSession().get()`
- Evita consultas desnecessárias

### Validações de Negócio
- Execução condicional baseada em PCD e tipo de movimento
- Preservação de dados existentes
- Criação apenas quando necessário

---

**Última Alteração:** 02/12/2025 às 09:35  
**Autor:** Bruno  
**Tipo:** Fórmula de Pré-Gravação  
**Versão:** 1.0  
**Módulo:** Strema