# SRF_DanfeSimplificada

## 📖 Descrição
Relatório para geração da DANFE (Documento Auxiliar da Nota Fiscal Eletrônica) Simplificada para documentos fiscais de saída no sistema SRF (Sistema de Registro Fiscal). Gera a representação gráfica simplificada da NF-e conforme legislação vigente.

## 🎯 Finalidade
Gerar a DANFE simplificada para documentos fiscais modelo 55 (NF-e) emitidos pela empresa, proporcionando uma representação gráfica legível e organizada das notas fiscais para arquivamento, conferência ou entrega ao cliente.

## 👥 Público-Alvo
- Departamento Fiscal
- Departamento Comercial
- Setor de Expedição
- Clientes (para acompanhamento)
- Auditores fiscais

## 📊 Dados e Fontes
**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento
- `Eaa0102` - Dados gerais do documento
- `Eaa0101` - Endereços do documento
- `Abb01` - Central de documentos
- `Abe01` - Entidades (clientes)
- `Aac10` - Empresa
- `Aah20` - Veículos
- `Abe30` - Condição de pagamento

**Entidades Envolvidas:**
- `Eaa01` - Documento fiscal NF-e
- `Abb01` - Central do documento
- `Abe01` - Cliente/destinatário
- `Aac10` - Empresa emitente
- `Eaa0102` - Dados gerais do documento
- `Eaa0101` - Endereço de entrega
- `Eaa0103` - Itens da nota fiscal

## ⚙️ Parâmetros da Fórmula
1. **numeroInicial** (Integer): Número inicial do documento fiscal (padrão: 000000001)
2. **numeroFinal** (Integer): Número final do documento fiscal (padrão: 999999999)
3. **serie** (String): Série do documento fiscal
4. **entInicial** (String): Código inicial da entidade
5. **entFinal** (String): Código final da entidade
6. **emissao** (Date[]): Período de emissão da NF-e
7. **entraSai** (Date[]): Período de entrada/saída da mercadoria
8. **eaa01id** (Long): ID específico do documento (para impressão individual)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração do relatório DANFE:
1. **Coleta parâmetros** de filtragem
2. **Configura recursos** (logos, imagens)
3. **Busca documentos** conforme critérios
4. **Processa cada documento** e seus componentes
5. **Gera PDF** com sub-relatórios

### `buscarIdsDocumentos()`
Busca os IDs dos documentos NF-e conforme critérios de filtro:
- Filtra por número inicial/final
- Filtra por código da entidade
- Filtra por período de emissão
- Filtra por período de entrada/saída
- Filtra por série
- Apenas documentos emitidos (eaa01emissao = 1)
- Apenas modelo 55 (NF-e)

### `comporDadosEmpresa()`
Coleta e organiza os dados da empresa emitente:
- Razão social e CNPJ
- Endereço completo
- Telefone e email
- Inscrição estadual
- Logotipo da empresa

### `comporDadosCentral()`
Processa os dados da central do documento:
- Número e série da NF-e
- Data de emissão
- Condição de pagamento
- Números de saída relacionados (orcamentos)

### `comporDadosDocumento()`
Coleta informações específicas do documento fiscal:
- Chave de acesso da NF-e
- Protocolo de autorização
- Data e hora de emissão
- Observações do contribuinte e fisco
- Valores totais da nota
- Informações de cancelamento (se houver)

### `comporDadosEndereco()`
Processa o endereço do destinatário:
- Endereço completo (logradouro, número, complemento)
- Bairro, município e UF
- CEP
- Telefone de contato
- Inscrição estadual do destinatário

### `comporDadosGerais()`
Coleta dados gerais da operação:
- Espécie do documento
- Tipo de frete (CIF/FOB)
- Dados do despachante (se houver)
- Dados do veículo transportador (se houver)
- Marca/volumes
- Indicador de consumidor final

### `buscarItensdaNfe()`
Processa todos os itens da nota fiscal:
- Código e descrição do produto
- Quantidade e valor unitário
- CFOP, NCM, CST/CSOSN
- Valores de ICMS e IPI
- Informações de lotes (se aplicável)
- Referência do pedido do cliente

