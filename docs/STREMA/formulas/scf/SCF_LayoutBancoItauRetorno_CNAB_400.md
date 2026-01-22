# SCF_LayoutBancoItauRetorno_CNAB_400

## 📖 Descrição
Fórmula de retorno de cobrança responsável por processar arquivos CNAB 400 do Banco Itaú, interpretando registros de liquidação, validando documentos financeiros e preparando informações para exibição e baixa de títulos no sistema SCF.

## 🎯 Finalidade
Realizar a leitura e validação de arquivos de retorno bancário CNAB 400, garantindo a consistência dos dados financeiros, identificação correta dos documentos (`Daa01`) e tratamento de ocorrências de cobrança informadas pelo banco.

## 👥 Público-Alvo
- Financeiro
- Contas a Receber
- Cobrança
- Controladoria
- TI / Desenvolvimento

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SCF_LayoutBancoItauRetorno_CNAB_400`

**Tipo de Fórmula:**
- `SCF_RETORNO_DE_COBRANCA`

## 📊 Dados e Fontes
**Entidades Principais:**
- `Daa01` – Títulos a receber
- `Abf20` – Plano financeiro (PLF)

**Fontes de Dados:**
- Arquivo texto CNAB 400 (Banco Itaú)
- Parâmetros internos do SCF

**Campos e Estruturas Utilizadas:**
- Registro tipo `1` (detalhe)
- Campos posicionais do layout CNAB 400
- JSON de campos customizados do `Daa01`

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|----------|------|-------------|----------|
| registros | Arquivo Texto | Sim | Conteúdo do arquivo CNAB 400 de retorno |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|------|-----------|------|
| tmList | Lista de registros processados | List<TableMap> |
| daa01 | Documento financeiro validado | Daa01 |
| abf20id | Identificador do plano financeiro | Long |
| ocorrencia | Descrição da ocorrência bancária | String |
| inconsistencias | Lista de inconsistências encontradas | List<String> |

## 🔄 Fluxo do Processo

1. **Inicialização**
    - Cria lista de resultados (`tmList`)
    - Abre arquivo CNAB 400 recebido
    - Instancia serviço `SCFService`

2. **Leitura do Arquivo**
    - Ignora o header do arquivo
    - Processa apenas registros do tipo `1`

3. **Identificação do Documento**
    - Extrai o ID do documento do retorno
    - Remove zeros à esquerda
    - Tenta localizar o documento pelo ID ou campo customizado

4. **Validações do Documento**
    - Documento inexistente
    - Documento já quitado
    - Divergência de valores
    - Ocorrência bancária inexistente
    - Ausência de data de pagamento

5. **Tratamento Financeiro**
    - Atualiza data de pagamento
    - Calcula valor líquido
    - Atualiza juros e descontos no JSON do documento

6. **Mapeamento da Ocorrência**
    - Obtém descrição da ocorrência
    - Define plano financeiro (PLF) quando aplicável

7. **Preparação do Retorno**
    - Consolida inconsistências
    - Retorna lista de documentos válidos e inválidos

## ⚠️ Regras de Negócio

### Validações Obrigatórias
- Documento deve existir no sistema
- Documento não pode estar quitado
- Valor do documento deve ser compatível com o valor do retorno
- Código de ocorrência deve existir nos parâmetros
- Data de pagamento deve estar presente quando informada pelo banco

### Regras Financeiras
- Valores monetários são convertidos dividindo por 100
- Juros e descontos são armazenados em campos JSON
- Documento com valor **0,01** ignora validação de divergência de valor

### Ocorrências Bancárias Tratadas

| Código | Descrição |
|------|----------|
| 02 | Entrada confirmada |
| 03 | Entrada rejeitada |
| 04 | Alteração de dados |
| 06 | Liquidação normal |
| 09 | Baixa simples |
| 29 | Tarifa manutenção boletos vencidos |

## 🎨 Inconsistências Registradas

| Situação | Descrição |
|--------|-----------|
| Documento não encontrado | ID inexistente ou inválido no retorno |
| Documento quitado | Título já recebido anteriormente |
| Divergência de valor | Valor do retorno diferente do valor do documento |
| Ocorrência inválida | Código não cadastrado |
| Sem data de pagamento | Data não informada no retorno |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Critérios e consultas ORM
- `multitec.utils` – Leitura de arquivos e utilitários
- `java.time` – Manipulação de datas

**Serviços:**
- `SCFService` – Serviços de cobrança

## 📝 Observações Técnicas

- Layout específico para **CNAB 400 – Banco Itaú**
- Processa apenas registros de detalhe (tipo `1`)
- Documento pode ser localizado por ID principal ou campo customizado
- Estrutura tolerante a erros, com registro detalhado de inconsistências
- JSON utilizado para armazenar juros e descontos de liquidação
- Retorno preparado para exibição e processamento posterior
- Não realiza baixa automática, apenas valida e prepara dados
