# SCV_Orcamento.md

## 📖 Descrição
Sistema de geração de relatórios de orçamentos para o SCV (Sistema de Controle de Vendas) da Strema, com funcionalidades de impressão e envio automático por e-mail.

## 🎯 Finalidade
Gerar relatórios detalhados de orçamentos comerciais com opções de visualização, exportação e distribuição automatizada para clientes.

## 👥 Público-Alvo
- Departamento Comercial
- Vendedores
- Atendimento ao Cliente
- Gestão Comercial

## ⚙️ Configuração
**Recursos Necessários:**
- Classe `SCV_Orcamento` - Relatório de orçamentos
- Arquivos de imagem (Logo.png, canceladas.png)

**Localização:** `strema/relatorios/scv/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `CBE10` - Cabeçalho de orçamentos
- `ABB01` - Documentos fiscais
- `ABE01` - Entidades/Clientes
- `CBE1001` - Itens do orçamento
- `ABM01` - Materiais/Produtos
- `AAB10` - Usuários
- `ABE0104` - Classificação de entidades (e-mails)

**Entidades Envolvidas:**
- `Cbe10` - Orçamento
- `Abb01` - Documento fiscal
- `Abe01` - Entidade/Cliente
- `Aab10` - Usuário
- `Abe0104` - Classificação de entidade
- `Aac10` - Empresa

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| numeroInicial | Integer | Não | Número inicial do orçamento |
| numeroFinal | Integer | Não | Número final do orçamento |
| entidades | List<Long> | Não | Lista de entidades para filtro |
| emissao | LocalDate[] | Não | Período de emissão |
| status | List<Integer> | Sim | Status dos orçamentos (0-Criado, 1-Concluído, 2-Cancelado) |
| impressao | Integer | Sim | Tipo de saída (0-PDF, 1-XLSX) |
| enviaEmail | Boolean | Não | Enviar por e-mail automaticamente |
| tipoSCV7010 | Long | Não | Filtro específico da tela SCV7010 |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF/XLSX | Relatório formatado | Arquivo |
| E-mail | Orçamento enviado por e-mail | Mensagem |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define valores padrão para filtros
   - Carrega logos e recursos visuais
   - Compõe dados da empresa

2. **Processamento de Filtros**
   - Aplica filtros de número, entidade, data e status
   - Processa filtro específico da tela SCV7010
   - Valida parâmetros obrigatórios

3. **Busca de Dados**
   - Executa consulta SQL com múltiplos joins
   - Aplica where padrão do sistema
   - Ordena resultados por número e código do material

4. **Geração de Saída**
   - Gera PDF ou XLSX conforme seleção
   - Opcionalmente envia e-mail com anexo
   - Aplica formatação e assinatura de e-mail

## ⚠️ Regras de Negócio

### Filtros e Validações
- Status padrão: Criado e Concluído
- Filtro por número range opcional
- Filtro por período de emissão
- Where padrão aplicado para segurança

### Envio de E-mail
- E-mail obrigatório no cadastro do usuário
- Destinatários obtidos da classificação "9001"
- Assinatura personalizada quando disponível
- Anexo PDF com nome personalizado
- Corpo do e-mail com tratamento de caracteres

### Dados da Empresa
- Razão social, CNPJ, IE
- Endereço completo formatado
- Contatos e site
- Dados fiscais por UF

## 🎨 Estrutura do Relatório

**Cabeçalho:**
- Logo da empresa
- Dados cadastrais da empresa
- Filtros aplicados

**Detalhes do Orçamento:**
- Número, data, cliente
- Condição de pagamento
- Itens com código, descrição, quantidade, unitário, total
- Tributos (IPI, ICMS, ST)
- Observações internas

**Rodapé:**
- Totais do documento
- Status do orçamento
- Informações de frete

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Critérios e joins
- `multitec.utils` - Utilitários e e-mail
- `jasperreports` - Geração de relatórios
- `javax.mail` - Envio de e-mail

**Serviços:**
- `CAS1010Service` - Processamento de assinaturas de e-mail

**Consultas:**
- Busca de orçamentos com múltiplos relacionamentos
- Dados da empresa ativa
- E-mails de destino por classificação
- Configuração de e-mail do usuário

## 📝 Observações Técnicas

- **Recursos Visuais**: Logos carregadas dinamicamente do filesystem
- **Formatação de Dados**: Endereços concatenados, telefones formatados
- **Tratamento de Caracteres**: Normalização para remover acentos no e-mail
- **Segurança**: Aplicação de where padrão em todas as consultas
- **Performance**: Joins otimizados com fetch estratégico

## 🔄 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo do relatório.

### `comporDadosEmpresa()`
Busca e formata dados cadastrais da empresa ativa.

### `buscarDadosOrcamento()`
Executa consulta principal com todos os filtros aplicados.

### `enviarEmail()`
Processa envio automático do orçamento por e-mail.

### `buscarEmailDestino()`
Obtém lista de e-mails de destino baseado na classificação.

## 💡 Template de E-mail
**Assunto**: "PROPOSTA COMERCIAL STREMA BATERIAS N° [número]"

**Corpo**: