# SCV_Pre_Gravacao.md

## 📖 Descrição
Sistema de validação pré-gravação para documentos do SCV (Sistema de Controle de Vendas) da Linhasita, realizando verificações de crédito e consistência antes da persistência dos dados.

## 🎯 Finalidade
Garantir a integridade e conformidade dos documentos de venda através de validações financeiras e comerciais antes da gravação definitiva no sistema.

## 👥 Público-Alvo
- Departamento Comercial
- Crédito e Cobrança
- Faturamento
- Controladoria

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SCV_Pre_Gravacao` - Validação pré-gravação

**Localização:** `strema/formulas/scv/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `ABE30` - Condições de pagamento
- `ABE01` - Entidades/Clientes
- `DAA01` - Títulos a receber
- `EAA0107` - Mensagens de inconsistência

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Abe30` - Condições de pagamento
- `Abe01` - Entidades
- `Daa01` - Títulos
- `Eaa0107` - Inconsistências

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento a ser validado |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| gravar | Indicador de gravação (0-Não, 1-Sim) | Integer |
| eaa01 | Documento com inconsistências registradas | Eaa01 |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Obtém documento fiscal (EAA01)
   - Verifica existência de condição de pagamento
   - Inicializa flags de controle

2. **Processamento por Tipo de Operação**
   - Aplica validações apenas para operação "201" (Venda)
   - Executa verificações financeiras específicas

3. **Validações de Inconsistência**
   - Títulos a receber vencidos
   - Limite de crédito excedido
   - Restrições financeiras da entidade

4. **Definição de Bloqueio**
   - Define se documento será bloqueado
   - Registra mensagens de inconsistência
   - Remove política de uso do documento

## ⚠️ Regras de Negócio

### Validações Obrigatórias
- Condição de pagamento é obrigatória
- Apenas operação "201" (Venda) sofre validações financeiras
- Documento deve ter classificação de documento ativa

### Verificações Financeiras

**Títulos Vencidos:**
- Soma valores de títulos com vencimento anterior à data atual
- Bloqueia documento se existirem títulos vencidos

**Limite de Crédito:**
- Calcula total financeiro em aberto
- Soma títulos a receber não quitados
- Considera valor do documento atual (se novo)
- Bloqueia se exceder limite cadastrado

**Restrições:**
- Verifica flag de restrição financeira no cadastro da entidade
- Bloqueia documento se entidade possui restrição

### Comportamento do Sistema
- Documento é bloqueado (`eaa01bloqueado = 1`) se houver inconsistências
- Mensagens detalhadas são registradas em `EAA0107`
- Política de uso é removida do documento
- Processamento continua mesmo com inconsistências

## 🎨 Inconsistências Registradas

| Tipo | Mensagem | Identificador |
|------|----------|---------------|
| Títulos Vencidos | "Título a receber vencido: [valor]" | BLOQUEIO |
| Limite Excedido | "Limite de crédito excedido..." | BLOQUEIO |
| Sem Limite | "A entidade não possui limite de crédito" | BLOQUEIO |
| Restrição | "A entidade possui restrição financeira" | BLOQUEIO |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `multitec.utils` - Utilitários e coleções

**Consultas:**
- Soma de títulos vencidos por entidade
- Total financeiro de documentos em aberto
- Total de títulos a receber não quitados

## 📝 Observações Técnicas

- Validações aplicadas apenas para novos documentos/vendas
- Sistema tolerante a falhas (continua processamento)
- Mensagens de erro detalhadas e específicas
- Integração com sistema de bloqueio de documentos
- Remoção automática de política de uso
- Consultas otimizadas com joins
- Suporte a campos JSON para configurações flexíveis