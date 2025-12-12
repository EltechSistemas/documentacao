SGT_Apuracao_EFD_IPI# SGT_Apuracao_EFD_IPI - Apuração de IPI para EFD

## 📖 Descrição
Fórmula para cálculo e apuração do Imposto sobre Produtos Industrializados (IPI) para geração da Escrituração Fiscal Digital (EFD), realizando a consolidação de débitos, créditos e saldos do período fiscal.

## 🎯 Finalidade
Calcular automaticamente os valores de IPI a pagar ou compensar em um período fiscal, consolidando operações de entrada e saída, ajustes manuais e saldos anteriores para geração da EFD-IPI.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento de Tributos
- Indústrias e empresas sujeitas ao IPI

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Edb0105` - Ajustes manuais do IPI
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Aaj19` - Tipos de ajustes do IPI
- `Aaj15` - CFOP (Código Fiscal de Operações e Prestações)
- `Aah01` - Modelos de documentos fiscais
- `Aaj03` - Situações dos documentos
- `Aaj28` - Tipos de apuração

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Objeto de apuração fiscal contendo mês, ano e tipo |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção do objeto de apuração (Edb01)
- Definição do período fiscal (mês/ano)
- Inicialização da estrutura JSON para armazenamento dos cálculos
- Seleção do alinhamento para campos dinâmicos
- Identificação do campo JSON para valores de IPI

### 2. **Cálculo de Saldo Anterior**
- **01 - Saldo credor anterior**: Busca apuração do período anterior
  - Consulta apurações do mesmo tipo em meses anteriores
  - Extrai saldo credor do JSON da apuração anterior
  - Se não houver apuração anterior, considera zero

### 3. **Cálculo de Débitos do IPI**
- **02 - Saídas com débito do imposto**: Consolida valores de IPI nas saídas
  - Processa documentos fiscais com CFOP iniciados em 5 ou 6
  - Considera modelos de documentos: 01, 1B, 04, 55
  - Filtra por situações válidas (exclui 02, 03, 04, 05)
  - Considera ambos: entradas (esData) e saídas (abb01data)
  - Soma valores do campo IPI no JSON dos itens

### 4. **Cálculo de Créditos do IPI**
- **03 - Entradas com crédito do imposto**: Consolida valores de IPI nas entradas
  - Processa documentos fiscais com CFOP iniciados em 1, 2 ou 3
  - Considera os mesmos modelos de documentos que os débitos
  - Aplica os mesmos filtros de situações
  - Soma valores do campo IPI no JSON dos itens

### 5. **Ajustes Manuais**
- **04 - Outros débitos e estorno de créditos**: Ajustes manuais de débito
  - Processa ajustes manuais registrados em Edb0105
  - Filtra por natureza do ajuste = 0 (débito)
  - Inclui estornos de créditos anteriores

- **05 - Outros créditos e estorno de débitos**: Ajustes manuais de crédito
  - Processa ajustes manuais registrados em Edb0105
  - Filtra por natureza do ajuste = 1 (crédito)
  - Inclui estornos de débitos anteriores

### 6. **Cálculo de Saldos Finais**
- **Saldo líquido**: Calcula saldo consolidado
  - Fórmula: Saldo = credAnt + credEntradas - deboutros + credoutros - debSaidas

- **Saldo devedor**: Valor a pagar
  - Se saldo líquido for negativo, transforma em positivo
  - Representa o valor devido ao fisco

- **Saldo credor**: Valor a compensar
  - Se saldo líquido for positivo ou zero
  - Representa crédito para períodos futuros

## ⚠️ Regras de Negócio

### Filtros de Documentos
- **Situações excluídas**: 02, 03, 04, 05 (conforme aaj03efd)
- **Indicador EFD-ICMS**: Apenas documentos com `eaa01iEfdIcms = 1`
- **Modelos válidos**: 01, 1B, 04, 55 (documentos de produtos industriais)
- **Período considerado**: 
  - Entradas: data de entrada (esData)
  - Saídas: data do documento (abb01data)

### Classificação por CFOP
- **Débitos (CFOP 5xxx, 6xxx)**:
  - 5xxx: Saídas ou prestações
  - 6xxx: Outras saídas ou prestações

- **Créditos (CFOP 1xxx, 2xxx, 3xxx)**:
  - 1xxx: Entradas ou aquisições
  - 2xxx: Devoluções
  - 3xxx: Serviços

### Ajustes Manuais (Edb0105)
- **Natureza 0**: Débitos e estornos de créditos
- **Natureza 1**: Créditos e estornos de débitos
- Vinculados à apuração específica
- Permitem correções manuais dos cálculos automáticos

### Busca de Apuração Anterior
- Considera mesmo tipo de apuração (aaj28id)
- Busca pelo grupo centralizador da empresa
- Ordena por ano e mês decrescente
- Pega apenas o registro mais recente anterior ao período atual

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração do IPI.

### `buscarApuracaoAnterior()`
Busca a apuração do período anterior para cálculo de saldo credor.
- Parâmetros: mês, ano, tipo de apuração
- Retorna: Objeto Edb01 da apuração anterior ou null

### `buscarDebitosIPI()`
Busca valores de IPI a débito nos documentos fiscais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: CFOP 5xxx/6xxx, modelos específicos, situações válidas
- Retorna: Soma dos valores de débito

### `buscarCreditosIPI()`
Busca valores de IPI a crédito nos documentos fiscais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: CFOP 1xxx/2xxx/3xxx, modelos específicos, situações válidas
- Retorna: Soma dos valores de crédito

## 📊 Estrutura de Saída

**JSON da Apuração (edb01json):**
- `credAnt` - Saldo credor do período anterior
- `debSaidas` - Saídas com débito do imposto
- `credEntradas` - Entradas com crédito do imposto
- `deboutros` - Outros débitos e estorno de créditos
- `credoutros` - Outros créditos e estorno de débitos
- `saldoDevedor` - Saldo devedor (valor a pagar)
- `saldoCredor` - Saldo credor (crédito para período seguinte)