### `comporValores()`
Calcula e organiza os valores totais da nota:
- Valor total da nota
- Valor do frete
- Valor do seguro
- Outras despesas acessórias
- Valores de impostos (ICMS, IPI, PIS, COFINS)
- Percentuais de impostos sobre o total

### `comporEnderecoEntrega()`
Processa endereço de entrega alternativo (se existir):
- Endereço completo de entrega
- Diferenciado do endereço do destinatário
- Para casos de entrega em local diferente do cadastro

### `buscarItens()`
Consulta otimizada dos itens da nota com agrupamentos:
- Agrupa itens por referência do produto
- Soma quantidades e valores
- Coleta dados fiscais dos itens
- Mantém organização por sequência

## 📝 Fluxo de Execução

### 1. **Configuração Inicial**
- Carrega parâmetros de entrada
- Configura caminhos das imagens (logo, carimbo de cancelada, sem valor fiscal)
- Coleta dados da empresa ativa

### 2. **Filtragem de Documentos**
- Busca IDs dos documentos conforme critérios
- Aplica filtros combinados (data, número, entidade, série)
- Ordena por número do documento

### 3. **Processamento por Documento**
Para cada documento NF-e encontrado:
1. **Coleta dados básicos** do documento
2. **Processa informações** da empresa e destinatário
3. **Coleta itens** da nota fiscal
4. **Calcula valores** e impostos
5. **Processa duplicatas** (se houver)
6. **Verifica endereço de entrega** alternativo

### 4. **Estruturação de Dados**
- Organiza dados principais em TableMap
- Cria lista de itens com vínculo ao documento
- Prepara dados para sub-relatórios

### 5. **Geração do PDF**
- Cria DataSource principal com dados dos documentos
- Adiciona sub-DataSource para itens
- Gera PDF utilizando template JasperReports
- Permite download do arquivo gerado

## ⚠️ Regras de Negócio

### Validações Críticas
1. **Empresa**: Deve ter dados completos (endereço, município, telefone)
2. **Documento**: Deve ser NF-e (modelo 55) emitida pela empresa
3. **Endereço**: Destinatário deve ter endereço principal com município definido
4. **Município**: Obrigatório no endereço do destinatário
5. **UF**: Obrigatório no endereço do destinatário

### Regras de Filtragem
- **Número**: Range entre número inicial e final
- **Entidade**: Range entre códigos inicial e final
- **Período**: Filtro por data de emissão e/ou entrada/saída
- **Série**: Filtro específico por série do documento
- **Emissão**: Apenas documentos emitidos pela empresa (não recebidos)

### Tratamento Especial
1. **Canceladas**: Exibe carimbo de "CANCELADA" se documento estiver cancelado
2. **Sem Valor Fiscal**: Exibe carimbo correspondente se aplicável
3. **Consumidor Final**: Destaca quando operação é para consumidor final
4. **Pedido Cliente**: Agrupa números de pedido do cliente nos itens
5. **Endereço Entrega**: Considera endereço alternativo de entrega se existir

### Cálculos Especiais
1. **Percentuais de Impostos**: Calcula percentual de impostos federais e estaduais sobre o total da nota
2. **Agrupamento de Itens**: Agrupa itens iguais por referência do produto
3. **Valores Totais**: Soma valores de todos os itens e impostos

## 🔄 Dependências

**Classes:**
- `RelatorioBase` - Classe base para relatórios do sistema
- Todas as entidades do modelo SAM mencionadas na seção "Entidades Envolvidas"

**Bibliotecas:**
- `br.com.multiorm` - ORM e consultas ao banco
- `br.com.multitec.utils` - Utilitários diversos
- `sam.server.samdev.relatorio` - Framework de relatórios
- `sam.server.samdev.utils.Parametro` - Parâmetros de consulta

**Recursos Externos:**
- Arquivo JRXML: `SRF_DanfeSimplificada.jrxml`
- Sub-relatório: `SRF_DanfeSimplificada_S1.jrxml`
- Imagens: Logo.png, canceladas.png, SemValorFiscal.png

## 🎨 Saída da Fórmula
A fórmula gera um arquivo PDF contendo a DANFE simplificada:

### Arquivo de Saída
- **Formato**: PDF
- **Layout**: DANFE simplificada conforme padrão SEFAZ
- **Conteúdo**: Uma ou múltiplas DANFEs conforme filtros
- **Organização**: Documentos ordenados por número

