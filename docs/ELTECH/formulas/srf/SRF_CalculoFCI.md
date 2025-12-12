# SRF - Cálculo da FCI (Equilibrio) - Versão Atualizada

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
- `Aag0201` / `Aag02` - Municípios e UFs

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
- Carregamento das configurações do repositório SRF2002
- Validação dos parâmetros obrigatórios
- Carregamento dos campos de alinhamento para valores e quantidades

### 2. **Inicialização de Serviços**
- Instanciação dos serviços necessários (CGSService, ParametroService, etc.)
- Busca da especificação da FCI e seus campos
- Obtenção da UF da empresa ativa

### 3. **Seleção de Produtos e Componentes**
- Busca de produtos com base no BOM informado
- Identificação de componentes importados (CST 1, 2, 6, 7, 8)
- Carregamento do último cálculo de FCI existente para cada produto

### 4. **Cálculo das Entradas (Importações)**
- Inicialização de mapas para itens importados
- Seleção de entradas no período configurado
- Cálculo do Valor Médio Unitário (VMU) por componente
- Controle de médias a partir da última FCI quando habilitado

### 5. **Cálculo das Saídas (Vendas)**
- Seleção de saídas no período configurado
- Filtro por UF (interestaduais ou todas) conforme tipoSaida
- Cálculo do unitário médio de saída
- Controle de médias a partir da última FCI

### 6. **Cálculo do CI e Geração da FCI**
- Cálculo do Conteúdo de Importação: `CI = (Total VMU Importações / Unitário Saídas) * 100`
- Validação de consistência (unitário importação ≤ unitário saída)
- Criação/atualização dos registros de FCI (Eab01, Eab0101, Eab01011)
- Composição do JSON da especificação
- Atualização automática da origem da mercadoria no item (CST-A)

### 7. **Validações Finais**
- Verificação de inclusão de pelo menos um cálculo
- Tratamento de exceções e mensagens de erro

## ⚠️ Regras de Negócio

### Configurações do Repositório (SRF2002 - JSON)
- **srf2002_tpsaida**: Tipo de saída (1=Interestaduais, 2=Todas)
- **srf2002_entretro**: Considerar entradas retroativas
- **srf2002_entretroultmes**: Considerar apenas último mês para entradas retroativas
- **srf2002_consisaidaretro**: Considerar saídas retroativas
- **srf2002_saidaultmes**: Considerar apenas último mês para saídas retroativas
- **srf2002_mediaunitultfci**: Considerar médias da última FCI
- **srf2002_cizero**: Gerar FCI mesmo com CI zero
- **srf2002_attorigmerc**: Atualizar origem da mercadoria automaticamente

### Cálculo do CI e Origem da Mercadoria
- **CI ≤ 40%**: Origem 5 (Nacional - Conteúdo de Importação ≤ 40%)
- **CI 40,01% a 70%**: Origem 3 (Nacional - Conteúdo de Importação 40-70%)
- **CI > 70%**: Origem 8 (Nacional - Conteúdo de Importação > 70%)

### Critérios para Componentes Importados
- CSTs considerados: 1, 2, 6, 7, 8
- Filtro por empresa ativa
- Consideração apenas de itens na composição (BOM)

### Filtros de Documentos
- **Entradas**: Documentos fiscais de classe 1, movimento 0, com indicação FCI
- **Saídas**: Documentos fiscais de classe 1, movimento 1, com indicação FCI
- **Status**: Não cancelados, aprovados, não bloqueados, status ativo
- **Retorno**: Sem indicação de retorno (retInd = 0)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de cálculo da FCI.

### `buscarRepositorio()`
Carrega as configurações da tarefa SRF2002 do repositório de dados (Aba20/Aba2001).

### `validarAlinhamento(String codigoAlinhamento)`
Valida os campos de alinhamento obrigatórios para o cálculo:
- FCICAMPOQTDIMP (quantidade importação)
- FCICAMPOQTDSAI (quantidade saída)

### `obterUFEmpresa()`
Obtém a UF da empresa ativa a partir do município cadastrado.

