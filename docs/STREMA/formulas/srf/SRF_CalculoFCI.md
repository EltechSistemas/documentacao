# SRF - Cálculo da FCI (Equilibrio)

## 📖 Descrição
Fórmula para cálculo e geração de Ficha de Conteúdo de Importação (FCI), considerando composições de produtos (BOM), entradas de itens importados, saídas do produto final e regras de negócio específicas do módulo Equilibrio.

## 🎯 Finalidade
Calcular automaticamente o conteúdo de importação (CI) de produtos compostos, com base nas entradas de componentes importados e nas vendas do produto final, gerando registros de FCI para fins fiscais e de controle de origem da mercadoria.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controle de Qualidade
- Compras

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abp20` - Composição de produtos (BOM)
- `Abm01` - Cadastro de itens
- `Abm0101` - Configurações fiscais do item
- `Abm12` - CST de origem da mercadoria
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal
- `Eab01` - Cadastro de FCI
- `Eab0101` - Cálculos da FCI
- `Eab01011` - Itens importados da FCI
- `Aba20` / `Aba2001` - Repositório de dados (configurações da tarefa SRF2002)

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dataCalculo | LocalDate | Sim | Data base para o cálculo da FCI |
| bom | String | Sim | Código da composição (BOM) |
| whereItem | ClientCriterion | Sim | Critério para seleção de produtos |
| dtIniEnt | LocalDate | Sim | Data inicial para entradas |
| dtFinEnt | LocalDate | Sim | Data final para entradas |
| dtIniSai | LocalDate | Sim | Data inicial para saídas |
| dtFinSai | LocalDate | Sim | Data final para saídas |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento das configurações do repositório (Aba2001)
- Definição dos campos de alinhamento para valores e quantidades

### 2. **Seleção de Produtos e Componentes**
- Busca de produtos com base no BOM informado
- Identificação de componentes importados (CST 1, 2, 6, 7, 8)
- Carregamento do último cálculo de FCI existente

### 3. **Cálculo das Entradas (Importações)**
- Soma de quantidades e valores de entradas no período
- Ajustes retroativos conforme configuração
- Cálculo do Valor Médio Unitário (VMU) por componente
- Controle de médias a partir da última FCI

### 4. **Cálculo das Saídas (Vendas)**
- Soma de quantidades e valores de saídas no período
- Filtro por UF (interestaduais ou todas)
- Ajustes retroativos conforme configuração
- Cálculo do unitário médio de saída

### 5. **Cálculo do CI e Geração da FCI**
- Cálculo do Conteúdo de Importação: `CI = (Total VMU Importações / Unitário Saídas) * 100`
- Validação de variação mínima de 5% em relação ao último CI
- Criação/atualização dos registros de FCI (Eab01, Eab0101, Eab01011)
- Atualização automática da origem da mercadoria no item (CST-A)

### 6. **Validações Finais**
- Verificação de inclusão de pelo menos um cálculo
- Validação de consistência entre unitários de entrada e saída
- Tratamento de exceções e mensagens de erro

## ⚠️ Regras de Negócio

### Configurações do Repositório (SRF2002)
- **tipoSaida**: 1=Interestaduais, 2=Todas
- **considEntRetro**: Considerar entradas retroativas
- **consEntRetroSomenteUltMes**: Somente último mês para retroativos
- **considSaiRetro**: Considerar saídas retroativas
- **consSaiRetroSomenteUltMes**: Somente último mês para retroativos
- **considMediaUnitAPartirDaUltimaFCI**: Considerar médias da última FCI
- **gerarFCIcomCIZero**: Gerar FCI mesmo com CI zero
- **atualizarOrigemMercadoria**: Atualizar CST-A automaticamente

### Cálculo do CI
- **CI ≤ 40%**: Origem 5 (Nacional - Mercadoria ou Bem com Conteúdo de Importação inferior ou igual a 40%)
- **CI 40,01% a 70%**: Origem 3 (Nacional - Mercadoria ou Bem com Conteúdo de Importação superior a 40% e inferior ou igual a 70%)
- **CI > 70%**: Origem 8 (Nacional - Mercadoria ou Bem com Conteúdo de Importação superior a 70%)

### Validações de Dados
- Interrupção se nenhum produto for selecionado
- Validação de unitário de importação ≤ unitário de saída
- Verificação de variação mínima de 5% no CI
- Suporte a cálculos retroativos

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de cálculo da FCI.

### `buscarRepositorio()`
Carrega as configurações da tarefa SRF2002 do repositório de dados.

### `validarAlinhamento(String codigoAlinhamento)`
Valida os campos de alinhamento para valores e quantidades.

### `buscarProdutosComBOM(Criterion critItem, String bom)`
Busca produtos com base no BOM informado.

### `buscarIdsItensImportados(Set<Long> abm01ids)`
Identifica componentes importados pelo CST.

### `selecionarEntradas(...)`
Seleciona e calcula entradas de componentes importados.

### `selecionarSaidas(...)`
Seleciona e calcula saídas do produto principal.

### `setarOrigemMercadoriaNoItemByCI(...)`
Atualiza a origem da mercadoria (CST-A) com base no CI calculado.

## 📊 Estrutura de Saída

**Registros Criados/Atualizados:**
- `Eab01` - Cadastro da FCI do item
- `Eab0101` - Cálculo da FCI (data, CI, valores)
- `Eab01011` - Itens importados do cálculo (VMU, quantidades)

**Campos do Eab0101:**
- `eab0101unitImp` - Unitário total das importações
- `eab0101unitSai` - Unitário médio das saídas
- `eab0101ci` - Conteúdo de Importação calculado
- `eab0101qtdSai` - Quantidade total de saídas
- `eab0101vlrSai` - Valor total de saídas
- `eab0101status` - Status da FCI (A_ENVIAR, etc.)

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data e validação
- `sam.dicdados` - Tipos de fórmula
- `sam.dto.cgs` - DTOs do módulo CGS
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Serviços:**
- `CGSService` - Serviço do módulo CGS
- `ParametroService` - Serviço de parâmetros
- `EspecificacaoService` - Serviço de especificações
- `CASService` - Serviço de composição de JSON

**Módulo:** Equilibrio

## 📝 Observações Técnicas

### Tratamento de Datas
- Suporte a períodos flexíveis para entradas e saídas
- Cálculos retroativos configuráveis
- Busca inteligente do último mês com movimentação

### Campos de Alinhamento
- `FCICAMPOVLRIMP` - Campo para valor de importação
- `FCICAMPOQTDIMP` - Campo para quantidade de importação
- `FCICAMPOVLRSAI` - Campo para valor de saída
- `FCICAMPOQTDSAI` - Campo para quantidade de saída

### JSON da Especificação
- Composição dinâmica dos campos da FCI
- Suporte a campos livres configuráveis
- Integração com o serviço CAS

### Validações de Negócio
- Controle rigoroso de arredondamentos (6 casas decimais)
- Tratamento de casos sem movimentação
- Suporte a múltiplos cálculos por item

---

**Última Alteração:** 02/12/2025 às 15:25  
**Autor:** Bruno  
**Tipo:** Fórmula de Cálculo de FCI  
**Versão:** 1.0  
**Observação:** O funcionamento dessa fórmula depende 100% do script na tarefa SRF2002 e do repositório de dados. Ver documentação no Trello.