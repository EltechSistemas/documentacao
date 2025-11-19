# SRF_Danfe.md

## 📖 Descrição
Sistema de geração e envio de Documento Auxiliar da Nota Fiscal Eletrônica (DANFE) para documentos fiscais, com suporte a impressão e distribuição automática por e-mail.

## 🎯 Finalidade
Gerar o DANFE em formato PDF para documentos fiscais eletrônicos, permitindo visualização, impressão e distribuição eletrônica automática para clientes e destinatários.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Clientes/Fornecedores
- Transportadoras

## ⚙️ Configuração
**Recursos Necessários:**
- Relatório `SRF_Danfe` - Geração do DANFE
- Sub-relatório `SRF_Danfe_S1` - Detalhamento de itens

**Localização:** `strema/relatorios/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `ABB01` - Centrais de documento
- `ABE01` - Entidades/clientes
- `EAA0101` - Endereços do documento
- `EAA0102` - Dados gerais do documento
- `EAA0103` - Itens do documento
- `EAA0113` - Duplicatas

**Recursos Externos:**
- `Logo.png` - Logotipo da empresa
- `canceladas.png` - Imagem para documentos cancelados
- `SemValorFiscal.png` - Imagem para documentos sem valor fiscal

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| numeroInicial | Integer | Sim | Número inicial do documento |
| numeroFinal | Integer | Sim | Número final do documento |
| entidade | List<Long> | Não | Filtro por entidades |
| emissao | LocalDate[] | Não | Período de emissão |
| entraSai | LocalDate[] | Não | Período de entrada/saída |
| eaa01id | Long | Não | ID específico do documento |
| entInicial | String | Não | Código inicial da entidade |
| entFinal | String | Não | Código final da entidade |
| serie | String | Não | Série do documento |
| enviaEmail | Boolean | Não | Flag para envio automático por e-mail |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Carrega recursos gráficos (logos, imagens)
   - Define parâmetros padrão do relatório
   - Configura sub-relatórios

2. **Busca de Documentos**
   - Filtra documentos por número, série, período e entidade
   - Valida documentos do modelo 55 (NF-e)
   - Ordena por número do documento

3. **Composição de Dados**
   - **Empresa:** Dados do emitente
   - **Documento:** Informações fiscais e numeração
   - **Entidade:** Dados do destinatário
   - **Itens:** Produtos/serviços com tributos
   - **Valores:** Totais e impostos
   - **Transporte:** Dados de frete e veículo

4. **Geração do PDF**
   - Processa relatório principal com sub-relatórios
   - Aplica formatação específica do DANFE
   - Gera arquivo PDF para download

5. **Envio por E-mail (Opcional)**
   - Busca e-mails do destinatário por classificação
   - Configura e-mail com anexo PDF
   - Envia automaticamente para contatos cadastrados

## ⚠️ Regras de Negócio

### Filtros de Documentos
- Apenas documentos do modelo 55 (NF-e)
- Documentos com emissão própria (eaa01emissao = 1)
- Ordenação por número do documento
- Suporte a filtros por série e período

### Validações de Dados
- Empresa deve ter município cadastrado
- Endereço principal do documento é obrigatório
- Município do endereço deve estar preenchido
- Dados de IE devem estar disponíveis por estado

### Estrutura do DANFE
- **Cabeçalho:** Dados do emitente e destinatário
- **Itens:** Descrição, quantidades, valores unitários e totais
- **Tributos:** ICMS, IPI, PIS, COFINS
- **Transporte:** Dados do frete e veículo
- **Duplicatas:** Parcelas e datas de vencimento
- **Informações Adicionais:** Observações fiscais

### Envio por E-mail
- Destinatários definidos por classificação "9001"
- Assinatura digital do usuário quando disponível
- Anexo PDF com nome personalizado
- Corpo da mensagem em HTML formatado

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| PDF | DANFE em formato PDF | DadosParaDownload |
| E-mail | Mensagem com anexo (opcional) | Email |

## 🔧 Dependências

**Bibliotecas:**
- `JasperReports` - Geração de relatórios
- `JavaMail` - Envio de e-mails
- `multiorm` - Persistência e consultas

**Serviços:**
- `CAS1010Service` - Processamento de assinaturas de e-mail

## 📝 Observações Técnicas

- **Processamento:** Batch para múltiplos documentos
- **Performance:** Consultas otimizadas com joins
- **Formatação:** Layout fiel ao DANFE oficial
- **Internacionalização:** Suporte a caracteres especiais

### Recursos Gráficos
- Logotipo da empresa em PNG
- Carimbo para documentos cancelados
- Marca d'água para documentos sem valor fiscal
- Formatação específica para cada tipo de documento

### Tratamento de Dados
- Normalização de caracteres para e-mail
- Formatação de valores monetários
- Conversão de datas e horas
- Cálculo de percentuais de impostos

### Integração com E-mail
- Configuração SMTP do usuário logado
- Suporte a anexos em Base64
- Assinatura digital em HTML
- Destinatários múltiplos por classificação