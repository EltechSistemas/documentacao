# SRF - Gerar XML de NFe (Fiscal) - Mitra

## 📖 Descrição
Fórmula customizada para geração do arquivo XML da Nota Fiscal Eletrônica (NFe) conforme layout 4.00, com implementações específicas para o cliente Mitra, incluindo tratamento especial para PCD (Pessoas com Deficiência) e ajustes nos valores de frete.

## 🎯 Finalidade
Gerar automaticamente o XML da NFe a partir dos dados do documento fiscal, aplicando validações, formatações, regras fiscais e tratamentos específicos para operações com PCD conforme a legislação vigente.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Contabilidade

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0101` - Endereços do documento
- `Eaa0102` - Dados gerais do documento
- `Eaa0103` - Itens do documento
- `Eaa01033` - Itens vinculados (devolução)
- `Eaa01034` - Declaração de importação
- `Eaa010341` - Adições da DI
- `Eaa01038` - Rastreabilidade
- `Eaa0104` - Declaração de exportação
- `Eaa0113` - Financeiro do documento
- `Eaa01131` - Formas de pagamento
- `Aac10` - Empresa emitente
- `Aac1002` - Inscrições estaduais da empresa
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe0101` - Endereços das entidades
- `Abb01` - Central de documentos
- `Abm01` - Itens cadastrais
- `Abm0101` - Configuração de itens por empresa
- `Abg01` - NCM
- `Abg0101` - NVE
- `Aaj03` - Situação do documento
- `Aaj04` - Código ANP
- `Aaj10` - CST ICMS
- `Aaj11` - CST IPI
- `Aaj12` - CST PIS
- `Aaj13` - CST COFINS
- `Aaj14` - CSOSN
- `Aaj15` - CFOP
- `Aam06` - Unidades de medida
- `Aah20` - Veículos/transportes
- `Aag01` - Países
- `Aag02` - Estados/UF
- `Abd01` - PCD (Pessoas com Deficiência)

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| formaEmis | Integer | Não | Forma de emissão (1=Normal, 2=Contingência) |
| contDt | LocalDate | Não | Data da contingência |
| contHr | LocalTime | Não | Hora da contingência |
| contJust | String | Não | Justificativa da contingência |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento da empresa emitente
- Carregamento do PCD associado ao documento
- Seleção do alinhamento fiscal (0010)
- Composição dos dados do documento (central, dados gerais, endereços)

### 2. **Validações de Dados**
- Validação de dados obrigatórios do emitente
- Validação de dados do documento fiscal
- Validação de itens, impostos e formas de pagamento
- Validação de documentos referenciados e transportes

### 3. **Geração da Estrutura XML**
- Criação do elemento raiz `NFe`
- Definição das informações básicas (`infNFe`)
- Montagem da identificação (`ide`)
- Inclusão de documentos referenciados (`NFref`)

### 4. **Montagem das Seções da NFe**
- Emitente (`emit`)
- Destinatário (`dest`)
- Itens (`det`) com produtos, impostos e informações específicas
- Totais (`total`) com valores de ICMS, ISS, retenções
- Transporte (`transp`) com veículos e volumes
- Cobrança (`cobr`) com faturas e duplicatas
- Pagamento (`pag`) com formas de pagamento
- Informações adicionais (`infAdic`)
- Exportação (`exporta`)
- Responsável técnico (`infRespTec`)

### 5. **Finalização**
- Geração do XML final
- Retorno da chave de acesso e dados do XML

## ⚠️ Regras de Negócio

### Implementações Específicas Mitra

#### **Tratamento de PCD**
- Carregamento do PCD associado ao documento (`Abd01`)
- Verificação do código do PCD para aplicar regras específicas
- Quando PCD código "314": valor do frete definido como 0 na tag `vFrete`

#### **Alterações na Geração**
- Alinhamento fixo: "0010" (não há seleção por classificação tributária)
- Tipo de impressão (`tpImp`): 1 (Documento Nacional)
- Formatação de valores unitários com 9 decimais (`vUnCom`, `vUnTrib`)
- Busca de declarações de exportação com fetch da UF de embarque

#### **Regras de Pagamento**
- Cálculo do total parcelado sem formas de pagamento específicas
- Quando houver parcelamento sem forma de pagamento: gera tag com `tPag = 15` (Boleto Bancário)
- Ordenação do financeiro por data de vencimento nominal