### Elementos da DANFE
- **Cabeçalho**: Dados do emitente e destinatário
- **Corpo**: Itens da nota fiscal com valores
- **Rodapé**: Valores totais, impostos e informações adicionais
- **Carimbos**: Cancelada, Sem Valor Fiscal (se aplicável)

### Campos Gerados
- **Dados do Emitente**: Razão social, CNPJ, endereço, IE
- **Dados do Destinatário**: Razão social, CPF/CNPJ, endereço, IE
- **Dados da NF-e**: Número, série, data, chave de acesso, protocolo
- **Itens**: Descrição, quantidade, valor unitário, valor total
- **Impostos**: ICMS, IPI, valores e percentuais
- **Transporte**: Dados do frete e veículo (se houver)
- **Pagamento**: Condições e duplicatas (se houver)

## 📌 Observações Técnicas

### Arquitetura
- Relatório baseado em JasperReports com templates JRXML
- Processamento em lote para múltiplos documentos
- Uso de TableMap para estruturação de dados
- Sub-relatórios para itens da nota fiscal

### Performance
- Consultas otimizadas com agrupamento de itens
- Processamento document por documento
- Cache de entidades frequentemente acessadas
- Paginação automática no PDF

### Manutenibilidade
- Métodos organizados por responsabilidade
- Separação clara entre coleta e processamento de dados
- Tratamento adequado de exceções
- Comentários indicando alterações importantes

### Metadados
- Última alteração: 15/12/2025 15:15 por Bruno
- Package: `linhasita.relatorios.srf`

## 🔧 Configurações Necessárias

### Pré-requisitos do Sistema
1. **Empresa**: Cadastro completo com dados fiscais e endereço
2. **Documentos NF-e**: Processados e autorizados no sistema
3. **Itens**: Cadastrados com referências e dados fiscais
4. **Clientes**: Cadastrados com endereços completos
5. **Recursos gráficos**: Logotipo e carimbos no diretório correto

### Configurações Específicas
1. **Template JRXML**: Arquivos de layout configurados
2. **Imagens**: Logotipo da empresa e carimbos especiais
3. **Campo referência**: Configurado no JSON dos itens (item_referencia)
4. **Caminhos**: Diretórios de recursos configurados corretamente

## ⚠️ Considerações de Implementação

### Design Visual
- Layout simplificado mas com todas informações obrigatórias
- Organização clara por seções
- Destaque para informações importantes
- Carimbos visíveis para situações especiais

### Tratamento de Dados
- Agrupamento inteligente de itens similares
- Cálculos precisos de valores e percentuais
- Formatação adequada de datas e valores monetários
- Tratamento de valores nulos ou ausentes

### Performance
- Otimizado para impressão de múltiplas DANFEs
- Consultas eficientes ao banco de dados
- Processamento em memória controlado
- Geração rápida do PDF final

### Manutenção
- Fácil adaptação para mudanças no layout da DANFE
- Configuração flexível de filtros
- Tratamento de erros robusto
- Logs adequados para diagnóstico

## 🎨 Estrutura do Código
- **Package**: `linhasita.relatorios.srf`
- **Imports**: ORM, utilitários, entidades SAM, relatórios
- **Constantes**: Caminhos de imagens, critérios fixos
- **Variáveis**: Estruturas de dados para documentos e itens
- **Métodos**: Organizados por tipo de dado processado

## 🔧 Métodos Auxiliares
- `buscarTipoFrete()`: Converte código numérico para descrição do frete
- `buscarNumSaida()`: Busca números de saída relacionados ao documento
- `buscarLoteItem()`: Processa informações de lote dos itens
- `criarParametroSql()`: Cria parâmetros para consultas SQL

## 📊 Otimizações Implementadas
1. **Agrupamento de Itens**: Evita duplicação de linhas iguais na DANFE
2. **Consultas Diretas SQL**: Para casos complexos de agrupamento
3. **Cache de Entidades**: Empresa e dados fixos em memória
4. **Processamento Incremental**: Documento por documento com liberação de memória

## ⚠️ Limitações Conhecidas
- Processamento pode ser lento para muitos documentos simultâneos
- Dependência da estrutura de campos JSON configurada
- Necessidade de imagens nos caminhos específicos
- Layout fixo baseado no template JRXML