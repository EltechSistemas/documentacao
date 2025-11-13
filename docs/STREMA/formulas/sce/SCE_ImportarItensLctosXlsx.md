# SCE_ImportarItensLctosXlsx.md

## 📖 Descrição
Sistema de importação de itens para lançamentos do SCE via arquivo Excel (XLSX) para a Linhasita, permitindo carga em lote de itens com suas quantidades e valores.

## 🎯 Finalidade
Automatizar o processo de importação de itens para lançamentos contábeis através de arquivos Excel, validando e processando os dados antes da inserção no sistema.

## 👥 Público-Alvo
- Departamento Contábil
- Almoxarifado/Estoque
- Financeiro
- Controladoria

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SCE_ImportarItensLctosXlsx` - Processamento de importação Excel

**Localização:** `strema/formulas/sce/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `ABM01` - Cadastro de itens
- `ABE4001` - Preços de itens
- `ABA20` - Repositório de dados (avisos)

**Entidades Envolvidas:**
- `Abm01` - Itens
- `Abe4001` - Preços
- `ItemSCEDto` - DTO para transferência de itens

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| arquivo | MultipartFile | Sim | Arquivo Excel com dados dos itens |

## 📋 Estrutura do Arquivo Excel

| Coluna | Descrição | Tipo | Obrigatório |
|--------|-----------|------|-------------|
| 0 | Tipo do item | Integer | Sim |
| 1 | Código do item | String | Sim |
| 4 | Quantidade | BigDecimal | Não |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Recebe arquivo Excel multipart
   - Abre a primeira planilha do workbook

2. **Processamento Linha a Linha**
   - Ignora cabeçalho (linha 0)
   - Processa a partir da linha 1
   - Valida células obrigatórias

3. **Busca e Validação de Itens**
   - Localiza item por tipo e código
   - Verifica existência no cadastro
   - Confirma configuração de estoque

4. **Obtenção de Valores**
   - Recupera quantidade da planilha
   - Busca preço unitário do item
   - Prepara DTO para transferência

5. **Tratamento de Erros**
   - Coleta mensagens de erro
   - Registra itens não encontrados
   - Mantém processamento mesmo com falhas

## ⚠️ Regras de Negócio

### Validações
- Item deve existir no cadastro (ABM01)
- Item deve ter configuração de estoque
- Células de tipo e código são obrigatórias
- Quantidade opcional (default 0)

### Processamento
- Apenas primeira planilha é processada
- Linha vazia na coluna 0 encerra processamento
- Continua processamento mesmo com erros parciais
- Coleta todas as mensagens de erro

### Dados Obtidos
- **Quantidade:** Da planilha Excel
- **Valor Unitário:** Do cadastro de preços (ABE4001)
- **Lote/Série:** Sempre nulos para importação

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| itensLctos | Lista de itens processados | List<ItemSCEDto> |

## 🔧 Dependências

**Bibliotecas:**
- `Apache POI` - Manipulação de arquivos Excel
- `multiorm` - Persistência e consultas
- `spring-web` - Upload de arquivos multipart

**Estruturas de Dados:**
- `ItemSCEDto` - Transporte de dados dos itens
- `TableMap` - Armazenamento de dados JSON

## 📝 Observações Técnicas

- Suporte a arquivos XLSX (Excel moderno)
- Processamento assíncrono e tolerante a falhas
- Validação em tempo real durante importação
- Mensagens de erro detalhadas por item
- Integração com DTO padrão do SCE
- Busca otimizada de itens com joins
- Tratamento de transações para persistência