
## 📖 Descrição
Sistema de geração e envio de boletos bancários do Banco do Brasil para a Linhasita, com cálculo automático de código de barras e linha digitável.

## 🎯 Finalidade
Automatizar a emissão de boletos bancários com envio por e-mail, garantindo a correta formatação dos campos according às especificações do Banco do Brasil e registro de status de impressão.

## 👥 Público-Alvo
- Departamento Financeiro
- Cobrança
- Clientes/Fornecedores
- Faturamento

## ⚙️ Configuração
**Recursos Necessários:**
- Arquivo `LogoBancoBrasil.png` - Logo do banco
- Template Jasper `SCF_BoletoBancoBrasil` - Layout do boleto

**Localização:** `samdev/resources/linhasita/relatorios/scf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA01` - Documentos a receber
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades/Clientes
- `ABF01` - Dados bancários
- `ABE0101` - Endereços de cobrança
- `AAC10` - Dados da empresa

**Entidades Envolvidas:**
- `Daa01` - Documentos a receber
- `Abb01` - Cabeçalho documentos
- `Abe01` - Clientes
- `Aab1008` - Configuração de e-mail do usuário

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| dataProc | LocalDate | Sim | Data de processamento | Data atual |
| numeroInicial | Integer | Não | Número inicial do documento | 000000000 |
| numeroFinal | Integer | Não | Número final do documento | 999999999 |
| carteira | String | Não | Carteira bancária | Especificação BB |
| aceite | String | Não | Aceite do título | S/N |
| movimento | Integer | Não | Movimento específico | ID movimento |
| entidade | List<Long> | Não | Clientes específicos | IDs entidades |
| dataVenc | LocalDate[] | Não | Período de vencimento | Data inicial/final |
| enviaBoletoPorEmail | Boolean | Sim | Envio automático | true/false |

## 📋 Campos do Boleto

| Campo | Descrição | Tipo |
|-------|-----------|------|
| daa01nossoNum | Nosso número | String (10) |
| daa01dtVctoN | Data de vencimento | Date |
| daa01valor | Valor do título | BigDecimal |
| abe01nome | Nome do cliente | String |
| abe01ni | CNPJ/CPF | String |
| codigoBarras | Código de barras | String (44) |
| codigoLinhaDigitavel | Linha digitável | String |
| abf01agencia | Agência | String |
| abf01conta | Conta corrente | String |

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Processa filtros por número, vencimento, entidades
   - Valida dados bancários obrigatórios

2. **Busca de Títulos**
   - Consulta documentos em aberto do Banco do Brasil (código 001)
   - Aplica filtros de cobrança e vencimento
   - Busca por IDs específicos ou por critérios

3. **Cálculo de Campos Bancários**
   - Gera fator de vencimento (base 07/10/1997)
   - Calcula código de barras (módulo 11)
   - Monta linha digitável (módulo 10)
   - Define campo livre conforme especificação BB

4. **Processamento de Saída**
   - Gera PDF dos boletos
   - Se solicitado, envia e-mails para clientes
   - Registra status de impressão

5. **Envio por E-mail** (opcional)
   - Agrupa boletos por documento origem
   - Valida e-mails de cobrança dos clientes
   - Envia em lote com assinatura

## ⚠️ Regras de Negócio

### Validações
- Considera apenas banco 001 (Banco do Brasil)
- Apenas documentos não pagos (`daa01dtPgto is null`)
- Endereço de cobrança obrigatório (`abe0101cobranca = 1`)
- Usuário deve ter e-mail configurado para envio

### Cálculos Bancários
- **Fator Vencimento:** Dias desde 07/10/1997
- **Código Barras:** Módulo 11 com pesos 2-9
- **Linha Digitável:** Módulo 10 com formatação específica
- **Campo Livre:** 6 zeros + 17 nosso número + 2 carteira

### Formatação
- Nosso número com 10 dígitos
- Valores em centavos (multiplicados por 100)
- Campos numéricos com zeros à esquerda

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Boleto formatado | `gerarPDF()` |
| E-mail | Envio automático para clientes | `enviarBoletosPorEmail()` |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `javax.mail` - Envio de e-mails
- `multitec.utils` - Utilitários e cálculos

**Serviços:**
- `SCFService` - Serviços do módulo financeiro

**Configurações:**
- E-mail do usuário logado (`Aab1008`)
- E-mails de cobrança (classificação 01 e 03)

## 📝 Observações Técnicas

- Implementa cálculo completo de código de barras segundo especificação BB
- Suporte a múltiplos formatos de entrada (IDs específicos ou filtros)
- Envio assíncrono de e-mails em lote com Thread separada
- Validação de e-mails de cobrança com fallback para e-mails de endereço
- Registro automático de status de impressão dos documentos
- Formatação específica para Banco do Brasil (carteira, convenio, aceite)