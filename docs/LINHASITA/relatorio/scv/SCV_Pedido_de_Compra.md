
## 📖 Descrição
Relatório de pedidos de compra da Linhasita com funcionalidade de emissão e envio automático por e-mail para fornecedores.

## 🎯 Finalidade
Gerar e distribuir pedidos de compra para fornecedores, automatizando o processo de solicitação de compras com envio de documentação por e-mail.

## 👥 Público-Alvo
- Departamento de Compras
- Fornecedores
- Gestores de Suprimentos
- Controle Patrimonial

## ⚙️ Configuração
**Recursos Necessários:**
- Arquivo `Logo2.png` - Logo secundária
- Arquivo `Logo3.png` - Logo terciária  
- Arquivo `NaoAutorizado.png` - Imagem para status não autorizado

**Localização:** `samdev/resources/linhasita/relatorios/scv/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos de compra
- `ABB01` - Cabeçalho pedidos
- `ABE01` - Entidades/Fornecedores
- `EAA0101` - Endereços de entrega
- `EAA0102` - Dados de contato
- `EAA0103` - Itens do pedido
- `ABE0104` - E-mails de fornecedores

**Entidades Envolvidas:**
- `Abe0104` - E-mails de entidades
- `Aab1008` - Configuração de e-mail do usuário
- `Aab10` - Usuário logado
- `Aac10` - Empresa ativa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| numeroInicial | Integer | Não | Número inicial do pedido | ≥ 0 |
| numeroFinal | Integer | Não | Número final do pedido | ≥ 0 |
| emissao | LocalDate[] | Sim | Período de emissão | Data inicial e final |
| entidades | List<Long> | Não | Fornecedores específicos | IDs das entidades |
| aprovado | Boolean | Sim | Filtro por status | true=aprovados, false=todos |
| enviaEmail | Boolean | Sim | Envio automático | true=enviar, false=apenas gerar |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abb01num | Número do pedido | Integer |
| abb01data | Data de emissão | Date |
| abb01aprovado | Status de aprovação | Boolean |
| eaa0102codigo | Código do fornecedor | String |
| eaa0102nome | Nome do fornecedor | String |
| eaa0102ni | CNPJ/CPF | String |
| eaa0101endereco | Endereço de entrega | String |
| abm01codigo | Código do item | String |
| eaa0103descr | Descrição do item | String |
| eaa0103qtComl | Quantidade | BigDecimal |
| eaa0103unit | Valor unitário | BigDecimal |
| eaa0103total | Valor total | BigDecimal |
| ipi_aliq | Alíquota IPI | BigDecimal |
| icm_aliq | Alíquota ICMS | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica números de pedido (inicial/final)
   - Valida período de emissão
   - Processa lista de fornecedores

2. **Busca de Dados**
   - Consulta pedidos com filtros aplicados
   - Busca informações completas (itens, endereços, contatos)
   - Aplica filtro de aprovação quando solicitado

3. **Processamento de Saída**
   - Carrega recursos visuais (logos)
   - Gera PDF do pedido de compra
   - Se solicitado, envia e-mail para fornecedores

4. **Envio por E-mail** (opcional)
   - Busca e-mails cadastrados do fornecedor
   - Valida configuração de e-mail do usuário
   - Anexa PDF e envia com corpo padronizado

## ⚠️ Regras de Negócio

### Validações
- Usuário deve ter e-mail configurado para envio
- Fornecedor deve ter e-mail cadastrado (classificação 9002)
- Período de emissão obrigatório
- Filtro de aprovação aplicado por padrão

### Processamento
- Considera apenas documentos de classe 0 (compras)
- Endereço principal de entrega
- Itens ordenados por sequência
- Calcula totais e impostos via campos JSON

### E-mail
- Assunto padronizado com número do pedido
- Corpo em HTML com formatação
- Anexo PDF com nome identificador
- Inclui assinatura digital se configurada

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Pedido de compra formatado | `gerarPDF()` |
| E-mail | Envio automático para fornecedores | `enviarEmail()` |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `javax.mail` - Envio de e-mails
- `multitec.utils` - Utilitários e e-mail

**Serviços:**
- `CAS1010Service` - Processamento de assinaturas de e-mail

**Configurações:**
- E-mail do usuário logado (`Aab1008`)
- E-mails de fornecedores (classificação 9002)

## 📝 Observações Técnicas

- Utiliza consulta SQL nativa para performance com múltiplos joins
- Processa campos de impostos diretamente do JSON da tabela
- Suporte a múltiplos anexos de e-mail
- Validação de recursos de imagem em tempo de execução
- Normalização de texto para remover acentos no corpo do e-mail