# SGT_Apuracao_LF_ICMS

## 📖 Descrição
Fórmula para cálculo e apuração do Imposto sobre Circulação de Mercadorias e Serviços (ICMS) para geração do Livro Fiscal (LF), realizando a consolidação de débitos, créditos, ajustes e saldos do período com foco nas obrigações do Livro Fiscal.

## 🎯 Finalidade
Calcular automaticamente os valores de ICMS para emissão do Livro Fiscal, consolidando operações de entrada e saída, ajustes fiscais, estornos e saldos anteriores, com tratamento específico para os registros exigidos pela legislação do Livro Fiscal.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento de Tributos
- Setor de Livros Fiscais

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Edb0101` - Ajustes da apuração (ocorrências)
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Eaa01035` - Ajustes fiscais dos itens
- `Eaa01031` - Informações adicionais de ICMS
- `Aaj17` - Tipos de ajustes fiscais
- `Aaj15` - CFOP (Código Fiscal de Operações e Prestações)
- `Aah01` - Modelos de documentos fiscais
- `Aaj03` - Situações dos documentos

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Objeto de apuração fiscal contendo mês, ano e tipo |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção do objeto de apuração (Edb01)
- Inicialização/validação da estrutura JSON
- Correção de valores negativos no saldo devedor
- Seleção do alinhamento específico (0040) para campos dinâmicos
- Definição do período fiscal (mês/ano)
- Inicialização de conjuntos para classificação de ajustes

### 2. **Processamento de Ocorrências/Ajustes**
- Verifica se há ajustes já registrados na apuração
- Se não houver, busca ajustes automáticos do período:
  - Consulta ajustes fiscais (Eaa01035) do período
  - Agrupa por tipo de ajuste (Aaj17)
  - Ordena por código do ajuste
  - Cria registros de ocorrência (Edb0101) na apuração
- Se já houver ajustes, consolida valores por subitem do código

### 3. **Cálculo de Débitos do ICMS**
- **Valor com débito do imposto**: Saídas tributadas
  - Processa documentos de saída (esMov = 1)
  - CFOP diferente de 5605 (para saídas)
  - Modelos: 01, 1B, 04, 55, 65
  - Filtra situações inválidas (01, 02, 03, 04, 05, 07)
  - Soma valores do campo ICMS no JSON dos itens

### 4. **Cálculo de Créditos do ICMS**
- **Valor com crédito do imposto**: Entradas com crédito
  - Processa documentos de entrada (esMov = 0)
  - CFOP iniciados em 1, 2 ou 3
  - Documentos não cancelados
  - Soma valores do campo ICMS no JSON dos itens

### 5. **Ajustes e Estornos**
- **Estorno de débitos**: Reversão de débitos registrados
  - Busca ajustes com código terminado em "03"
  - Inclui ajustes manuais da apuração
  - Modelos diversos de documentos

- **Estorno de créditos**: Reversão de créditos utilizados
  - Busca ajustes com código terminado em "01"
  - Inclui ajustes manuais da apuração
  - Modelos diversos de documentos

- **Outros débitos**: Ajustes diversos de débito
  - Busca ajustes com código específico (terceira posição: 0, quarta: 2)
  - Inclui ajustes manuais com código terminado em "00"
  - Modelos diversos de documentos

- **Outros créditos**: Ajustes diversos de crédito
  - Busca valores da tabela Eaa01031 (ICMS adicional)
  - Inclui ajustes manuais com código terminado em "02"
  - Modelos diversos de documentos

### 6. **Saldo Anterior**
- **Saldo credor anterior**: Busca saldo do período anterior
  - Consulta apurações do mesmo tipo em meses anteriores
  - Extrai saldo credor do JSON
  - Considera apenas o registro mais recente

## ⚠️ Regras de Negócio

### Processamento de Ocorrências
- **Agrupamento automático**: Ajustes do período são agrupados por tipo
- **Ordenação**: Ajustes ordenados por código para organização do Livro Fiscal
- **Limite de observação**: Texto da ocorrência limitado a 100 caracteres
- **Mapeamento de campos**: Copia informações GIA do tipo de ajuste

### Filtros de Documentos
- **Situações excluídas**: 01, 02, 03, 04, 05, 07 (para débitos)
- **Indicador EFD-ICMS**: Apenas documentos com `eaa01iEfdIcms = 1`
- **Modelos para débitos**: 01, 1B, 04, 55, 65
- **Modelos para ajustes**: Lista mais ampla incluindo serviços, energia, etc.

### Classificação por CFOP
- **Débitos**: CFOP diferente de 5605 (para saídas)
- **Créditos**: CFOP iniciados em 1, 2 ou 3
- **Lógica simplificada**: Uso de `LIKE` para verificação de padrões

### Classificação de Ajustes
- **Código Aaj17 (posições 3-4)**:
  - "00": Outros débitos
  - "01": Estorno de créditos
  - "02": Outros créditos
  - "03": Estorno de débitos

