# SGT_Apuracao_LF_IPI

## 📖 Descrição
Fórmula para cálculo e apuração do Imposto sobre Produtos Industrializados (IPI) para geração do Livro Fiscal (LF), realizando a consolidação de débitos, créditos e saldos do período com foco nas obrigações do Livro Fiscal para indústrias.

## 🎯 Finalidade
Calcular automaticamente os valores de IPI para emissão do Livro Fiscal, consolidando operações de entrada e saída industriais, e calculando saldos compensáveis entre períodos, com tratamento específico para os registros exigidos pela legislação do Livro Fiscal para produtos industrializados.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade Industrial
- Faturamento
- Departamento de Tributos
- Setor de Livros Fiscais de indústrias

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Aaj15` - CFOP (Código Fiscal de Operações e Prestações)
- `Aaj28` - Tipos de apuração

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Objeto de apuração fiscal contendo mês, ano e tipo |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção do objeto de apuração (Edb01)
- Inicialização da estrutura JSON para armazenamento dos cálculos
- Seleção do alinhamento específico (0040) para campos dinâmicos
- Definição do período fiscal (mês/ano)
- Inicialização de todos os campos do JSON com valor zero

### 2. **Inicialização de Campos**
Todos os campos do JSON são inicializados com valor 0 para garantir estrutura consistente:
- `debImp` - Débitos do imposto
- `outrosDeb` - Outros débitos
- `estCred` - Estorno de créditos
- `credImp` - Créditos do imposto
- `outrosCred` - Outros créditos
- `estDeb` - Estorno de débitos
- `credAnt` - Saldo credor anterior
- `deducoes` - Deduções
- `subTotSai` - Subtotal saídas
- `subTotEnt` - Subtotal entradas
- `total` - Total
- `sdoDevedor` - Saldo devedor
- `sdoCredor` - Saldo credor
- `impRecolher` - Imposto a recolher

### 3. **Cálculo de Débitos do IPI**
- **Valor com débito do imposto**: Saídas industriais tributadas
  - Processa documentos de saída (esMov = 1)
  - CFOP iniciados em 5, 6 ou 7 (saídas industriais)
  - Documentos não cancelados
  - Período: data de entrada (esData) entre data inicial e final
  - Soma valores do campo IPI no JSON dos itens usando campos dinâmicos

### 4. **Cálculo de Créditos do IPI**
- **Valor com crédito do imposto**: Entradas industriais com crédito
  - Processa documentos de entrada (esMov = 0)
  - CFOP iniciados em 1, 2 ou 3 (entradas industriais)
  - Documentos não cancelados
  - Período: data de entrada (esData) entre data inicial e final
  - Soma valores do campo IPI no JSON dos itens usando campos dinâmicos

### 5. **Cálculo de Saldo Anterior**
- **Saldo credor anterior**: Calcula saldo compensável do período anterior
  - Busca apuração do período anterior do mesmo tipo
  - Se existir apuração anterior:
    - Calcula créditos totais do período anterior:
      ```
      Créditos = credImp + outrosCred + estDeb + credAnt
      ```
    - Calcula débitos totais do período anterior:
      ```
      Débitos = debImp + outrosDeb + estCred
      ```
    - Calcula saldo credor líquido:
      ```
      SaldoCredor = Créditos - Débitos
      ```
    - Se saldo credor for positivo, armazena no campo `CredAnt`
    - Se negativo ou zero, mantém 0

## ⚠️ Regras de Negócio

### Classificação por CFOP
- **Débitos (CFOP 5xxx, 6xxx, 7xxx)**:
  - 5xxx: Saídas ou prestações
  - 6xxx: Outras saídas ou prestações
  - 7xxx: Saídas industriais específicas

- **Créditos (CFOP 1xxx, 2xxx, 3xxx)**:
  - 1xxx: Entradas ou aquisições
  - 2xxx: Devoluções
  - 3xxx: Serviços

### Filtros de Documentos
- **Cancelados excluídos**: Apenas documentos com `eaa01cancData IS NULL`
- **Data de referência**: Usa data de entrada (esData) para ambos débitos e créditos
- **Movimento**: 
  - Débitos: `esMov = 1` (saídas)
  - Créditos: `esMov = 0` (entradas)

### Cálculo do Saldo Anterior
- **Lógica de compensação**: Saldo é a diferença entre créditos e débitos do período anterior
- **Apenas saldo positivo**: Armazena apenas se resultado for maior que zero
- **Acumulação**: Considera todos os tipos de créditos e débitos do período anterior
- **Cálculo completo**: Inclui créditos, outros créditos, estornos de débito e saldo anterior anterior

### Inicialização Rigorosa
- Todos os campos são inicializados com 0
- Prevenção de null pointer exceptions
- Estrutura consistente mesmo sem valores calculados
- Uso de `getBigDecimal_Zero()` para segurança

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração do IPI para Livro Fiscal.

### `buscarSaidas_DebitoImposto()`
Busca valores de IPI a débito nas saídas industriais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: saídas (esMov=1), CFOP 5xxx/6xxx/7xxx, não cancelados
- Consulta: Soma valores do campo IPI no JSON usando `jGet()`
- Retorna: Soma dos valores de débito ou 0

### `buscarEntradas_CreditoImposto()`
Busca valores de IPI a crédito nas entradas industriais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: entradas (esMov=0), CFOP 1xxx/2xxx/3xxx, não cancelados
- Consulta: Soma valores do campo IPI no JSON usando `jGet()`
- Retorna: Soma dos valores de crédito ou 0

### `buscarApuracaoAnterior()`
Busca apuração do período anterior.
- Parâmetros: mês, ano, tipo de apuração
- Filtro: mesmo tipo de apuração, meses anteriores
- Considera grupo centralizador (`getSamWhere().getWhereGc()`)
- Ordena por ano e mês decrescente
- Retorna: Apuração anterior ou null

## 📊 Estrutura de Saída

**JSON da Apuração (edb01json) com campos dinâmicos:**
- `debImp` - Valor com débito do imposto (saídas industriais)
- `credImp` - Valor com crédito do imposto (entradas industriais)
- `CredAnt` - Saldo credor anterior (calculado do período anterior)
- `outrosDeb` - Outros débitos (inicializado como 0)
- `estCred` - Estorno de créditos (inicializado como 0)
- `outrosCred` - Outros créditos (inicializado como 0)
- `estDeb` - Estorno de débitos (inicializado como 0)
- `deducoes` - Deduções (inicializado como 0)
- `subTotSai` - Subtotal saídas (inicializado como 0)
- `subTotEnt` - Subtotal entradas (inicializado como 0)
- `total` - Total (inicializado como 0)
- `sdoDevedor` - Saldo devedor (inicializado como 0)
- `sdoCredor` - Saldo credor (inicializado como 0)
- `impRecolher` - Imposto a recolher (inicializado como 0)

**Cálculo do Saldo Anterior:**
CréditosAnterior = credImp_anterior + outrosCred_anterior + estDeb_anterior + credAnt_anterior
DébitosAnterior = debImp_anterior + outrosDeb_anterior + estCred_anterior
SaldoCredor = CréditosAnterior - DébitosAnterior

Se SaldoCredor > 0:
credAnt_atual = SaldoCredor
Senão:
credAnt_atual = 0


## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos (DateUtils)
- `java.time` - Manipulação de datas
- `sam.server.samdev.utils` - Utilitários de parâmetros

**Módulos:**
- Módulo Fiscal
- Módulo de Documentos Fiscais
- Módulo de Apurações Fiscais
- Módulo de Livros Fiscais
- Módulo Industrial

## 📝 Observações Técnicas

### Especificidades do IPI Industrial
- **CFOP industriais**: Inclui série 7xxx para operações industriais específicas
- **Data de entrada**: Usa `esData` como referência para ambos débitos e créditos
- **Foco industrial**: Fórmula especializada para empresas do ramo industrial

### Consultas com JSON
- Uso de `jGet()` para extrair valores de campos JSON dinâmicos
- Consultas parametrizadas com filtros por padrão de CFOP
- Somas diretas no banco de dados para performance

### Inicialização Completa
- Todos os 15 campos do JSON são inicializados com 0
- Garantia de estrutura consistente para o Livro Fiscal
- Prevenção de erros em cálculos subsequentes

### Cálculo de Saldo Anterior
- **Cálculo em tempo de execução**: Não armazena saldo bruto, recalcula
- **Compensação completa**: Considera todos os componentes do período anterior
- **Apenas positivo**: Saldo negativo do período anterior é descartado
- **Cumulatividade**: Saldo anterior considera saldo anterior do anterior

### Segurança e Validação
- Verificação de null no JSON da apuração anterior
- Uso de `getBigDecimal_Zero()` para evitar null pointer
- Filtro por grupo centralizador nas consultas
- Validação de documentos não cancelados

### Performance
- Consultas otimizadas com filtros simples
- Uso de `LIKE` para padrões de CFOP (5%, 6%, 7%, etc.)
- Busca única para apuração anterior
- Cálculos mínimos em tempo de execução

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0  
**Tributo:** IPI (Imposto sobre Produtos Industrializados)  
**Destino:** Livro Fiscal (LF)  
**Setor:** Industrial  
**Especificidade:** Operações industriais com CFOP 7xxx