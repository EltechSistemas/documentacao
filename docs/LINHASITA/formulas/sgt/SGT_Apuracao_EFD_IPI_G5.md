# SGT_Apuracao_EFD_IPI_G5 - Apuração de IPI (Grupo 5) para EFD

## 📖 Descrição
Fórmula alternativa para cálculo e apuração do Imposto sobre Produtos Industrializados (IPI) conforme Grupo 5 da EFD, realizando a consolidação de débitos, créditos e saldos com lógica específica para empresas do Grupo 5.

## 🎯 Finalidade
Calcular automaticamente os valores de IPI a pagar ou compensar em um período fiscal para empresas classificadas no Grupo 5, utilizando regras específicas de cálculo e classificação conforme exigências fiscais para este grupo.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento de Tributos
- Empresas industriais do Grupo 5

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apuração fiscal principal
- `Edb0105` - Ajustes manuais do IPI
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
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
- Seleção do alinhamento específico (0032) para campos dinâmicos
- Identificação do campo JSON para valores de IPI

### 2. **Cálculo de Saldo Anterior**
- **01 - Saldo credor anterior**: Busca apuração do período anterior
  - Consulta apurações do mesmo tipo em meses anteriores
  - Extrai saldo credor do JSON da apuração anterior
  - Verifica se há JSON na apuração anterior
  - Se não houver apuração anterior ou JSON, considera zero

### 3. **Cálculo de Débitos do IPI**
- **02 - Saídas com débito do imposto**: Consolida valores de IPI nas saídas
  - Processa documentos fiscais com CFOP iniciados em 5 ou 6
  - Considera modelos de documentos: 01, 1B, 04, 55
  - Filtra por situações válidas (exclui 02, 03, 04, 05)
  - Considera ambos: entradas (esData) e saídas (abb01data)
  - Soma valores do campo IPI no JSON dos itens usando campos dinâmicos

### 4. **Cálculo de Créditos do IPI**
- **03 - Entradas com crédito do imposto**: Consolida valores de IPI nas entradas
  - Processa documentos fiscais com CFOP iniciados em 1, 2 ou 3
  - Considera os mesmos modelos de documentos que os débitos
  - Aplica os mesmos filtros de situações
  - Soma valores do campo IPI no JSON dos itens usando campos dinâmicos

### 5. **Ajustes Manuais**
- **04 - Outros débitos e estorno de créditos**: Ajustes manuais de débito
  - Processa todos os ajustes manuais registrados em Edb0105
  - Soma todos os valores sem filtro por natureza
  - Representa ajustes diversos de débito

- **05 - Outros créditos e estorno de débitos**: Ajustes manuais de crédito
  - Processa todos os ajustes manuais registrados em Edb0105
  - Soma todos os valores sem filtro por natureza
  - Representa ajustes diversos de crédito

### 6. **Cálculo de Saldos Finais**
- **06 - Total de IPI a recuperar**: Calcula saldo credor
  - Fórmula: Saldo1 = debSaidas + outrosDeb
  - Fórmula: Saldo2 = credAnt + outrosCred
  - Fórmula: Saldo = Saldo1 - Saldo2
  - Se saldo for negativo: saldoCredor = saldo * (-1)
  - Representa créditos a recuperar

- **07 - Total de IPI a recolher**: Calcula saldo devedor
  - Se saldo for positivo ou zero: saldoDevedor = saldo
  - Representa débitos a recolher

## ⚠️ Regras de Negócio

### Especificidades do Grupo 5
- **Lógica diferente de cálculo**: Inversão na fórmula de saldo
- **Tratamento de ajustes**: Sem distinção por natureza nos ajustes manuais
- **Campos dinâmicos**: Uso intensivo de campos via alinhamento

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
- **Sem filtro por natureza**: Soma todos os ajustes
- **Dupla contagem**: Mesmos ajustes considerados em débitos e créditos
- **Vinculados à apuração específica**
- **Permitem correções manuais** dos cálculos automáticos

### Fórmula Específica do Grupo 5

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração do IPI para Grupo 5.

### `buscarApuracaoAnterior()`
Busca a apuração do período anterior para cálculo de saldo credor.
- Parâmetros: mês, ano, tipo de apuração
- Verifica existência do JSON na apuração anterior
- Retorna: Objeto Edb01 da apuração anterior ou null

### `buscarDebitosIPI()`
Busca valores de IPI a débito nos documentos fiscais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: CFOP 5xxx/6xxx, modelos específicos, situações válidas
- Usa campos dinâmicos via `getCampo()`
- Retorna: Soma dos valores de débito

### `buscarCreditosIPI()`
Busca valores de IPI a crédito nos documentos fiscais.
- Parâmetros: nome do campo JSON, período (data inicial/final)
- Filtros: CFOP 1xxx/2xxx/3xxx, modelos específicos, situações válidas
- Usa campos dinâmicos via `getCampo()`
- Retorna: Soma dos valores de crédito

## 📊 Estrutura de Saída

**JSON da Apuração (edb01json) com campos dinâmicos:**
- `credAnt` - Saldo credor do período anterior
- `debSaidas` - Saídas com débito do imposto
- `credEntradas` - Entradas com crédito do imposto
- `outrosDeb` - Outros débitos e estorno de créditos
- `outrosCred` - Outros créditos e estorno de débitos
- `saldoCredor` - Total de IPI a recuperar (crédito)
- `saldoDevedor` - Total de IPI a recolher (débito)

**Cálculo do Saldo (Grupo 5):**


## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos (DateUtils, Utils)
- `java.time` - Manipulação de datas
- `sam.server.samdev.utils` - Utilitários de parâmetros

**Módulos:**
- Módulo Fiscal
- Módulo de Documentos Fiscais
- Módulo de Apurações Fiscais
- Módulo de Cadastros Fiscais

## 📝 Observações Técnicas

### Diferenças em Relação à Versão Padrão
1. **Lógica invertida** no cálculo do saldo
2. **Ajustes manuais sem filtro** por natureza
3. **Campos totalmente dinâmicos** via alinhamento
4. **Nomenclatura diferente** dos campos no JSON
5. **Fórmula específica** para Grupo 5

### Consultas com Campos Dinâmicos
- Uso de `getCampo()` para obter nomes de campos do alinhamento
- Consultas parametrizadas com campos variáveis
- Flexibilidade para diferentes configurações

### Tratamento de Períodos
- Considera o mês completo (do dia 1 ao último dia do mês)
- Usa `lengthOfMonth()` para determinar o último dia corretamente
- Diferenciamento entre data de entrada e data de documento

### Validações
- Verificação de null no JSON da apuração anterior
- Uso de `getBigDecimal_Zero()` para evitar null pointer
- Filtro por grupo centralizador nas consultas

### Performance
- Consultas otimizadas com filtros compostos
- Somas diretas no banco de dados
- Uso de índices por período e situação

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração Fiscal  
**Versão:** 1.0  
**Tributo:** IPI (Imposto sobre Produtos Industrializados)  
**Destino:** EFD (Escrituração Fiscal Digital)  
**Grupo:** 5 (Regime Específico)  
**Aplicação:** Empresas industriais do Grupo 5