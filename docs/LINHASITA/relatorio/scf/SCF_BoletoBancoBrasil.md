# SCF_BoletoBancoBrasil - Geração e Envio de Boletos Banco do Brasil

## 📖 Descrição
Sistema completo de geração, impressão e envio por email de boletos bancários do Banco do Brasil, seguindo o layout CNAB 400 com cálculo automático de código de barras, linha digitável e validações fiscais.

## 🎯 Finalidade
Automatizar todo o processo de cobrança bancária, desde a geração do boleto até o envio por email, garantindo conformidade com as regras do Banco do Brasil e proporcionando eficiência operacional para o departamento financeiro.

## 👥 Público-Alvo
- Departamento Financeiro
- Tesouraria
- Cobrança
- Faturamento

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| movimento | Integer | Não | Movimento específico |
| dataProc | LocalDate | Sim | Data de processamento |
| carteira | String | Não | Carteira bancária |
| aceite | String | Não | Aceite do título |
| instrucao1/2/3 | String | Não | Instruções do boleto |
| numeroInicial/Final | Integer | Não | Range de números |
| entidades | List<Long> | Não | Entidades específicas |
| dataVenc | LocalDate[] | Não | Período de vencimento |
| eaa01id | Long | Não | ID documento SRF |
| daa01id | Long | Não | ID documento SCF |
| enviaBoletoPorEmail | Boolean | Não | Envio automático |

## 📋 Estrutura do Boleto

### Dados do Cedente (Empresa):
- Razão social, endereço, CNPJ
- Agência, conta, dígitos
- Código do convênio

### Dados do Sacado (Cliente):
- Nome, CNPJ/CPF, endereço
- Município, UF, CEP
- Email para envio

### Dados do Título:
- Nosso número, número documento
- Data vencimento, valor
- Parcela, juros
- Código de barras, linha digitável

## 🔄 Fluxo do Processo

### 1. **Validação e Filtragem**
- Processa múltiplas origens de dados (SRF, SCF, filtros)
- Aplica filtros por carteira, aceite, movimento
- Valida existência do banco e dados obrigatórios

### 2. **Cálculo de Componentes do Boleto**
- **Fator vencimento**: Dias desde 07/10/1997
- **Campo livre**: Conforme regras BB
- **Código de barras**: Com dígito verificador
- **Linha digitável**: Formatada para impressão

### 3. **Geração do PDF**
- Monta dados para template JasperReports
- Aplica logo do Banco do Brasil
- Formata instruções e dados

### 4. **Envio por Email (Opcional)**
- Agrupa boletos por documento origem
- Busca emails do cliente
- Envia em thread separada
- Atualiza status de impressão

## ⚠️ Regras de Negócio

### Validações de Dados:
- Banco deve existir (abf01id obrigatório)
- Documento não pode estar pago (daa01dtPgto null)
- Endereço de cobrança obrigatório (abe0101cobranca = 1)
- Email destino obrigatório para envio

### Cálculos Bancários:
- **Fator Vencimento**: Base 07/10/1997, ajuste para >9999 dias
- **Módulo 11**: Dígito verificador código barras
- **Módulo 10**: Dígitos verificadores linha digitável
- **Valor**: Multiplicado por 100 (sem decimais)

### Layout Banco do Brasil:
- Código banco: 001
- Moeda: 9 (Real)
- Campo livre: 6 zeros + 17 nosso número + 2 carteira
- Formatação específica linha digitável

### Envio de Email:
- Agrupamento por documento origem
- Anexo PDF único por grupo
- Cópia para boleto@linhasita.com.br
- Assinatura do usuário logado
- Thread separada para não bloquear interface

## 🎨 Saídas Geradas

| Tipo | Formato | Descrição |
|------|---------|-----------|
| PDF | Boleto bancário | Layout padrão BB |
| Email | HTML + Anexo | Com instruções e contatos |
| Banco | Atualização status | daa01imprDoc = 1 |

## 🔧 Dependências

### Bibliotecas:
- `Apache JasperReports` - Geração PDF
- `JavaMail` - Envio de emails
- `MultiORM` - Acesso a dados

### Serviços:
- `SCFService` - Serviços financeiros
- `FileSystem` - Manipulação de arquivos

### Entidades:
- `Daa01` - Documentos financeiros
- `Abe01` - Entidades (clientes)
- `Abf01` - Bancos
- `Aac10` - Empresas
- `Aab10` - Usuários

## 📝 Observações Técnicas

### Performance:
- Processamento em lote para múltiplos boletos
- Thread separada para envio de emails
- Consultas otimizadas com filtros no banco

### Segurança:
- Validação de dados antes do processamento
- Verificação de emails válidos
- Prevenção de duplo processamento

### Tratamento de Erros:
- Validação de bancos suportados (apenas BB)
- Proteção contra divisão por zero
- Tratamento de campos nulos
- Log de inconsistências

### Customizações:
- Logo específica do Banco do Brasil
- Instruções personalizáveis
- Texto de email configurável
- Assinatura digital do usuário

### Limitações:
- Suporte apenas ao Banco do Brasil (001)
- Formato CNAB 400 específico
- Estrutura fixa de campo livre