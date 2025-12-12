# SRF - Documento

## 📖 Descrição
Fórmula para processamento e composição de documentos fiscais, aplicando regras de tributação, observações fiscais, cálculo de comissões, validações de itens e tratamento de documentos referenciados.

## 🎯 Finalidade
Processar documentos fiscais (notas fiscais) aplicando regras de negócio específicas, como validação de itens, cálculo de taxas de comissão, composição de observações e tratamento de documentos referenciados.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Tesouraria
- Contabilidade

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe02` - Dados comerciais da entidade
- `Abe03` - Dados fiscais da entidade
- `Abe05` - Representantes
- `Abe40` - Tabelas de preço
- `Abb01` - Documentos centrais
- `Abb10` - Operações comerciais
- `Abd01` - Parâmetros de cálculo de documentos (PCD)
- `Abd02` - Campos fiscais do PCD
- `Abd05` - Campos industriais do PCD
- `Eaa01` - Documentos fiscais
- `Eaa0101` - Endereços do documento
- `Eaa0103` - Itens do documento fiscal
- `Eaa01033` - Itens de devolução referenciados
- `Abm01` - Produtos
- `Abm0101` - Configurações do produto por empresa
- `Abm10` - Valores do produto
- `Abm1001` - Valores do produto por UF
- `Aac10` - Empresas
- `Aac13` - Dados fiscais da empresa
- `Aag02` - Estados (UF)
- `Aag0201` - Municípios
- `Aaj10` - Códigos de situação tributária (CST)

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| procInvoc | String | Sim | Processo de invocação da fórmula |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação do documento fiscal (Eaa01)
- Carregamento da operação comercial (Abb10)
- Obtenção do PCD (Abd01)
- Carregamento da entidade (Abe01) e dados fiscais
- Carregamento da empresa (Aac10) e dados fiscais

### 2. **Processamento por Item**
- Validação de itens sem código de produto
- Carregamento de configurações do produto (Abm0101)
- Obtenção de valores do produto por UF (Abm1001)
- Aplicação de observações fiscais específicas por item
- Tratamento de CST ICMS com redução de base de cálculo

### 3. **Cálculo de Taxas de Comissão**
- Obtenção de taxas fixadas na entidade (Abe02)
- Obtenção de taxas fixadas na tabela de preço (Abe40)
- Obtenção de taxas de representantes (Abe05)
- Definição final das taxas no documento

### 4. **Composição de Observações**
- Composição de observações de uso interno
- Composição de observações fiscais
- Composição de observações ao contribuinte
- Composição de observações de retenção/indenização
- Composição de observações gerais

### 5. **Processamento Final**
- Cálculo de fidelidade (para PCD específico)
- Cálculo de crédito/cashback (para PCD específico)
- Composição de observações com documentos referenciados
- Atualização do documento fiscal

## ⚠️ Regras de Negócio

### Validação de Itens
- Todos os itens devem ter código de produto preenchido
- Configurações do produto são carregadas por empresa
- Valores do produto são aplicados conforme UF do endereço principal

### Observações Fiscais
- Observações do fisco podem ser definidas por produto/UF
- Tratamento especial para CST ICMS 20 (redução de base de cálculo)
- Regime especial Simples Nacional com aproveitamento de ICMS

### Taxas de Comissão
- Hierarquia de fontes: Entidade → Tabela de Preço → Representante
- Taxas podem ser fixadas em múltiplos níveis
- Valores zerados são substituídos pelas fontes disponíveis

### Documentos Referenciados
- Identificação de itens devolvidos referenciados
- Composição automática de observação com dados dos documentos
- Inclusão de número, data e chave de acesso da NFe

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processamento do documento.

### `setarObterTaxasComissaoDocumento()`
Calcula e define as taxas de comissão do documento seguindo a hierarquia estabelecida.

### `obterTaxaDoRepresentante(Abe01 abe01rep)`
Obtém a taxa de comissão de um representante específico.

### `comporObservacoesDocumento()`
Composição das observações do documento a partir das configurações da entidade e do PCD.

### `comporObsContribuinteComChaveNFeDocumentosReferenciados()`
Adiciona à observação do contribuinte informações sobre documentos referenciados.

## 📊 Estrutura de Processamento

**Entradas:**
- Documento fiscal (Eaa01) com todos os relacionamentos
- Processo de invocação para contexto de execução

**Processamentos:**
- Validações de consistência
- Cálculos financeiros e fiscais
- Composição de textos e observações

**Saídas:**
- Documento fiscal atualizado com:
  - Observações preenchidas
  - Taxas de comissão definidas
  - Campos JSON calculados
  - Validações aplicadas

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários e exceções
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas e formatação
- `java.util` - Coleções e utilitários

**Módulo:** SRF (Sistema de Faturamento)

## 📝 Observações Técnicas

### Tratamento de JSON
- Campos livres (JSON) são utilizados para dados flexíveis
- Estruturas JSON são manipuladas via TableMap
- Dados específicos podem ser armazenados e recuperados dinamicamente

### Consultas ao Banco
- Uso de Criteria para consultas dinâmicas
- Consultas SQL com parâmetros nomeados
- Aplicação automática de filtros de segurança (getSamWhere)

### Hierarquia de Configurações
- Configurações são herdadas em níveis: Empresa → Entidade → Documento
- Valores mais específicos sobrepõem os mais genéricos
- Fallback automático para valores padrão

### Validações de Negócio
- Interrupção por exceção em caso de erros críticos
- Validações preventivas em itens e configurações
- Mensagens de erro claras para o usuário final

---

**Última Alteração:** 02/12/2025 às 10:00  
**Autor:** Bruno  
**Tipo:** Fórmula de Documento Fiscal  
**Versão:** 1.0