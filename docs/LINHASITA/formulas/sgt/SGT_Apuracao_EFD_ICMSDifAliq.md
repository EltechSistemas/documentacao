# SGT_Apuracao_EFD_ICMSDifAliq - Apuração de Diferencial de Alíquota do ICMS e FCP

## 📖 Descrição
Fórmula para cálculo e apuração do Diferencial de Alíquota do ICMS (DIFAL) e do Fundo de Combate à Pobreza (FCP) para geração da Escrituração Fiscal Digital (EFD), contemplando operações interestaduais com diferentes alíquotas entre estados.

## 🎯 Finalidade
Calcular automaticamente os valores de DIFAL e FCP devidos em operações interestaduais, considerando a diferença entre alíquotas interna e interestadual, além de gerar apurações específicas por estado da federação.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento de Tributos

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Eaa01035` - Ajustes fiscais dos itens
- `Eaa0101` - Informações complementares dos documentos
- `Aag02` - Estados da federação
- `Aag0201` - Municípios
- `Aaj17` - Tipos de ajustes fiscais
- `Aah01` - Modelos de documentos fiscais
- `Aac10` - Empresa

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Objeto de apuração fiscal contendo mês, ano e tipo |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Obtenção do objeto de apuração (Edb01)
- Identificação da empresa ativa
- Definição do período fiscal (mês/ano)
- Seleção do alinhamento para campos dinâmicos
- Identificação dos campos JSON para DIFAL e FCP

### 2. **Identificação dos Estados**
- Verifica se há estado específico na apuração
- Se não houver, busca todos os estados da federação
- Exclui estado "EX" (Exterior)
- Identifica se o estado é o mesmo da empresa

### 3. **Processamento por Estado**
Para cada estado identificado:

#### **Verificação Inicial**
- Verifica se é a primeira apuração para o estado
- Identifica se o estado é o mesmo da empresa (UF de origem)

#### **Cálculo do DIFAL (Diferencial de Alíquota)**

**3.1 Saldo Credor Anterior - DIFAL**
- Busca saldo credor do período anterior para o estado

**3.2 Saídas com Débito do Imposto - DIFAL**
- Processa 2 grupos de documentos (C101 e D101)
- Diferencia campos conforme UF da empresa:
  - Se UF da empresa: usa campo ICMS UF Remetente
  - Se outra UF: usa campo ICMS UF Destinatário
- Considera entradas e saídas
- Calcula valores do primeiro período (situações especiais)

**3.3 Ajustes de Débito - DIFAL**
- Busca ajustes fiscais de débito específicos para DIFAL

**3.4 Créditos do ICMS referente a DIFAL**
- Processa 2 grupos de documentos (C101 e D101)
- Diferencia campos conforme UF da empresa:
  - Se UF da empresa: usa campo ICMS UF Destinatário
  - Se outra UF: usa campo ICMS UF Remetente

**3.5 Ajustes de Crédito - DIFAL**
- Busca ajustes fiscais de crédito específicos para DIFAL

**3.6 Cálculo de Saldos - DIFAL**
- Saldo devedor antes das deduções
- Deduções específicas para DIFAL
- Valor a recolher (saldo líquido)
- Saldo credor a transportar
- Valores extra-apuração

#### **Cálculo do FCP (Fundo de Combate à Pobreza)**

**3.7 Saldo Credor Anterior - FCP**
- Busca saldo credor do período anterior para o estado

**3.8 Saídas com Débito do Imposto - FCP**
- Processa 2 grupos de documentos (C101 e D101)
- Usa campo FCP UF Destinatário
- Considera entradas e saídas
- Calcula valores do primeiro período

**3.9 Ajustes de Débito - FCP**
- Busca ajustes fiscais de débito específicos para FCP

**3.10 Entradas com Crédito do Imposto - FCP**
- Processa 2 grupos de documentos (C101 e D101)
- Usa campo FCP UF Destinatário

**3.11 Ajustes de Crédito - FCP**
- Busca ajustes fiscais de crédito específicos para FCP

**3.12 Cálculo de Saldos - FCP**
- Saldo devedor antes das deduções
- Deduções específicas para FCP
- Valor a recolher (saldo líquido)
- Saldo credor a transportar
- Valores extra-apuração

### 4. **Criação das Apurações por Estado**
- Cria objeto Edb01 específico para cada estado
- Preenche JSON com todos os valores calculados
- Valida se há valores significativos para criar a apuração
- Persiste apuração no banco de dados

## ⚠️ Regras de Negócio

### Lógica de Campos por UF
- **UF da Empresa (Remetente)**:
  - DIFAL Débito: `vl_icms_uf_rem`
  - DIFAL Crédito: `vl_icms_uf_dest`
- **Outras UFs (Destinatário)**:
  - DIFAL Débito: `vl_icms_uf_dest`
  - DIFAL Crédito: `vl_icms_uf_rem`
- **FCP (sempre)**: `vl_fcp_uf_dest`

### Filtros de Documentos
- **Situações excluídas**: 01, 07 (para valores normais)
- **Situações incluídas**: 01, 07 (para primeiro período)
- **Indicador EFD-ICMS**: Apenas documentos com `eaa01iEfdIcms = 1`
- **Documentos não cancelados**: `eaa01cancData IS NULL`

### Classificação de Ajustes
- **Posição 3 do código Aaj17**:
  - "2": Ajustes de DIFAL
  - "3": Ajustes de FCP
- **Posição 4 do código Aaj17**:
  - "0", "1": Débitos
  - "2", "3": Créditos/Deduções
  - "5": Extra-apuração

### Modelos de Documentos
- **Grupo C101**: Modelos 01, 1B, 04, 55, 65
- **Grupo D101**: Modelos 07, 08, 8B, 09, 10, 11, 26, 27, 57, 67, 63

### Primeira Apuração
- Considera valores com situações 01 e 07 como "primeiro período"
- Aplica lógica diferente para cálculo de extra-apuração

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração de DIFAL e FCP.

### `buscarEstados()`
Busca todos os estados da federação para processamento.

### `isPrimeiraApuracao()`
Verifica se é a primeira apuração para um determinado estado.

### `buscarSaldoCredorAnterior()`
Busca saldo credor do período anterior para um estado específico.

### `buscarTotalImpostoDifalFcp()`
Busca valores totais de DIFAL ou FCP nos documentos fiscais.
- Parâmetros: estado, é UF da empresa, campo JSON, período, tipo (saída/entrada), grupo
- Retorna: Soma dos valores do imposto

### `buscarTotalImpostoPrimeiroPeriodoDifalFcp()`
Busca valores do primeiro período (situações especiais).

### `buscarAjustesDebitosICMSDifalFcp()`, `buscarAjustesCreditosICMSDifalFcp()`
Buscam ajustes de débito/crédito para DIFAL ou FCP.
- Diferenciam por tipo de imposto (0=DIFAL, 1=FCP)

### `buscarDeducoesICMSDifalFcp()`
Busca valores de deduções para DIFAL ou FCP.

### `buscarExtraApurICMS()`
Busca valores de extra-apuração para DIFAL ou FCP.

### `buscarModelosTotalImposto()`
Define listas de modelos de documentos por grupo de registro.

### `validaContemApuracao()`
Valida se há valores significativos para criar a apuração.

## 📊 Estrutura de Saída

**JSON da Apuração por Estado (edb01json):**

**Campos DIFAL:**
- `sdoCredAntDifal` - Saldo credor anterior DIFAL
- `debDifal` - Débitos de DIFAL
- `ajuDebDifal` - Ajustes de débito DIFAL
- `credDifal` - Créditos de DIFAL
- `ajuCredDifal` - Ajustes de crédito DIFAL
- `sdoDevDifal` - Saldo devedor DIFAL
- `deducoesDifal` - Deduções DIFAL
- `vlrRecolDifal` - Valor a recolher DIFAL
- `sdoCredDifal` - Saldo credor a transportar DIFAL
- `extraDifal` - Valores extra-apuração DIFAL

**Campos FCP:**
- `sdoCredAntFcp` - Saldo credor anterior FCP
- `debFcp` - Débitos de FCP
- `ajuDebFcp` - Ajustes de débito FCP
- `credFcp` - Créditos de FCP
- `ajuCredFcp` - Ajustes de crédito FCP
- `sdoDevFcp` - Saldo devedor FCP
- `deducoesFcp` - Deduções FCP
- `vlrRecolFcp` - Valor a recolher FCP
- `sdoCredFcp` - Saldo credor a transportar FCP
- `extraFcp` - Valores extra-apuração FCP

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos
- `java.time` - Manipulação de datas
- `sam.server.samdev.utils` - Utilitários de parâmetros

**Módulos:**
- Módulo Fiscal
- Módulo de Documentos Fiscais
- Módulo de Apurações Fiscais
- Módulo de Cadastros (Estados/Municípios)

## 📝 Observações Técnicas

### Processamento por Estado
- Gera apurações independentes para cada estado
- Otimiza consultas filtrando por estado
- Permite compensações específicas por jurisdição

### Consultas JSON Dinâmicas
- Uso de operador `->>` para extrair valores JSON
- Campos dinâmicos configuráveis via alinhamento
- Consultas otimizadas com filtros específicos

### Performance
- Consultas parametrizadas por estado e período
- Somas diretas no banco de dados
- Validação para evitar apurações vazias

### Situações Especiais
- Tratamento diferenciado para primeiro período
- Lógica específica para extra-apuração
- Consideração de situações de documentos (01, 07)

### Validações
- Exclusão de estados do exterior (EX)
- Verificação de cancelamento de documentos
- Validação do indicador EFD-ICMS
- Filtro por endereço principal do documento

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0  
**Tributo:** DIFAL (ICMS) e FCP  
**Destino:** EFD (Escrituração Fiscal Digital)  
**Alcance:** Operações Interestaduais