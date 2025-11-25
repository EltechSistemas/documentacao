# CAS_ImportarFCIEab01011.md

## 📖 Descrição
Sistema de importação de dados FCI (Ficha de Conteúdo de Importação) para a tabela EAB01011 no módulo CAS (Controle Aduaneiro e Sped), processando arquivos Excel com informações de cálculos aduaneiros.

## 🎯 Finalidade
Automatizar a importação em massa de dados de cálculos aduaneiros a partir de planilhas Excel, inserindo os registros diretamente na tabela de itens de cálculo FCI.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria Aduaneira
- Gestão de Importação

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `CAS_ImportarFCIEab01011` - Importação de dados FCI

**Localização:** `strema/formulas/cas/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAB01011` - Itens de cálculo FCI
- `EAB0101` - Cabeçalho de cálculo FCI

**Entidades Envolvidas:**
- `Eab0101` - Cabeçalho do cálculo FCI
- `Eab01011` - Itens do cálculo FCI

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| arquivo | MultipartFile | Sim | Arquivo Excel (.xlsx) com dados para importação |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| (nenhuma saída explícita) | Registros inseridos na tabela EAB01011 | Operação de banco |

## 🔄 Fluxo do Processo

1. **Leitura do Arquivo**
   - Obtém arquivo Excel do parâmetro
   - Cria workbook a partir do InputStream
   - Seleciona primeira planilha (sheet) para processamento

2. **Processamento Linha por Linha**
   - Itera sobre todas as linhas da planilha
   - Ignora cabeçalho (linha 0)
   - Processa apenas linhas com ID válido na primeira coluna

3. **Validação de Dados**
   - Verifica existência do registro EAB0101 correspondente
   - Converte valores das células para tipos apropriados
   - Valida dados numéricos obrigatórios

4. **Construção do SQL Dinâmico**
   - Monta instrução INSERT otimizada com múltiplos VALUES
   - Utiliza placeholders (?) para prevenção de SQL injection
   - Agrupa todos os registros em uma única execução

5. **Execução em Lote**
   - Prepara statement com parâmetros
   - Executa inserção em massa
   - Utiliza sequence para geração de IDs

## ⚠️ Regras de Negócio

### Estrutura do Arquivo Excel

| Coluna | Tipo | Descrição | Obrigatório |
|--------|------|-----------|-------------|
| 0 | Long | ID do cálculo FCI (EAB0101) | Sim |
| 1 | Long | ID do item | Sim |
| 2 | BigDecimal | Quantidade de composição | Sim |
| 3 | BigDecimal | Quantidade | Sim |
| 4 | BigDecimal | Valor | Sim |
| 5 | BigDecimal | Valor por unidade de medida | Sim |

### Validações
- Registro EAB0101 deve existir no banco
- Células numéricas não podem estar vazias
- Apenas processa linhas com ID na primeira coluna
- Ignora linhas completamente vazias

### Estrutura do INSERT
```sql
INSERT INTO eab01011 
(eab01011id, eab01011calc, eab01011item, eab01011qtdComp, eab01011qtde, eab01011valor, eab01011vmu)
VALUES
(nextval('default_sequence'), ?, ?, ?, ?, ?, ?),
(nextval('default_sequence'), ?, ?, ?, ?, ?, ?),
...