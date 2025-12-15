# SGT - EFD_Leiaute15_2021

## 📖 Descrição
Fórmula para geração do arquivo digital da Escrituração Fiscal Digital (EFD) das Contribuições (PIS/COFINS), conforme Leiaute 15/2021, com suporte a múltiplos blocos e registros exigidos pela legislação fiscal.

## 🎯 Finalidade
Gerar arquivo texto no formato EFD Contribuições (PIS/COFINS) contendo todas as operações fiscais do período, incluindo documentos fiscais, operações diversas, créditos, débitos e apurações.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Compliance Tributário

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal
- `Eaa0102` - Complemento do documento
- `Eaa01034` - Declarações de importação
- `Eaa0113` - Pagamentos do documento
- `Abb01` - Lançamentos contábeis
- `Aah01` - Tipos de documento
- `Aac10` - Empresas (matriz e filiais)
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Itens (produtos/serviços)
- `Edb10` - Período de apuração
- `Edb11` - Operações geradoras de crédito
- `Edb12` - Retenções na fonte
- `Edb13` - Deduções diversas
- `Edb14` - Créditos por incorporação/fusão/cisão

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtInicial | LocalDate | Sim | Data inicial do período |
| dtFinal | LocalDate | Sim | Data final do período |
| situacao | Integer | Sim | Situação da EFD (0-normal, 9-retificadora) |
| numRecibo | String | Não | Número do recibo (para retificação) |
| aliqPisF150 | BigDecimal | Sim | Alíquota do PIS para registro F150 |
| aliqCofinsF150 | BigDecimal | Sim | Alíquota da COFINS para registro F150 |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Busca da empresa matriz e filiais
- Verificação da apuração de receita do período
- Inicialização dos contadores e arquivos texto

### 2. **Geração do Bloco 0 - Abertura e Identificação**
- Registro 0000: Abertura do arquivo
- Registro 0001: Abertura do bloco 0
- Registro 0100: Dados do contabilista
- Registro 0110: Regimes de apuração
- Registro 0111: Receita bruta mensal para rateio

### 3. **Geração dos Blocos de Documentos Fiscais**
- **Bloco A**: Documentos fiscais de serviços
- **Bloco C**: Documentos fiscais de mercadorias (ICMS/IPI)
- **Bloco D**: Documentos fiscais de serviços (ICMS)

### 4. **Geração dos Blocos de Operações Diversas**
- **Bloco F**: Demais documentos e operações
  - F100: Operações geradoras de contribuição e créditos
  - F120: Bens que geram créditos por depreciação
  - F130: Bens que geram créditos por aquisição
  - F150: Crédito presumido sobre estoque
  - F550: Consolidação para lucro presumido
  - F600: Contribuição retida na fonte
  - F700: Deduções diversas
  - F800: Créditos por incorporação/fusão/cisão

### 5. **Geração dos Blocos Complementares**
- **Bloco M**: Apuração do PIS/COFINS
- **Bloco 1**: Operações extemporâneas
  - 1010: Processos judiciais referenciados
  - 1020: Processos administrativos referenciados
  - 1900: Consolidação documentos PJ lucro presumido

### 6. **Finalização do Bloco 0**
- Registros 0140-0600: Tabelas cadastrais
- Registro 0990: Encerramento do bloco 0

### 7. **Geração do Bloco 9 - Controle e Encerramento**
- Registro 9001: Abertura do bloco 9
- Registro 9900: Quantitativo de registros por tipo
- Registro 9990: Encerramento do bloco 9
- Registro 9999: Encerramento do arquivo

## ⚠️ Regras de Negócio

### Classificação de Documentos
- **Bloco A**: Notas fiscais de serviço (modelos específicos)
- **Bloco C**: Notas fiscais de produtos (01, 1B, 04, 55, 65)
- **Bloco D**: Documentos de transporte (07, 08, 09, 10, 11, 26, 27, 57) e comunicação (21, 22)

### Situação de Documentos
- **Situações especiais (02, 03, 04, 05)**: Geram apenas registros pai sem filhos
- **Documentos normais**: Geram registros completos com todos os campos

### Cálculos Fiscais
- **Base de cálculo**: Valores extraídos dos campos JSON dos documentos
- **Alíquotas**: Configuráveis por item/operação
- **CST PIS/COFINS**: Determinadas conforme movimento e classificação

### Validações
- Documentos devem ter situação definida
- Apuração de receita deve existir para o período
- Empresa matriz deve ter informações fiscais cadastradas

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração da EFD.

### Métodos de Geração por Bloco
- `gerarAberturaBloco0()` - Bloco 0 inicial
- `gerarBlocoA()` - Documentos de serviços
- `gerarBlocoC()` - Documentos de mercadorias
- `gerarBlocoD()` - Documentos de serviços (ICMS)
- `gerarBlocoF()` - Operações diversas
- `gerarBlocoM()` - Apuração
- `gerarBloco1()` - Operações extemporâneas
- `gerarFechamentoBloco0()` - Tabelas cadastrais
- `gerarBloco9()` - Controle e encerramento

### Métodos de Suporte
- `formatarValor()` - Formatação de valores monetários
- `formatarSerie()` - Formatação de série do documento
- `selecionarCSTPis()` - Determinação do CST do PIS
- `selecionarCSTCofins()` - Determinação do CST da COFINS
- `comporRegistro0150()` - Composição da tabela de participantes
- `comporRegistro0200()` - Composição da tabela de itens

## 📊 Estrutura de Saída

**Arquivo Texto:**
- Formato delimitado por pipe (`|`)
- Codificação UTF-8
- Layout conforme Leiaute 15/2021
- Dividido em duas partes (txt1 e txt2) para organização

**Conteúdo Gerado:**
- ~150 tipos de registros diferentes
- Tabelas cadastrais (participantes, itens, unidades, etc.)
- Documentos fiscais do período
- Operações diversas e créditos
- Controles quantitativos

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data, texto e arquivos
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Versão do Leiaute:** 6
**Alinhamento Contábil:** 0051

## 📝 Observações Técnicas

### Performance
- Processamento paginado para grandes volumes
- Uso de `TableMap` para manipulação eficiente
- Agrupamento no banco de dados quando possível

### Tratamento de Dados
- Campos JSON para valores calculados
- Formatação rigorosa conforme layout oficial
- Validações de consistência durante a geração

### Contadores
- Contagem precisa de registros por tipo
- Atualização em tempo real durante a geração
- Utilizados no registro 9900 (quantitativos)

### Fluxo de Controle
- Verificação periódica de cancelamento
- Atualização de status do processo
- Tratamento de exceções com mensagens claras

### Configurações Empresariais
- Suporte a matriz e filiais
- Grupos centralizadores por tabela (EA, EC, ED)
- Informações fiscais por empresa

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Geração de EFD  
**Versão:** 1.0