- **Posição 3**:
  - "0": Ajustes diversos

### Tratamento de Períodos
- **Débitos**: Data do documento (abb01data)
- **Créditos**: Data de entrada (esData)
- **Ajustes**: Data de entrada (esData)
- **Busca anterior**: Considera meses anteriores ao atual

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração do ICMS para Livro Fiscal.

### `buscarAjustesDaApurICMS()`
Busca ajustes fiscais do período para criação automática de ocorrências.
- Parâmetros: período (data inicial/final)
- Retorna: Lista de ajustes agrupáveis

### `buscarSaidas_DebitoImposto()`
Busca valores de ICMS a débito nas saídas.
- Parâmetros: nome do campo JSON, período
- Filtros: saídas, CFOP específico, modelos, situações
- Retorna: Soma dos valores de débito

### `buscarEntradas_CreditoImposto()`
Busca valores de ICMS a crédito nas entradas.
- Parâmetros: nome do campo JSON, período
- Filtros: entradas, CFOP 1xxx/2xxx/3xxx, não cancelados
- Retorna: Soma dos valores de crédito

### `buscarOutrosDedICMS()`, `buscarOutrosCredICMS()`
Buscam ajustes diversos de débito e crédito.
- Filtram por códigos específicos de ajuste
- Incluem múltiplos modelos de documentos

### `buscarEstornoDebICMS()`, `buscarEstornoCredICMS()`
Buscam estornos de débitos e créditos.
- Filtram por códigos específicos de estorno
- Modelos diversos de documentos

### `buscarApuracaoAnterior()`
Busca apuração do período anterior.
- Parâmetros: mês, ano, tipo de apuração
- Considera grupo centralizador
- Retorna: Apuração anterior ou null

### `buscarSaldoCredorAnterior()`
Busca saldo credor do período anterior.
- Parâmetros: nome do campo, ano, mês, tipo de apuração
- Extrai valor do JSON da apuração anterior
- Retorna: Valor do saldo credor ou 0

## 📊 Estrutura de Saída

**JSON da Apuração (edb01json) com campos dinâmicos:**
- `debImp` - Valor com débito do imposto (saídas)
- `outrosDeb` - Outros débitos
- `estCred` - Estorno de créditos
- `credImp` - Valor com crédito do imposto (entradas)
- `outrosCred` - Outros créditos
- `estDeb` - Estorno de débitos
- `credAnt` - Saldo credor anterior
- `deducoes` - Deduções (inicializado como 0)
- `subTotSai` - Subtotal saídas (inicializado como 0)
- `subTotEnt` - Subtotal entradas (inicializado como 0)
- `total` - Total (inicializado como 0)
- `saldoDevedor` - Saldo devedor (corrigido se negativo)
- `saldoCredor` - Saldo credor (inicializado como 0)
- `impRecolher` - Imposto a recolher (inicializado como 0)

**Ocorrências (Edb0101):**
- Ajustes agrupados por tipo (Aaj17)
- Com informações GIA (SI, Função Legal, Ocorrência)
- Observação limitada a 100 caracteres

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos (DateUtils, Utils)
- `java.time` - Manipulação de datas
- `sam.server.samdev.utils` - Utilitários de parâmetros

**Estruturas de Dados:**
- `Set<String>` para classificação de débitos, créditos e estornos
- `Map<String, BigDecimal>` para consolidação de valores por subitem
- `TreeSet<Aaj17>` para ordenação de ajustes

**Módulos:**
- Módulo Fiscal
- Módulo de Documentos Fiscais
- Módulo de Apurações Fiscais
- Módulo de Livros Fiscais

## 📝 Observações Técnicas

### Diferenciais do Livro Fiscal
- **Foco em ocorrências**: Agrupamento automático de ajustes
- **Campos GIA**: Informações específicas para Guia de Informação e Apuração
- **Organização por código**: Ordenação dos ajustes para apresentação
- **Validação de saldo**: Correção de valores negativos no saldo devedor

### Consultas Híbridas
- **JSON para valores principais**: Uso de `jGet()` para campos dinâmicos
- **Tabelas específicas para ajustes**: Eaa01035 para ajustes, Eaa01031 para créditos adicionais
- **Consultas parametrizadas**: Com filtros complexos por código de ajuste

### Inicialização Completa
- Todos os campos do JSON são inicializados com 0
- Prevenção de null pointer exceptions
- Estrutura consistente mesmo sem valores

### Performance
- Agrupamento no banco quando possível
- Consultas otimizadas com filtros específicos
- Uso de conjuntos para classificação eficiente

### Validações
- Verificação de coleções vazias com `Utils.isEmpty()`
- Correção de valores negativos no saldo devedor
- Limitação de texto para observações
- Filtro por grupo centralizador

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0  
**Tributo:** ICMS  
**Destino:** Livro Fiscal (LF)  
**Finalidade:** Apuração e escrituração fiscal