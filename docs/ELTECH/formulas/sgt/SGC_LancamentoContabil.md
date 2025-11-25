# SGC_LancamentoContabil.md

## 📖 Descrição
Sistema de formatação automática de histórico contábil para o SGC (Sistema de Gestão Contábil), processando coringas dinâmicos para compor o histórico dos lançamentos contábeis.

## 🎯 Finalidade
Automatizar a construção do histórico contábil através da substituição de coringas por informações dinâmicas do documento fiscal, entidade, classificação patrimonial e eventos.

## 👥 Público-Alvo
- Departamento Contábil
- Financeiro
- Faturamento
- Controladoria

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SGC_LancamentoContabil` - Formatação de histórico contábil

**Localização:** `eltech/formulas/sgc/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EBB05` - Lançamentos contábeis
- `AAH01` - Tipos de documento
- `ABE01` - Entidades/Clientes
- `ABB01` - Documentos fiscais
- `ABH21` - Eventos contábeis
- `ECA01` - Classificação patrimonial
- `EBA02` - Planilha de lançamento programado

**Entidades Envolvidas:**
- `Ebb05` - Lançamento contábil
- `Aah01` - Tipo de documento
- `Abe01` - Entidade
- `Abb01` - Documento fiscal
- `Abh21` - Evento
- `Eca01` - Classificação patrimonial
- `Eba02` - Lançamento programado

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| ebb05 | Ebb05 | Sim | Lançamento contábil a ser processado |
| abh21 | Abh21 | Não | Evento contábil (alternativo) |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| ebb05historico | Histórico contábil formatado | String |

## 🔄 Fluxo do Processo

1. **Inicialização**
   - Obtém lançamento contábil (EBB05)
   - Verifica existência de histórico
   - Inicializa entidades relacionadas

2. **Busca de Dados Relacionados**
   - Tipo de documento (AAH01) através do documento central
   - Entidade (ABE01) do documento
   - Classificação patrimonial (ECA01) quando aplicável
   - Planilha de lançamento programado (EBA02)
   - Evento contábil (ABH21)

3. **Processamento de Coringas**
   - Substitui coringas por valores dinâmicos
   - Aplica formatação específica para cada tipo de dado
   - Mantém valores vazios para dados não encontrados

4. **Atualização do Histórico**
   - Atribui histórico formatado ao lançamento
   - Preserva estrutura original do histórico

## ⚠️ Regras de Negócio

### Coringas Suportados

| Coringa | Descrição | Fonte |
|---------|-----------|-------|
| $1 | Nome abreviado do documento | AAH01.aah01na |
| $2 | Número do documento | ABB01.abb01num |
| $3 | Série do documento | ABB01.abb01serie |
| $4 | Data do documento (dd/MM/yyyy) | ABB01.abb01data |
| $5 | Nome da entidade | ABE01.abe01nome |
| $6 | Parcela do documento | ABB01.abb01parcela |
| $7 | Nome da classificação patrimonial | ECA01.eca01nome |
| $8 | Observação SGC (CampoLivre) | EBA02.eba02json.obs_sgc |
| $9 | Nome do evento | ABH21.abh21nome |

### Validações
- Histórico é obrigatório para processamento
- Dados nulos são substituídos por string vazia
- Formatação de data padrão brasileiro (dd/MM/yyyy)
- Consultas aplicam where padrão do sistema para segurança

## 🎨 Comportamento do Sistema

- **Processamento Condicional**: Apenas executa se histórico existir
- **Tolerância a Nulos**: Substitui dados não encontrados por string vazia
- **Busca Otimizada**: Consultas específicas apenas para dados necessários
- **Formatação Consistente**: Datas no formato brasileiro

## 🔧 Dependências

**Bibliotecas:**
- `sam.dicdados` - Definição de tipos de fórmula
- `sam.model.entities` - Entidades do sistema
- `sam.server.samdev.formula` - Framework de fórmulas

**Consultas:**
- Busca de tipo de documento por ID
- Busca de entidade com where padrão
- Busca de classificação patrimonial
- Busca de lançamento programado por documento central
- Busca de evento contábil

## 📝 Observações Técnicas

- **Tipo de Fórmula**: `LCTO_SGC` - Específico para lançamentos contábeis do SGC
- **Tratamento de Nulos**: Robustez contra dados incompletos
- **Performance**: Consultas diretas apenas quando necessário
- **Flexibilidade**: Suporte a coringas dinâmicos e expansível
- **Segurança**: Aplicação de where padrão em consultas de entidade

## 🔄 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de formatação do histórico.

### `obterTipoFormula()`
Define o tipo específico da fórmula para o sistema SGC.

## 💡 Estrutura do Histórico
O sistema permite templates flexíveis como:
- "Fatura \$1 nº \$2 série \$3 - \$5"
- "Classificação: \$7 - Evento: \$9"
- "Parcela \$6 - Obs: \$8"

Sendo automaticamente expandido para:
"Fatura NF nº 12345 série 1 - Cliente XYZ Ltda"