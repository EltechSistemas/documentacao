# SGT_Apuracao_LF_ManipulacaoCampos - Manipulação de Campos da Apuração do Livro Fiscal

## 📖 Descrição
Fórmula complementar para cálculo e manipulação dos campos derivados da apuração fiscal do Livro Fiscal (LF), realizando os cálculos finais de subtotais, totais, saldos e impostos a recolher com base nos valores já apurados.

## 🎯 Finalidade
Processar os valores já calculados da apuração fiscal para gerar os campos derivados necessários para o Livro Fiscal, incluindo subtotais, totais, saldos devedor/credor e cálculo final do imposto a recolher após deduções.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Sistema de Livros Fiscais
- Módulo de Apurações Fiscais

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `edb01json` - Estrutura JSON com os valores apurados

**Campos de Entrada (já calculados):**
- `debImp` - Débitos do imposto
- `outrosDeb` - Outros débitos
- `estCred` - Estorno de créditos
- `credImp` - Créditos do imposto
- `outrosCred` - Outros créditos
- `estDeb` - Estorno de débitos
- `credAnt` - Saldo credor anterior
- `deducoes` - Deduções aplicáveis

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Objeto de apuração fiscal com JSON já preenchido |

## 🔄 Fluxo do Processo

### 1. **Validação Inicial**
- Verifica se o objeto Edb01 existe
- Verifica se o JSON da apuração existe
- Retorna silenciosamente se faltarem dados essenciais
- Seleciona alinhamento específico (0040) para campos dinâmicos

### 2. **Cálculo de Subtotal de Saídas**
- **Fórmula**: `subTotSai = debImp + outrosDeb + estCred`
- **Componentes**:
  - `debImp`: Débitos principais do imposto
  - `outrosDeb`: Outros débitos diversos
  - `estCred`: Estornos de créditos (aumentam o débito)
- **Propósito**: Consolidar todas as obrigações de débito do período

### 3. **Cálculo de Subtotal de Entradas**
- **Fórmula**: `subTotEnt = credImp + outrosCred + estDeb`
- **Componentes**:
  - `credImp`: Créditos principais do imposto
  - `outrosCred`: Outros créditos diversos
  - `estDeb`: Estornos de débitos (aumentam o crédito)
- **Propósito**: Consolidar todos os direitos de crédito do período

### 4. **Cálculo do Total**
- **Fórmula**: `total = subTotEnt + credAnt`
- **Componentes**:
  - `subTotEnt`: Subtotal de entradas do período atual
  - `credAnt`: Saldo credor acumulado de períodos anteriores
- **Propósito**: Calcular o total de créditos disponíveis (atuais + anteriores)

### 5. **Cálculo do Saldo Devedor**
- **Condição**: Apenas se `subTotalSaidas > subTotalEntradas`
- **Fórmula**: `saldoDevedor = subTotalSaidas - total`
- **Lógica**: 
  - Se saídas forem maiores que entradas: há débito líquido
  - Se entradas forem maiores ou iguais: saldo devedor = 0
- **Propósito**: Determinar se há saldo devedor após compensação

### 6. **Cálculo do Imposto a Recolher**
- **Fórmula**: `impRecolher = saldoDevedor - deducoes`
- **Componentes**:
  - `saldoDevedor`: Débito líquido após compensação
  - `deducoes`: Deduções legalmente permitidas
- **Propósito**: Calcular o valor final a pagar ao fisco

### 7. **Cálculo do Saldo Credor**
- **Condição**: Apenas se `subTotalSaidas <= total`
- **Fórmula**: `saldoCredor = total - subTotalSaidas`
- **Lógica**:
  - Se créditos totais forem maiores que débitos totais: há crédito excedente
  - Se débitos forem maiores: saldo credor = 0
- **Propósito**: Determinar crédito a compensar em períodos futuros

## ⚠️ Regras de Negócio

### Lógica Condicional dos Saldos
- **Saldo Devedor**: Só existe se débitos > créditos
Se subTotalSaidas > subTotalEntradas:
saldoDevedor = subTotalSaidas - total
Senão:
saldoDevedor = 0

- **Saldo Credor**: Só existe se créditos ≥ débitos
Se subTotalSaidas <= total:
saldoCredor = total - subTotalSaidas
Senão:
saldoCredor = 0


### Relacionamento entre Campos
- **Exclusividade**: Apenas um dos saldos (devedor ou credor) pode ser positivo
- **Compensação completa**: Saldo anterior é totalmente compensado com débitos atuais
- **Deduções aplicáveis**: Apenas reduzem o imposto a recolher, não afetam saldo credor

