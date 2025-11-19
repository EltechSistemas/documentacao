# SCF_LayoutBancoItauRetorno_CNAB_400 - Processamento de Retorno Bancário Itaú

## 📖 Descrição
Classe responsável pelo processamento de arquivos de retorno bancário no layout CNAB 400 do Banco Itaú, realizando a conciliação automática de documentos financeiros com as ocorrências informadas pelo banco.

## 🎯 Finalidade
Automatizar o processo de conciliação bancária, validando e processando retornos de cobrança, identificando inconsistências e aplicando liquidações automáticas nos documentos do sistema.

## 👥 Público-Alvo
- Departamento Financeiro
- Tesouraria
- Contabilidade
- Backoffice bancário

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| registros | TextFileLeitura | Sim | Arquivo de retorno bancário CNAB 400 |

## 📋 Estrutura do Arquivo CNAB 400

### Layout do Registro Detalhe (Tipo 1):
| Posição | Tamanho | Descrição |
|---------|---------|-----------|
| 0-1 | 1 | Tipo registro (1) |
| 37-62 | 25 | Identificação do documento |
| 108-110 | 2 | Código da ocorrência |
| 152-165 | 13 | Valor do documento (com 2 decimais) |
| 240-253 | 13 | Valor do desconto |
| 253-266 | 13 | Valor líquido |
| 266-279 | 13 | Valor da multa/juros |
| 295-301 | 6 | Data pagamento (DDMMAA) |

## 🔄 Fluxo do Processo

### 1. **Leitura do Arquivo**
- Ignora header (primeira linha)
- Processa apenas registros do tipo 1 (detalhe)
- Itera linha por linha do arquivo

### 2. **Validação do Documento**
- Extrai ID do documento (posições 37-62)
- Remove zeros à esquerda do ID
- Busca documento no sistema por ID ou campo customizado

### 3. **Validações de Negócio**
- Verifica se documento já foi quitado
- Compara valores do documento com retorno bancário
- Valida código de ocorrência existente
- Verifica presença de data de pagamento

### 4. **Processamento de Ocorrências**
- Identifica tipo de ocorrência (02, 03, 04, 06, 09, 29)
- Mapeia para PLF (Parametro e Lançamento Fiscal) correspondente
- Aplica liquidação para ocorrência "06" (Liquidação normal)

### 5. **Atualização de Dados**
- Define data de pagamento
- Calcula valor líquido
- Registra juros e descontos no JSON do documento

## ⚠️ Regras de Negócio

### Validações de Documento:
- **Documento não encontrado**: Gera inconsistência se ID não existir
- **Documento já quitado**: Impede processamento duplicado
- **Valor divergente**: Compara valor do documento com retorno (exceto para R$ 0,01)
- **Ocorrência inválida**: Verifica se código existe no mapeamento

### Códigos de Ocorrência Suportados:
| Código | Descrição | Ação |
|--------|-----------|------|
| 02 | Entrada confirmada | Informação |
| 03 | Entrada rejeitada | Informação |
| 04 | Alteração de Dados | Informação |
| 06 | Liquidação normal | **Aplica baixa** |
| 09 | Baixa Simples | Informação |
| 29 | Tarifa manutenção Boletos Vencidos | Informação |

### Mapeamento PLF:
- **06** → Código PLF "201" (Liquidação normal)

### Tratamento de Valores:
- Valores monetários divididos por 100 (2 casas decimais)
- Data no formato DDMMAA convertida para LocalDate
- Campos numéricos com zeros são considerados vazios

## 🎨 Saídas Geradas

| Campo | Tipo | Descrição |
|-------|------|-----------|
| tmList | List<TableMap> | Lista de documentos processados |
| daa01 | Daa01 | Documento financeiro |
| abf20id | Long | ID do PLF para liquidação |
| ocorrencia | String | Descrição da ocorrência |
| inconsistencias | List<String> | Lista de erros encontrados |

## 🔧 Dependências

### Bibliotecas:
- `TextFileLeitura` - Leitura de arquivos texto
- `SCFService` - Serviço do módulo financeiro
- `MultiORM` - Acesso a dados

### Entidades:
- `Daa01` - Documentos financeiros
- `Abf20` - PLF (Parametro e Lançamento Fiscal)

### Configurações:
- Mapeamento fixo de ocorrências bancárias
- Códigos PLF pré-definidos

## 📝 Observações Técnicas

### Busca de Documentos:
- Primeiro busca por ID direto (daa01id)
- Fallback para campo customizado (id_sam3)
- Join com central de documentos para dados complementares

### Tratamento de Erros:
- Continua processamento mesmo com documentos inválidos
- Coleta todas as inconsistências por documento
- Retorna lista completa para decisão manual

### Performance:
- Processamento linha a linha do arquivo
- Consultas otimizadas com criteria
- Uso de TableMap para dados temporários

### Segurança:
- Validação rigorosa de dados antes do processamento
- Prevenção de dupla liquidação
- Verificação de consistência de valores