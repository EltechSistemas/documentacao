# SGT_Apuracao_EFD_ICMS

## 📖 Descrição
Fórmula para cálculo e apuração do Imposto sobre Circulação de Mercadorias e Serviços (ICMS) para geração da Escrituração Fiscal Digital (EFD), realizando a consolidação de débitos, créditos, ajustes e saldos do período.

## 🎯 Finalidade
Calcular automaticamente os valores de ICMS a pagar ou compensar em um período fiscal, consolidando operações de entrada e saída, ajustes fiscais, deduções e saldos anteriores para geração da EFD-ICMS.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento de Tributos

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Edb0101` - Ajustes da apuração
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Eaa01035` - Ajustes fiscais dos itens
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
- Inicialização da estrutura JSON para armazenamento dos cálculos
- Definição do período fiscal (mês/ano)
- Seleção do alinhamento para campos dinâmicos

### 2. **Cálculo de Débitos do ICMS**
- **01 - Saídas com débito do imposto**: Consolida valores de ICMS nas saídas
  - Processa 8 grupos de documentos (C190, C320, C390, C490, C590, D190, D590, C850/C890)
  - Considera entradas e saídas conforme CFOP específico
  - Filtra por modelos de documentos válidos

- **02 - Ajustes de débito 1**: Ajustes fiscais nos documentos
  - Soma ajustes com códigos específicos (terceira posição: 3,4,5; quarta: 0,3,4,5,6,7,8)

- **03 - Ajustes de débito 2**: Ajustes na apuração
  - Inclui ajustes da própria apuração (Edb0101) com códigos terminados em "00"

- **04 - Estorno de créditos**: Reversão de créditos utilizados
  - Inclui ajustes da apuração com códigos terminados em "01"

### 3. **Cálculo de Créditos do ICMS**
- **05 - Entradas com crédito do imposto**: Consolida valores de ICMS nas entradas
  - Processa 4 grupos de documentos (C190, C590, D190, D590)
  - Considera entradas e saídas conforme CFOP específico

- **06 - Ajustes à crédito 1**: Ajustes fiscais favoráveis nos documentos
  - Soma ajustes com códigos específicos (terceira posição: 0,1,2; quarta: 0,3,4,5)

- **07 - Ajustes à crédito 2**: Ajustes favoráveis na apuração
  - Inclui ajustes da apuração com códigos terminados em "02"

- **08 - Estorno de débitos**: Reversão de débitos registrados
  - Inclui ajustes da apuração com códigos terminados em "03"

### 4. **Cálculo de Saldos e Compensações**
- **09 - Saldo credor anterior**: Busca saldo credor do período anterior
- **10 - Saldo devedor antes das deduções**: Calcula saldo bruto (débitos - créditos)
- **11 - Deduções**: Aplica deduções autorizadas
  - Ajustes fiscais com código específico (terceira posição: 6)
  - Ajustes na apuração com código "04"
  - Ajustes da própria apuração com códigos terminados em "04"

- **12 - Saldo devedor depois das deduções**: Saldo líquido a pagar
- **13 - Saldo credor a transportar**: Saldo favorável para período seguinte

### 5. **Valores Extra-Apuração**
- **14 - Valores extra-apuração**: Operações não incluídas na apuração normal
  - Processa 7 grupos de documentos (C100, C300, C400, C500, D100, D500, D590)
  - Inclui ajustes com códigos específicos
  - Considera situações especiais de documentos (01, 07)

## ⚠️ Regras de Negócio

### Filtros de Documentos
- **Situações excluídas**: 01, 02, 03, 04, 05, 07 (conforme aaj03efd)
- **Indicador EFD-ICMS**: Apenas documentos com `eaa01iEfdIcms = 1`
- **CFOP para débitos**: 5605 (saídas) e 1605 (entradas)
- **CFOP para créditos**: 1605 (entradas) e 5605 (saídas)

### Classificação de Ajustes
- **Posição 3 do código**: Tipo do ajuste
  - 0,1,2: Créditos
  - 3,4,5: Débitos
  - 6: Deduções
  - 7: Extra-apuração