### Tratamento de Valores
- **Valores zero**: Uso de `getBigDecimal_Zero()` para segurança
- **Campos dinâmicos**: Uso de `getCampo()` para flexibilidade
- **Validação silenciosa**: Retorno antecipado se dados estiverem incompletos

### Ordem dos Cálculos
1. Subtotais (saídas e entradas)
2. Total (entradas + saldo anterior)
3. Saldo devedor (se aplicável)
4. Imposto a recolher (saldo devedor - deduções)
5. Saldo credor (se aplicável)

## 🔧 Métodos Principais

### `executar()`
Método principal que executa toda a sequência de cálculos derivados.
- Valida presença dos dados necessários
- Executa cálculos em ordem lógica
- Atualiza o JSON da apuração

### Cálculos Implementados:
1. **Subtotal de Saídas**: Consolidação de todos os débitos
2. **Subtotal de Entradas**: Consolidação de todos os créditos
3. **Total**: Créditos totais disponíveis
4. **Saldo Devedor**: Débito líquido após compensação
5. **Imposto a Recolher**: Valor final a pagar
6. **Saldo Credor**: Crédito excedente para períodos futuros

## 📊 Estrutura de Saída

**Campos de Entrada (pré-calculados):**
- `debImp` - Débitos do imposto
- `outrosDeb` - Outros débitos
- `estCred` - Estorno de créditos
- `credImp` - Créditos do imposto
- `outrosCred` - Outros créditos
- `estDeb` - Estorno de débitos
- `credAnt` - Saldo credor anterior
- `deducoes` - Deduções

**Campos Calculados (saída):**
- `subTotSai` - Subtotal de saídas
subTotSai = debImp + outrosDeb + estCred
- `subTotEnt` - Subtotal de entradas
subTotEnt = credImp + outrosCred + estDeb
- `total` - Total de créditos disponíveis
total = subTotEnt + credAnt
- `saldoDevedor` - Saldo devedor (se débitos > créditos)
Se subTotSai > subTotEnt:
saldoDevedor = subTotSai - total
Senão: 0
- `impRecolher` - Imposto a recolher (após deduções)
impRecolher = saldoDevedor - deducoes
- `saldoCredor` - Saldo credor (se créditos ≥ débitos)
Se subTotSai <= total:
saldoCredor = total - subTotSai
Senão: 0

## 🔧 Dependências

**Bibliotecas:**
- `br.com.multitec.utils.collections` - TableMap para manipulação de JSON
- `sam.dicdados` - Tipos de fórmula
- `sam.server.samdev.formula` - Base para fórmulas

**Pré-requisitos:**
- Apuração principal já calculada (Edb01 com JSON preenchido)
- Campos básicos já definidos no JSON
- Alinhamento 0040 configurado

**Módulos:**
- Módulo de Apurações Fiscais
- Módulo de Livros Fiscais
- Módulo de Cálculos Derivados

## 📝 Observações Técnicas

### Natureza Complementar
- **Fórmula auxiliar**: Não calcula valores primários, apenas derivados
- **Dependência de dados**: Requer apuração principal já processada
- **Execução sequencial**: Geralmente executada após fórmulas de apuração

### Tratamento de Erros
- **Validação defensiva**: Verifica nulls antes de processar
- **Retorno silencioso**: Não lança exceções para dados faltantes
- **Valores padrão**: Usa zero para valores não existentes

### Performance
- **Cálculos simples**: Operações aritméticas básicas
- **Sem consultas ao banco**: Trabalha apenas com dados em memória
- **Execução rápida**: Processamento direto dos valores

### Flexibilidade
- **Campos dinâmicos**: Usa `getCampo()` para adaptação a diferentes configurações
- **Alinhamento específico**: Configurável para diferentes tipos de apuração
- **JSON como interface**: Estrutura flexível para diferentes tributos

### Segurança
- **`getBigDecimal_Zero()`**: Garante valores numéricos válidos
- **Verificação de null**: Previne NullPointerException
- **Cálculos determinísticos**: Sempre produz mesmo resultado para mesmos inputs

### Integração
- **Encadeamento**: Projetada para execução em pipeline com outras fórmulas
- **Manutenção de estado**: Atualiza o objeto Edb01 original
- **Interface consistente**: Mesmo padrão de outras fórmulas de apuração

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Manipulação de Campos  
**Versão:** 1.0  
**Finalidade:** Cálculos derivados da apuração fiscal  
**Destino:** Livro Fiscal (LF)  
**Dependência:** Requer apuração principal já calculada