### `buscarProdutosComBOM(Criterion critItem, String bom)`
Busca produtos com base no BOM informado e critério de seleção.

### `buscarIdsItensImportados(Set<Long> abm01ids)`
Identifica componentes importados pelo CST entre os itens da composição.

### `buscarUltimoCalculo(Long abm01id)`
Busca o último cálculo de FCI para o item principal.

### `buscarUltimoVMU(Long abm01idFci, Long abm01idImp, LocalDate dataCalc)`
Busca o último VMU calculado para um componente específico.

### `selecionarEntradas(...)`
Seleciona e calcula entradas de componentes importados com suporte a retroativos.

### `buscarSomaEntradasFCIPorItemQtde(...)`
Busca a soma de quantidades e valores de entradas para um componente.

### `selecionarSaidas(...)`
Seleciona e calcula saídas do produto principal com suporte a retroativos.

### `selecionarDocumentosDeSaidas(...)`
Seleciona documentos de saída conforme tipo configurado.

### `buscarSomaSaidasFCIPorItemQtde(...)`
Busca a soma de quantidades e valores de saídas para o produto principal.

### `setarOrigemMercadoriaNoItemByCI(...)`
Atualiza a origem da mercadoria (CST-A) com base no CI calculado.

### `buscarNCMDoItem(Long abm01id)`
Busca o código NCM do item.

### `buscarUMVDoItem(Long abm01id)`
Busca a unidade de medida de venda do item.

## 📊 Estrutura de Saída

**Registros Criados/Atualizados:**
- `Eab01` - Cadastro da FCI do item
- `Eab0101` - Cálculo da FCI com os campos:
  - `eab0101unitImp` - Unitário total das importações
  - `eab0101unitSai` - Unitário médio das saídas
  - `eab0101ci` - Conteúdo de Importação calculado
  - `eab0101qtdSai` - Quantidade total de saídas
  - `eab0101vlrSai` - Valor total de saídas
  - `eab0101status` - Status da FCI (A_ENVIAR)
  - `eab0101json` - JSON com campos da especificação
- `Eab01011` - Itens importados do cálculo (VMU, quantidades, valores)

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data, validação e coleções
- `sam.dicdados` - Tipos de fórmula e parâmetros
- `sam.dto.cgs` - DTOs do módulo CGS
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Serviços:**
- `CGSService` - Serviço do módulo CGS (para composições)
- `ParametroService` - Serviço de parâmetros do sistema
- `EspecificacaoService` - Serviço de especificações de campos
- `CASService` - Serviço de composição de JSON

**Módulo:** Equilibrio

## 📝 Observações Técnicas

### Campos de Alinhamento
- **FCICAMPOVLRIMP** - Campo para valor de importação (opcional)
- **FCICAMPOQTDIMP** - Campo para quantidade de importação (obrigatório)
- **FCICAMPOVLRSAI** - Campo para valor de saída (opcional)
- **FCICAMPOQTDSAI** - Campo para quantidade de saída (obrigatório)

### Cálculo de Unitários
- Arredondamento para 6 casas decimais em quantidades e VMUs
- Arredondamento para 2 casas decimais em valores monetários
- Controle de divisão por zero em cálculos de unitários

### Suporte a Retroativos
- Busca inteligente do último mês com movimentação
- Suporte a períodos desde sempre quando não há movimentação recente
- Consideração de cálculos anteriores da FCI como referência

### Validações de Negócio
- Verificação de consistência: unitário importação ≤ unitário saída
- Validação de produtos selecionados a partir do BOM
- Verificação de inclusão de pelo menos um cálculo
- Tratamento de casos sem componentes importados

### SQL Personalizado para Saídas
- Cálculo de unitário de saída: `(total - ICMS) / quantidade`
- Filtro por UF para documentos interestaduais
- Consideração apenas de documentos principais (principal = 1)

---

**Última Alteração:** 01/12/2025 às 17:50  
**Autor:** Bruno  
**Tipo:** Fórmula de Cálculo de FCI  
**Versão:** 2.0  
**Observação:** O funcionamento dessa fórmula depende 100% do script na tarefa SRF2002 e do repositório de dados. Ver documentação no Trello.