### Validações Obrigatórias
- Empresa emitente deve ter município, IE, endereço e CNPJ/CPF válidos
- Documento deve ter tipo, modelo, situação e dados gerais
- Itens devem ter unidade de medida, descrição, CFOP e CST/CSOSN conforme classificação tributária
- Para Simples Nacional (classTrib = "001"): CSOSN obrigatório
- Para outras classificações: CST ICMS obrigatório (exceto para itens com ISS)

### Regras Fiscais
- Identificação do destino (1=Interna, 2=Interestadual, 3=Exterior)
- Tratamento diferenciado para operações com e sem ISS
- Cálculo de impostos (ICMS, IPI, PIS, COFINS, ISS) conforme CST/CSOSN
- Informações de ST, FCP e retenções conforme configuração
- Tratamento especial para combustíveis (ANP, CIDE)

### Formatação e Estrutura
- Formatação de valores monetários com 2 decimais
- Formatação de quantidades com 4 decimais
- Formatação de valores unitários com 9 decimais
- Ajuste de caracteres especiais no XML
- Geração de chave de acesso com dígito verificador
- Versão do layout: 4.00

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração do XML.

### `validarDadosDaNFe()`
Realiza validações abrangentes dos dados necessários para a geração da NFe.

### `comporFilhosDocumento()`
Carrega e associa os dados relacionados ao documento (central, dados gerais, endereços).

### `emitente()`
Monta a seção do emitente com dados da empresa.

### `destinatario()`
Monta a seção do destinatário com dados da entidade.

### `item()`
Processa todos os itens do documento, incluindo produtos, impostos e informações específicas.

### Métodos de Busca
- `buscarInscricaoEstadualPorEstado()`: Busca IE da empresa por UF
- `buscarDocumentosReferenciados()`: Busca notas referenciadas
- `buscarItensDoDocumento()`: Busca itens do documento
- `buscarNVEsPorNCM()`: Busca NVE relacionados ao NCM
- `buscarDeclaracaoDeImportacaoPorItem()`: Busca DI dos itens
- `buscarAdicoesPorDI()`: Busca adições da DI
- `buscarDeclaracoesDeExportacao()`: Busca declarações de exportação (com fetch da UF)
- `buscarRastreabilidadeDoItem()`: Busca dados de rastreabilidade
- `buscarLancamentosDoItem()`: Busca lançamentos de estoque
- `buscarFinanceiroPorDocumento()`: Busca parcelas financeiras ordenadas por vencimento
- `buscarFormasDePagamentoPorDocumento()`: Busca formas de pagamento
- `buscarTotalParcelado()`: Busca total das parcelas sem forma de pagamento específica

## 📊 Estrutura de Saída

**XML da NFe:** String formatada conforme layout 4.00

**Parâmetros de Retorno:**
- `chaveNfe`: Chave de acesso da NFe (44 dígitos)
- `dados`: XML completo da NFe

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários (DateUtils, StringUtils, ValidacaoException)
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `sam.server.samdev.utils` - Utilitários do SAM (NFeUtils, Parametro)
- `java.time` - Manipulação de datas
- `java.math` - Cálculos com BigDecimal

**Módulo:** Fiscal

## 📝 Observações Técnicas

### Versão do Layout
- Layout 4.00 da NFe
- Suporte a NFC-e (modelo 65) e NFe normal (modelo 55)

### Ambiente
- Produção: `tpAmb = 1`
- Homologação: `tpAmb = 2`
- CNPJ de homologação: 99999999000191

### Contingência
- Forma de emissão normal: `tpEmis = 1`
- Contingência: `tpEmis = 2` com data/hora e justificativa

### Tratamento de Dados
- Formatação de IE (remoção de caracteres não numéricos)
- Ajuste de fone (DDD + número)
- Substituição de caracteres especiais no XML (&, <, >, ", ')
- Formatação de datas no padrão UTC

### Campos Livres (JSON)
Os dados fiscais detalhados são armazenados em campos livres (JSON) nas tabelas:
- `eaa01json` para totais do documento
- `eaa0103json` para valores por item

### Implementações Específicas
- **PCD código 314**: Zera o valor do frete na NFe
- **Tipo de impressão**: Fixo em 1 (Documento Nacional)
- **Alinhamento**: Fixo em "0010"
- **Busca de exportação**: Com fetch da UF para otimização

---

**Última Alteração:** 09/12/2025 às 16:26  
**Autor:** NAGYLA  
**Tipo:** Fórmula de Geração de Arquivos de NFe  
**Versão:** 2.0 (Customizada para Mitra)