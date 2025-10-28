# SRF - Enviar Emails NFe (Equilibrio)

## 📖 Descrição
Fórmula especializada para envio automatizado de e-mails relacionados a Notas Fiscais Eletrônicas no módulo Equilibrio, com tratamento de caracteres especiais e cópia obrigatória para e-mail interno.

## 🎯 Finalidade
Automatizar o envio de e-mails para clientes com documentos fiscais eletrônicos (XML, DANFE, boletos) incluindo cópia para contato@equilibrioind.com.br e tratamento de normalização de caracteres.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Cobrança
- Administração

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
- **Normalização de caracteres** para remover acentos

### 3. **Definição dos Destinatários por Tipo**
- **Cancelamento/CC-e**: Apenas e-mail principal do cliente com XML
- **Faturamento**: E-mail principal com XML + DANFE + cópia interna
- **Cobrança**: E-mail específico de cobrança com boleto
- **Cópia Interna**: contato@equilibrioind.com.br (apenas faturamento)

### 4. **Configuração dos Anexos**
- **Cancelamento/CC-e**: Apenas XML
- **Faturamento**: XML + DANFE (boleto opcional)
- **Cobrança**: Apenas boleto
- **Cópia Interna**: XML + DANFE

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
- **Faturamento/Cópia Interna**: Remetente 2 (Faturamento)
- **Cobrança**: Remetente 1 (Cobrança)

### Funcionalidades Específicas
- **Normalização de Caracteres**: Remove acentos do nome do cliente
- **Cópia Obrigatória**: contato@equilibrioind.com.br para todos os faturamentos
- **Transportadora**: Código comentado - não envia e-mails para transportadora

### Validações
- Não processa se não houver e-mail principal no documento
- Verifica diferença entre e-mails de faturamento e cobrança
- Envia boleto junto com faturamento se mesmo e-mail

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de envio de e-mails.

### `comporCorpoMsg(Aaa16 aaa16, Eaa01 eaa01, Abe01 abe01, Abb01 abb01, Long eaa0114id)`
Compoi o corpo do e-mail conforme o tipo de operação com formatação HTML e normalização de caracteres.

### `obterEmailCobranca(Long eaa01id, Long abe01id, Eaa0102 eaa0102)`
Busca o e-mail de cobrança seguindo a hierarquia definida.

## 📊 Estrutura de Saída

**EmailNFeDto:**
- `assunto` - Assunto do e-mail formatado
- `corpo` - Corpo em HTML da mensagem (caracteres normalizados)
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
- `java.text.Normalizer` - Normalização de caracteres

**Módulo:** Equilibrio

## 📝 Observações Técnicas

### Normalização de Caracteres
- Uso de `Normalizer.normalize(corpo, Normalizer.Form.NFD).replaceAll("[^\\p{ASCII}]", "")`
- Remove acentos e caracteres especiais do nome do cliente
- Aplica tanto no corpo da mensagem quanto no nome do cliente

### Cópia Interna Obrigatória
- E-mail: contato@equilibrioind.com.br
- Apenas para operações de faturamento (não cancelamento/CC-e)
- Recebe XML e DANFE para controle interno

### Funcionalidades Comentadas
- Método `obterEmailTransportadoraDespacho()` disponível mas não utilizado
- Bloco de código para transportadora comentado

---

**Última Alteração:** 27/10/2025 às 10:48  
**Autor:** Bruno  
**Tipo:** Fórmula de Envio de E-mails NF-e (Equilibrio)  
**Versão:** 1.0