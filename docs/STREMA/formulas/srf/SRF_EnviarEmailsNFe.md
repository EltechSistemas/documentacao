# SRF_EnviarEmailsNFe.md

## 📖 Descrição
Sistema de envio automatizado de e-mails para Notas Fiscais Eletrônicas (NF-e), incluindo documentos fiscais, boletos e cartas de correção para múltiplos destinatários.

## 🎯 Finalidade
Automatizar o processo de distribuição de documentos fiscais eletrônicos via e-mail para clientes, representantes, transportadoras e setores internos, garantindo conformidade com a legislação fiscal.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Cobrança
- Representantes Comerciais
- Transportadoras

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_EnviarEmailsNFe` - Sistema de envio de e-mails NF-e

**Localização:** `strema/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `AAA16` - Processamento de mensageria
- `ABB01` - Centrais de documento
- `ABE01` - Entidades/clientes
- `EAA0102` - Dados gerais do documento
- `EAA0114` - Cartas de correção

**Entidades Envolvidas:**
- `EmailNFeDto` - DTO para transporte de dados de e-mail
- `Eaa01` - Documento fiscal
- `Abe01` - Entidade/cliente

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01id | Long | Sim | ID do documento fiscal |
| aaa16id | Long | Sim | ID do processamento da mensageria |
| eaa0114id | Long | Não | ID da carta de correção (quando aplicável) |

## 🔄 Fluxo do Processo

1. **Inicialização e Validações**
   - Recebe parâmetros de entrada (documento, processamento, carta de correção)
   - Carrega dados do documento fiscal e entidade
   - Verifica tipo de processamento (faturamento, cancelamento, carta de correção)

2. **Composição da Mensagem**
   - Monta assunto padrão com dados da empresa e número da NF
   - Compõe corpo do e-mail conforme tipo de operação
   - Inclui dados do cliente, chave de acesso e protocolo

3. **Distribuição de Destinatários**
   - **Faturamento:** E-mail principal do cliente
   - **Cobrança:** E-mail específico de cobrança quando diferente
   - **Representantes:** E-mails dos representantes comerciais
   - **Transportadora:** E-mail da transportadora quando houver despacho
   - **Contatos da Entidade:** E-mails adicionais por classificação (9002, 9003, 9004)
   - **Cópia Interna:** E-mail nfe@stremasbaterias.com.br

4. **Configuração de Anexos**
   - XML da NF-e
   - DANFE (PDF)
   - Boleto bancário
   - Configuração por tipo de destinatário

## ⚠️ Regras de Negócio

### Tipos de Envio
- **Faturamento:** XML + DANFE (obrigatório) + Boleto (condicional)
- **Cobrança:** Boleto (quando e-mail diferente do faturamento)
- **Cancelamento/CC-e:** XML apenas
- **Representantes:** XML + DANFE + Boleto
- **Transportadora:** XML + DANFE

### Classificação de Contatos da Entidade
- **9002:** Envia somente boleto
- **9003:** Envia somente XML e PDF
- **9004:** Envia boleto, XML e PDF

### Remetentes
- **0:** Principal
- **1:** Cobrança  
- **2:** Faturamento

### Validações
- E-mail de faturamento é obrigatório
- Cancelamentos e cartas de correção enviam apenas XML
- Representantes recebem todos os anexos
- Cópia sempre enviada para nfe@stremasbaterias.com.br

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| emails | Lista de e-mails configurados para envio | List<EmailNFeDto> |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `sam.dto.srf` - DTOs do módulo SRF

**Estruturas de Dados:**
- `EmailNFeDto` - Transporte de configurações de e-mail
- `TableMap` - Armazenamento de dados de contatos

## 📝 Observações Técnicas

- **Processamento:** Síncrono, executado durante processamento fiscal
- **Formatação:** E-mails em HTML com dados formatados
- **Flexibilidade:** Múltiplos destinatários com configurações específicas
- **Rastreabilidade:** Inclui chave de acesso e protocolo da SEFAZ

### Estrutura do EmailNFeDto
- `assunto`: Assunto do e-mail
- `corpo`: Corpo da mensagem em HTML
- `emailsDestinoPara`: Lista de destinatários
- `enviarXML`: Flag para envio do XML
- `enviarDanfe`: Flag para envio do DANFE
- `enviarBoleto`: Flag para envio do boleto
- `emailRemetente`: Tipo de remetente (0, 1, 2)

### Busca de Destinatários
- **Cobrança:** Prioridade para endereço no documento > endereço principal da entidade
- **Representantes:** Busca em até 5 representantes vinculados ao documento
- **Transportadora:** Obtido do campo de despacho do documento
- **Contatos:** Consulta direta via SQL por classificação específica