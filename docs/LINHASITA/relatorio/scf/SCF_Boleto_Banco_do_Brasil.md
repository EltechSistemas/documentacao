# SCF - Boleto Banco do Brasil (Linhasita)

## 📖 Descrição
Relatório para geração e envio de boletos bancários do Banco do Brasil, incluindo cálculo de código de barras, linha digitável, e funcionalidade de envio por email com personalização para a empresa Linhasita.

## 🎯 Finalidade
Automatizar a geração de boletos bancários do Banco do Brasil, calcular códigos de barras e linhas digitáveis corretamente, e fornecer opção de envio automático por email aos clientes.

## 👥 Público-Alvo
- Financeiro/Contas a Receber
- Cobrança
- Faturamento
- Administrativo

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Daa01` - Documentos financeiros
- `Abb01` - Central de documento
- `Abe01` - Entidades (clientes)
- `Abe0101` - Endereços da entidade
- `Abe0104` - Emails da entidade
- `AbF01` - Bancos
- `Aah01` - Tipos de documento
- `Aac10` - Empresas
- `Aag02` - Estados (UF)
- `Aag0201` - Municípios
- `Abb0102` - Relacionamento de documentos
- `Aab10` - Usuários do sistema
- `Aab1008` - Configurações de email do usuário

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valor Padrão |
|-----------|------|-------------|-----------|--------------|
| movimento | Integer | Não | Tipo de movimento financeiro | - |
| dataProc | LocalDate | Sim | Data de processamento | Data atual |
| carteira | String | Não | Carteira do banco | - |
| aceite | String | Não | Aceite do título | - |
| instrucao1 | String | Não | Instrução bancária 1 | - |
| instrucao2 | String | Não | Instrução bancária 2 | - |
| instrucao3 | String | Não | Instrução bancária 3 | - |
| numeroInicial | Integer | Sim | Número inicial do documento | 000000000 |
| numeroFinal | Integer | Sim | Número final do documento | 999999999 |
| entidade | List<Long> | Não | IDs das entidades | - |
| dataVenc | LocalDate[] | Não | Período de vencimento | - |
| eaa01id | Long | Não | ID do documento fiscal (SRF) | - |
| idDoc | Long | Não | ID do documento financeiro | - |
| abe01id | Long | Não | ID da entidade (filtro SCF0101) | - |
| aah01id | Long | Não | ID do tipo de documento | - |
| parcela | String | Não | Número da parcela | - |
| idsDocs | List<Long> | Não | IDs dos documentos (filtro SCF1001) | - |
| enviaBoletoPorEmail | Boolean | Sim | Enviar boleto por email | false |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Definição dos valores padrão dos filtros
- Carregamento dos parâmetros fornecidos pelo usuário
- Configuração do logo do Banco do Brasil
- Preparação dos parâmetros do relatório

### 2. **Identificação dos Documentos**
- **Via ID único**: Quando fornecido `daa01id` ou `eaa01id`
- **Via filtros múltiplos**: Quando usados filtros de carteira, aceite, movimento, etc.
- **Via lista de IDs**: Quando fornecida lista `idsDocs` do SCF1001
- Validação de documentos pendentes e com nosso número

### 3. **Busca de Dados dos Boletos**
- Consulta ao banco com múltiplos joins para obter todos os dados necessários
- Filtragem por banco do Brasil (número 001)
- Inclusão de dados da empresa, entidade, endereços e configurações bancárias
- Aplicação de filtros de segurança (where padrão)

### 4. **Cálculo dos Códigos do Boleto**
- **Fator de vencimento**: Cálculo baseado na data base 07/10/1997
- **Campo livre**: Construção conforme especificação do Banco do Brasil
- **Código de barras**: Cálculo com módulo 11 e dígito verificador
- **Linha digitável**: Formatação com pontos e espaços padrão

### 5. **Processamento por Documento**
- Para cada documento encontrado:
  - Extração de dados da entidade (nome, endereço, CNPJ/CPF)
  - Obtenção de dados bancários (agência, conta, convênio)
  - Cálculo dos códigos do boleto
  - Montagem do TableMap com todos os dados

### 6. **Envio por Email (Opcional)**
- Agrupamento de boletos por documento de origem
- Geração de PDF individual por grupo
- Busca de emails do cliente para envio
- Configuração do email com anexo, corpo personalizado e assinatura
- Envio assíncrono em thread separada

### 7. **Atualização de Status**
- Marcação dos documentos como impressos (`daa01imprDoc = 1`)
- Persistência no banco de dados
- Tratamento de erros para evitar falhas no processo principal

### 8. **Geração do Relatório**
- Geração do PDF final com todos os boletos
- Formatação conforme template JasperReports
- Retorno do arquivo para download

## ⚠️ Regras de Negócio

### Filtros de Documentos
- Apenas documentos do Banco do Brasil (número 001)
- Documentos não pagos (`daa01dtPgto is null`)
- Com nosso número preenchido (`daa01nossonum is not null`)
- Endereço de cobrança da entidade (`abe0101cobranca = 1`)

### Cálculo do Boleto (Banco do Brasil)
- **Fator de vencimento**: Dias desde 07/10/1997, com ajuste para >9999
- **Campo livre**: 6 zeros + 17 dígitos do nosso número + 2 dígitos da carteira
- **Código de barras**: 3 bancos + 9 + 4 fator + 10 valor + campo livre, com DV módulo 11
- **Linha digitável**: Formato grupos de 5 com pontos e espaços

### Validações de Dados
- Banco deve existir e ter configuração JSON
- Convênio deve estar configurado no JSON do banco
- Entidade deve ter endereço de cobrança
- Email do usuário logado deve estar configurado para envio

### Envio por Email
- Agrupamento por documento de origem (nota fiscal)
- Corpo do email personalizado com dados da empresa Linhasita
- Anexo único PDF com todos os boletos do grupo
- CC automático para `boleto@linhasita.com.br`
- Assinatura do usuário (arquivo ou texto)
- Envio assíncrono para não bloquear interface

### Formatação de Campos
- Nosso número: 10 dígitos com zeros à esquerda
- Valores: Multiplicados por 100 para centavos
- CNPJ/CPF: Apenas números
- Datas: Formato dd/MM/yyyy

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração de boletos.

### `buscarDadosBoleto()`
Busca documentos financeiros com base em múltiplos filtros.

### `buscarIdsDocsSCFPeloIdDocSRF()`
Relaciona documentos financeiros com documentos fiscais.

### `montarCodigoBarras()`
Calcula o código de barras do boleto conforme especificação do Banco do Brasil.

### `montarLinhaDigitavel()`
Formata a linha digitável do boleto.

### `enviarBoletosPorEmail()`
Gerencia o processo de agrupamento e envio de boletos por email.

### `enviarBoletoPorEmail()`
Configura e prepara um email individual com anexo PDF.

## 🔧 Cálculos Bancários

### Fator de Vencimento

fator = data_vencimento - 07/10/1997 (em dias)
Se fator > 9999: fator = fator - 10000 + 1000


### Módulo 11 (Código de Barras)
- Pesos de 2 a 9, reiniciando após 9
- DV = 11 - (soma % 11)
- Se DV ∈ {0,10,11} → DV = 1

### Módulo 10 (Linha Digitável)
- Pesos alternados 2 e 1
- Soma com decomposição de dígitos (ex: 14 → 1+4)
- DV = múltiplo de 10 mais próximo - soma

### Campo Livre (Banco do Brasil)

### Módulo 11 (Código de Barras)
- Pesos de 2 a 9, reiniciando após 9
- DV = 11 - (soma % 11)
- Se DV ∈ {0,10,11} → DV = 1

### Módulo 10 (Linha Digitável)
- Pesos alternados 2 e 1
- Soma com decomposição de dígitos (ex: 14 → 1+4)
- DV = múltiplo de 10 mais próximo - soma

### Campo Livre (Banco do Brasil)

### Módulo 11 (Código de Barras)
- Pesos de 2 a 9, reiniciando após 9
- DV = 11 - (soma % 11)
- Se DV ∈ {0,10,11} → DV = 1

### Módulo 10 (Linha Digitável)
- Pesos alternados 2 e 1
- Soma com decomposição de dígitos (ex: 14 → 1+4)
- DV = múltiplo de 10 mais próximo - soma

### Campo Livre (Banco do Brasil)
000000 + nosso_numero(17) + carteira(2)

## 📊 Estrutura de Saída

**Dados do Boleto:**
- Dados da entidade (nome, CNPJ/CPF, endereço completo)
- Dados da empresa (razão social, endereço, CNPJ)
- Dados bancários (agência, conta, convênio, carteira)
- Valores (vencimento, valor, nosso número)
- Códigos calculados (barras, linha digitável)

**Formato de Saída:**
- **PDF**: Boletos formatados para impressão
- **Email**: PDF anexado com corpo personalizado

**Parâmetros do Relatório:**
- Data de processamento
- Instruções bancárias
- Logo do banco
- Dados da empresa

## 📝 Observações Técnicas

### Integração com Múltiplas Telas
- **SCF0101**: Filtros básicos por entidade, tipo, parcela
- **SCF1001**: Seleção múltipla de documentos via lista de IDs
- **SRF**: Relacionamento via `eaa01id` para documentos fiscais

### Tratamento de JSON
- Configurações bancárias armazenadas em `abf01json`
- Dados adicionais do documento em `daa01json`
- Campos específicos: carteira, aceite, convênio, juros

### Performance
- Consultas otimizadas com joins necessários
- Processamento em lote para múltiplos documentos
- Envio de email assíncrono em thread separada
- Uso de TableMap para manipulação eficiente de dados

### Segurança
- Aplicação de where padrão por empresa/usuário
- Validação de permissões através da classe base
- Logs de erro no envio de emails sem interromper processo

### Personalização Empresarial
- Conteúdo do email específico para Linhasita
- Contatos e informações da empresa no corpo do email
- Logo do Banco do Brasil embutido no relatório
- CC automático para email da empresa

---

**Última Alteração:** 25/11/2025 às 11:11  
**Autor:** Bruno  
**Tipo:** Relatório de Boletos Bancários  
**Versão:** 1.0