- **Posição 4 do código**: Sub-tipo do ajuste
  - Varia conforme o tipo principal

### Modelos de Documentos por Grupo
- **Débitos**: 8 grupos com modelos específicos
- **Créditos**: 4 grupos com modelos específicos
- **Extra-apuração**: 7 grupos com modelos específicos

### Tratamento de Períodos
- **Entradas**: Considera data de entrada (esData)
- **Saídas**: Considera data do documento (abb01data)
- **Período anterior**: Busca último registro do mesmo tipo

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração do ICMS.

### `buscarDebitosICMS()`
Busca valores de ICMS a débito nos documentos fiscais.
- Parâmetros: nome do campo, período, tipo (entrada/saída), grupo de registros
- Retorna: Soma dos valores de débito

### `buscarCreditosICMS()`
Busca valores de ICMS a crédito nos documentos fiscais.
- Parâmetros: nome do campo, período, tipo (entrada/saída), grupo de registros
- Retorna: Soma dos valores de crédito

### `buscarAjustesDebDoctsICMS()`, `buscarAjustesCredDoctsICMS()`
Buscam ajustes de débito/crédito nos documentos.
- Filtram por código de ajuste específico
- Consideram diferentes modelos de documentos

### `buscarAjustesDebApurICMS()`, `buscarAjustesCredApurICMS()`
Buscam ajustes de débito/crédito na apuração.
- Incluem mais modelos de documentos
- Filtram situações de documentos diferentes

### `buscarEstornoCredICMS()`, `buscarEstornoDebICMS()`
Buscam estornos de créditos e débitos.
- Códigos específicos para estornos

### `buscarSaldoCredorAnterior()`
Busca saldo credor do período anterior.
- Consulta apurações do mesmo tipo em meses anteriores

### `buscarDeducoesAjustesFiscais()`, `buscarDeducoesAjustesApuracao()`
Buscam valores de deduções autorizadas.
- Ajustes fiscais e de apuração específicos

### `buscarValorExtraApur()`, `buscarValorExtraApurICMS()`, `buscarValorExtraApurICMSEstorno()`
Buscam valores fora da apuração normal.
- Documentos com situações especiais
- Ajustes de extra-apuração

### `buscarModelosDeb()`, `buscarModelosCred()`, `buscarModelosExtraApur()`
Definem listas de modelos de documentos por grupo.
- Retornam lista específica para cada tipo de consulta

## 📊 Estrutura de Saída

**JSON da Apuração (edb01json):**
- `debSaidas` - Saídas com débito do imposto
- `debAjustes` - Ajustes de débito nos documentos
- `debAjustesApur` - Ajustes de débito na apuração
- `estornoCred` - Estorno de créditos
- `credEntradas` - Entradas com crédito do imposto
- `credAjustes` - Ajustes à crédito nos documentos
- `credAjustesApur` - Ajustes à crédito na apuração
- `estornoDeb` - Estorno de débitos
- `saldoCredorAnt` - Saldo credor do período anterior
- `saldo` - Saldo devedor antes das deduções
- `deducoes` - Deduções aplicáveis
- `saldoDevedor` - Saldo devedor líquido
- `saldoCredor` - Saldo credor a transportar
- `valoresExtra` - Valores extra-apuração

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos
- `java.time` - Manipulação de datas

**Módulos:**
- Módulo Fiscal
- Módulo de Documentos Fiscais
- Módulo de Apurações Fiscais

## 📝 Observações Técnicas

### Consultas Dinâmicas
- Uso de campos JSON dinâmicos (`jGet()`)
- Filtros por posição de caracteres em códigos
- Consultas parametrizadas por grupo de modelos

### Performance
- Consultas otimizadas com filtros específicos
- Somas diretas no banco de dados
- Uso de índices por período e situação

### Flexibilidade
- Suporte a múltiplos modelos de documentos
- Configuração por tipo de apuração
- Campos dinâmicos via alinhamento

### Validações
- Exclusão de situações de documento inválidas
- Verificação do indicador EFD-ICMS
- Filtros por CFOP adequados

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0  
**Tributo:** ICMS  
**Destino:** EFD (Escrituração Fiscal Digital)