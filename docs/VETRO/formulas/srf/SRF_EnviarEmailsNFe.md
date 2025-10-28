# SRF - Enviar Emails NFe (Vetro)

## 📖 Descrição
Fórmula especializada para envio automatizado de e-mails relacionados a Notas Fiscais Eletrônicas no módulo Vetro, incluindo faturamento, cobrança, cancelamentos e cartas de correção.

## 🎯 Finalidade
Automatizar o envio de e-mails para clientes e transportadoras com documentos fiscais eletrônicos (XML, DANFE, boletos) de acordo com o tipo de operação, adaptado para as necessidades específicas do módulo Vetro.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Cobrança
- Logística/Transporte

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
- Composição do assunto com nome da empresa e número da nota

### 2. **Composição do Corpo da Mensagem**
- **Cancelamento**: Inclui motivo do cancelamento (Aae11)
- **Carta de Correção**: Inclui texto da correção (Eaa0114)
- **Faturamento Normal**: Mensagem padrão de envio de NF-e
- Formatação HTML com dados do cliente e chave de acesso

### 3. **Definição dos Destinatários por Tipo**
- **Cancelamento/CC-e**: Apenas e-mail principal do cliente com XML
- **Faturamento**: E-mail principal com XML + DANFE
- **Cobrança**: E-mail específico de cobrança com boleto
- **Transportadora**: E-mail da empresa de despacho com XML + DANFE

### 4. **Configuração dos Anexos**
- **Cancelamento/CC-e**: Apenas XML
- **Faturamento**: XML + DANFE (boleto opcional)
- **Cobrança**: Apenas boleto
- **Transportadora**: XML + DANFE

## ⚠️ Regras de Negócio

### Tipos de Operação
- **Cancelamento**: Aaa16.TIPO_CANCELAMENTO_NFE
- **Carta de Correção**: Aaa16.TIPO_CCEEVENTO  
- **Faturamento**: Demais tipos

### Hierarquia de E-mails de Cobrança
1. E-mail específico no documento (Eaa0101) marcado como cobrança
2. E-mail principal da entidade (Abe0101) marcado como cobrança
3. E-mail principal do documento (Eaa0102)

### Configuração de Remetentes
- **Faturamento/Transportadora**: Remetente 2 (Faturamento)
- **Cobrança**: Remetente 1 (Cobrança)

### Validações
- Não processa se não houver e-mail principal no documento
- Verifica diferença entre e-mails de faturamento e cobrança
- Transportadora apenas se houver empresa de despacho informada

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de envio de e-mails.

### `comporCorpoMsg(Aaa16 aaa16, Eaa01 eaa01, Abe01 abe01, Abb01 abb01, Long eaa0114id)`
Compoi o corpo do e-mail conforme o tipo de operação com formatação HTML.

### `obterEmailCobranca(Long eaa01id, Long abe01id, Eaa0102 eaa0102)`
Busca o e-mail de cobrança seguindo a hierarquia definida.

### `obterEmailTransportadoraDespacho(Eaa0102 eaa0102)`
Busca e-mail da transportadora responsável pelo despacho.

## 📊 Estrutura de Saída

**EmailNFeDto:**
- `assunto` - Assunto do e-mail formatado
- `corpo` - Corpo em HTML da mensagem
- `emailsDestinoPara` - Lista de destinatários
- `enviarXML` - Indica se envia arquivo XML
- `enviarDanfe` - Indica se envia DANFE
- `enviarBoleto` - Indica se envia boleto
- `emailRemetente` - Tipo de remetente (1: Cobrança, 2: Faturamento)

**Lista de E-mails Gerados:**
- Retornada no parâmetro `emails` para processamento pela mensageria

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e joins para consultas
- `sam.dicdados` - Tipos de fórmula
- `sam.dto.srf` - DTOs específicos do módulo SRF
- `sam.model` - Entidades do sistema

**Módulo:** Vetro

## 📝 Observações Técnicas

### Formatação HTML
- Corpo dos e-mails em HTML com tags básicas
- Links para portal da NF-e e site da Multitec
- Dados do cliente, CNPJ e chave de acesso
- Protocolo de autorização da SEFAZ

### Controle de Anexos
- XML sempre enviado para cliente e transportadora
- DANFE para faturamento e transportadora
- Boleto apenas para cobrança (separado ou junto)

### Performance
- Consultas otimizadas com campos específicos
- Uso de criteria para buscas relacionadas
- Carregamento lazy de entidades

---

**Última Alteração:** 27/10/2025 às 10:42  
**Autor:** Bruno  
**Tipo:** Fórmula de Envio de E-mails NF-e (Vetro)  
**Versão:** 1.0