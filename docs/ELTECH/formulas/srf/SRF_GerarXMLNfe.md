# SRF_GerarXMLNfe - Geração de XML para Nota Fiscal Eletrônica

## 📖 Descrição
Classe responsável pela geração do arquivo XML da Nota Fiscal Eletrônica (NFe) seguindo o layout oficial da SEFAZ, com suporte para diferentes modelos (55, 65), regimes tributários e operações específicas.

## 🎯 Finalidade
Gerar XML válido para NFe contendo todas as informações fiscais, tributárias e comerciais necessárias para transmissão à SEFAZ, incluindo dados do emitente, destinatário, produtos, impostos e informações adicionais.

## 👥 Público-Alvo
- Departamento Fiscal
- Desenvolvedores do sistema
- Integradores de sistemas fiscais
- Administradores do SAM

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| formaEmis | Integer | Sim | Forma de emissão (1=Normal, 2=Contingência) |
| contDt | LocalDate | Não | Data de contingência |
| contHr | LocalTime | Não | Hora de contingência |
| contJust | String | Não | Justificativa da contingência |
| empresa | Aac10 | Sim | Empresa emitente |

## 📋 Estrutura de Dados Principais

### Entidades Envolvidas:
- **Eaa01** - Documento fiscal
- **Abb01** - Cabeçalho do documento
- **Eaa0102** - Dados gerais do documento
- **Eaa0103** - Itens do documento
- **Eaa0101** - Endereços do documento
- **Aac10** - Empresa emitente
- **Abe01** - Entidades (clientes/fornecedores)

### Campos XML Gerados:
- Identificação (ide)
- Emitente (emit)
- Destinatário (dest)
- Produtos/Serviços (det)
- Impostos (ICMS, IPI, PIS, COFINS, ISSQN)
- Transporte (transp)
- Cobrança (cobr)
- Informações adicionais (infAdic)

## 🔄 Fluxo do Processo

### 1. **Inicialização e Validação**
- Carrega dados do documento e empresa
- Valida dados obrigatórios
- Seleciona alinhamento conforme regime tributário

### 2. **Geração da Chave de Acesso**
- Calcula código numérico (cNF)
- Gera chave de acesso única
- Define ambiente (produção/homologação)

### 3. **Construção da Estrutura XML**
- Cria elemento raiz NFe
- Monta estrutura hierárquica do XML
- Aplica formatação e caracteres especiais

### 4. **Preenchimento das Seções**
- **Identificação**: dados básicos da NF
- **Emitente**: dados da empresa
- **Destinatário**: dados do cliente
- **Produtos**: itens com impostos
- **Totais**: valores consolidados
- **Transporte**: dados de frete
- **Pagamento**: formas de pagamento

### 5. **Validação Final e Saída**
- Valida consistência dos dados
- Gera XML final
- Retorna chave e dados para assinatura

## ⚠️ Regras de Negócio

### Validações Obrigatórias:
- Dados completos da empresa emitente
- Informações fiscais do destinatário
- CFOP válido para cada item
- CST/CSOSN conforme regime tributário
- Dados de transporte quando aplicável

### Regimes Tributários:
- **Simples Nacional**: Usa CSOSN e alinhamento 0009
- **Outros Regimes**: Usa CST e alinhamento 0010

### Tipos de Operação:
- Operações internas (CFOP 1xxx, 5xxx)
- Operações interestaduais (CFOP 2xxx, 6xxx)
- Operações com exterior (CFOP 3xxx)

### Cálculos Automáticos:
- Valores de impostos (ICMS, IPI, PIS, COFINS)
- Base de cálculo e aliquotas
- Totais da nota fiscal

## 🎨 Saídas Geradas

| Formato | Descrição | Estrutura |
|---------|-----------|-----------|
| XML | Arquivo XML da NFe | Layout 4.00 da SEFAZ |
| Chave | Chave de acesso | 44 dígitos |
| Dados | XML pronto | Para assinatura digital |

## 🔧 Dependências

### Bibliotecas:
- `multitec.utils` - Utilitários e datas
- `java.time` - Manipulação de datas
- `br.com.multiorm` - Acesso a dados

### Configurações:
- Templates de alinhamento (0009, 0010)
- Parâmetros do sistema (NFeDataProducao)
- Certificado digital (para assinatura)

## 📝 Observações Técnicas

### Estrutura XML:
- Segue padrão oficial da SEFAZ
- Namespace: http://www.portalfiscal.inf.br/nfe
- Versão do layout: 4.00
- Codificação UTF-8

### Tratamento de Dados:
- Substituição de caracteres especiais
- Formatação de valores decimais
- Validação de tamanhos de campos
- Ajuste de timezone para datas

### Performance:
- Uso de TableMap para dados JSON
- Consultas otimizadas ao banco
- Processamento em lote para itens

### Validações:
- MultiValidationException para erros
- Verificação de consistência fiscal
- Validação de documentos referenciados

### Suporte a Cenários Especiais:
- Contingência offline
- Exportação
- Importação com DI
- Combustíveis (ANP)
- Rastreabilidade
- Devolução