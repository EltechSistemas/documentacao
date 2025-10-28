# SRF - Enviar Emails NFe

## 📖 Descrição
Fórmula para envio automatizado de e-mails relacionados a Notas Fiscais Eletrônicas, incluindo faturamento, cobrança, cancelamentos e cartas de correção.

## 🎯 Finalidade
Automatizar o envio de e-mails para clientes, representantes e transportadoras com documentos fiscais eletrônicos (XML, DANFE, boletos) de acordo com o tipo de operação.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Cobrança
- Representantes Comerciais

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais NF-e
- `Abb01` - Cabeçalho de documentos
- `Abe01` - Entidades (clientes)
- `Eaa0102` - Dados complementares do documento
- `Aaa16` - Processamento da mensageria
- `Eaa0114` - Cartas de correção
- `Eaa0101` - E-mails específicos por documento
- `Abe0101` - Endereços de e-mail das entidades

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01id | Long | Sim | ID do documento fiscal (Eaa01) |
| aaa16id | Long | Sim | ID do processamento da mensageria |
| eaa0114id | Long | Não | ID da carta de correção (se aplicável) |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção dos parâmetros de entrada
- Carregamento das entidades principais (Eaa01, Aaa16, Abb01, Abe01)
- Verificação da existência de e-mail principal no documento

### 2. **Composição do Corpo da Mensagem**
- **Cancelamento**: Inclui motivo do cancelamento
- **Carta de Correção**: Inclui texto da correção
- **Faturamento Normal**: Mensagem padrão de envio de NF-e
- Formatação HTML com dados do cliente e chave de acesso

### 3. **Definição dos Destinatários**
- **E-mail Principal**: Cliente (faturamento)
- **E-mail Cobrança**: Endereço específico para cobrança
- **Representantes**: Até 5 representantes vinculados ao documento
- **Transportadora**: Empresa de despacho (se informada)

### 4. **Configuração dos Anexos**
- **Faturamento**: XML + DANFE
- **Cobrança**: Boleto (separado ou junto com faturamento)
- **Cancelamento/CC-e**: Apenas XML
- **Representantes**: Apenas DANFE
- **Transportadora**: Apenas XML

## ⚠️ Regras de Negócio

### Tipos de E-mail
- **Faturamento**: Remetente 2 (Faturamento), com XML e DANFE
- **Cobrança**: Remetente 1 (Cobrança), apenas boleto
- **Cancelamento/CC-e**: Remetente 2, apenas XML
- **Representantes**: Remetente 2, apenas DANFE
- **Transportadora**: Remetente 2, apenas XML

### Hierarquia de E-mails de Cobrança
1. E-mail específico no documento (Eaa0101) marcado como cobrança
2. E-mail principal da entidade (Abe0101) marcado como cobrança
3. E-mail principal do documento (Eaa0102)

### Representantes
- Considera até 5 representantes vinculados (rep0 a rep4)
- Apenas e-mails principais das entidades representantes
- Não envia XML, apenas DANFE para acompanhamento

### Transportadora
- Apenas se houver empresa de despacho informada
- Envia apenas XML para fins de logística

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de envio de e-mails.

### `comporCorpoMsg(Aaa16 aaa16, Eaa01 eaa01, Abe01 abe01, Abb01 abb01, Long eaa0114id)`
Compoi o corpo do e-mail conforme o tipo de operação:
- **Cancelamento**: Inclui motivo
- **Carta de Correção**: Inclui texto da correção
- **Faturamento**: Mensagem padrão

### `obterEmailCobranca(Long eaa01id, Long abe01id, Eaa0102 eaa0102)`
Busca o e-mail de cobrança seguindo a hierarquia definida.

### `obterEmailsRepresentantes(Eaa01 eaa01)`
Obtém e-mails dos representantes vinculados ao documento.

### `obterEmailTransportadoraDespacho(Eaa0102 eaa0102)`
Busca e-mail da transportadora responsável pelo despacho.

## 📊 Estrutura de Saída

**EmailNFeDto:**
- `assunto` - Assunto do e-mail
- `corpo` - Corpo em HTML da mensagem
- `emailsDestinoPara` - Lista de destinatários
- `enviarXML` - Indica se envia arquivo XML
- `enviarDanfe` - Indica se envia DANFE
- `enviarBoleto` - Indica se envia boleto
- `emailRemetente` - Tipo de remetente (0: Principal, 1: Cobrança, 2: Faturamento)

**Lista de E-mails Gerados:**
- Retornada no parâmetro `emails` para processamento pela mensageria

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e joins para consultas
- `sam.dicdados` - Tipos de fórmula
- `sam.dto.srf` - DTOs específicos do módulo SRF
- `sam.model` - Entidades do sistema

**Entidades:**
- `EmailNFeDto` - Estrutura de dados para e-mails
- `FormulaBase` - Classe base para fórmulas

## 📝 Observações Técnicas

### Formatação HTML
- Corpo dos e-mails em HTML com tags básicas
- Links para portal da NF-e
- Dados do cliente e chave de acesso formatados
- Assinatura automática do sistema

### Controle de Remetentes
- **0 - Principal**: E-mail principal da empresa
- **1 - Cobrança**: E-mail específico para cobrança
- **2 - Faturamento**: E-mail específico para faturamento

### Validações
- Verifica existência de e-mail principal antes do processamento
- Tratamento de valores nulos em representantes
- Controle de duplicidade de destinatários

### Performance
- Consultas otimizadas com campos específicos
- Uso de criteria para buscas relacionadas
- Limite de resultados para evitar sobrecarga

---

**Última Alteração:** 27/10/2025 às 10:40  
**Autor:** Bruno  
**Tipo:** Fórmula de Envio de E-mails NF-e  
**Versão:** 1.0