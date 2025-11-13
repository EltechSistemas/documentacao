# SRF_Danfe.md

## 📖 Descrição
Sistema de geração do Documento Auxiliar da Nota Fiscal Eletrônica (DANFE) para a Linhasita, responsável pela impressão do representação gráfica da NF-e.

## 🎯 Finalidade
Gerar o DANFE de documentos fiscais eletrônicos para acompanhamento, conferência e documentação de operações comerciais.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Transportadoras
- Clientes
- Almoxarifado

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SRF_Danfe` - Layout principal do DANFE
- Template Jasper `SRF_Danfe_S1` - Layout dos itens
- Arquivos de imagem (Logo, Cancelada, SemValorFiscal)

**Localização:** `strema/relatorios/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `ABB01` - Cabeçalho de documentos
- `AAC10` - Empresas/Filiais
- `ABE01` - Entidades/Clientes
- `EAA0101` - Endereços do documento
- `EAA0102` - Dados gerais do documento
- `EAA0103` - Itens do documento

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Abb01` - Cabeçalho documentos
- `Aac10` - Empresa emissora
- `Abe01` - Cliente/destinatário
- `Eaa0103` - Itens da nota

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| numeroInicial | Integer | Não | Número inicial do documento |
| numeroFinal | Integer | Não | Número final do documento |
| entInicial | String | Não | Código inicial da entidade |
| entFinal | String | Não | Código final da entidade |
| emissao | LocalDate[] | Não | Período de emissão |
| entraSai | LocalDate[] | Não | Período de entrada/saída |
| serie | String | Não | Série do documento |

## 📋 Estrutura do DANFE

### Seção 1 - Dados do Emitente
- Razão social e CNPJ
- Endereço completo
- Inscrição Estadual
- Contato (telefone/email)

### Seção 2 - Dados do Destinatário
- Razão social e CNPJ/CPF
- Endereço de entrega
- Inscrição Estadual
- Indicador de IE do destinatário

### Seção 3 - Dados do Documento
- Número e série da NF-e
- Data de emissão e saída
- Chave de acesso
- Protocolo de autorização

### Seção 4 - Dados de Transporte
- Modalidade do frete
- Dados do veículo (placa, RNTRC)
- Dados do transportador
- Volumes e pesos

### Seção 5 - Itens da Nota
- Código e descrição do produto
- NCM e CFOP
- Quantidade e valor unitário
- Valor total do item
- Tributos incidentes

### Seção 6 - Cálculo de Impostos
- Base de cálculo do ICMS
- Valor do ICMS
- Valor do IPI
- Valor do PIS/COFINS

### Seção 7 - Duplicatas
- Número das parcelas
- Datas de vencimento
- Valores das duplicatas

## 🔄 Fluxo do Processo

1. **Busca de Documentos**
   - Filtra por número, série, período e entidade
   - Apenas documentos modelo 55 (NF-e)
   - Apenas documentos de emissão própria

2. **Composição dos Dados**
   - Dados da empresa emitente
   - Dados do destinatário
   - Dados do documento fiscal
   - Itens e valores tributários

3. **Processamento de Itens**
   - Busca todos os itens do documento
   - Calcula totais e impostos
   - Processa lotes e séries

4. **Geração do Relatório**
   - Cria datasource principal
   - Adiciona sub-relatório de itens
   - Gera PDF final

## ⚠️ Regras de Negócio

### Validações Obrigatórias
- Município da empresa deve estar cadastrado
- Endereço principal do documento deve existir
- Município do destinatário deve estar informado
- UF deve estar definida no endereço

### Classificações
- **Modalidade de Frete:** 0-Remetente, 1-Destinatário, 2-Terceiros, 3-Próprio remetente, 4-Próprio destinatário
- **Tipo de Documento:** Apenas NF-e própria (modelo 55)
- **Status:** Apenas documentos não cancelados

### Cálculos Tributários
- ICMS: base de cálculo e valor
- IPI: valor do imposto
- PIS/COFINS: valores federais
- Percentuais sobre o total do documento

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | DANFE completo | `gerarPDF()` |

## 🔧 Dependências

**Recursos Gráficos:**
- `Logo.png` - Logo da empresa
- `canceladas.png` - Marca d'água para documentos cancelados
- `SemValorFiscal.png` - Marca para documentos sem valor fiscal

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Acesso a dados
- `java.time` - Manipulação de datas

## 📝 Observações Técnicas

- Suporte a múltiplos documentos em lote
- Layout fiel ao padrão oficial da DANFE
- Tratamento de documentos cancelados
- Cálculo automático de percentuais tributários
- Integração com dados de transporte e frete
- Suporte a endereço de entrega diferenciado
- Processamento de lotes e séries dos itens
- Formatação específica para dados fiscais