# SGT - Leiaute 15/2021 Contribuições (EFD Contribuições)

## 📖 Descrição
Fórmula para geração do arquivo EFD Contribuições (PIS/COFINS) no formato Leiaute 15/2021, contemplando todos os blocos obrigatórios para escrituração fiscal digital.

## 🎯 Finalidade
Gerar arquivo digital da EFD Contribuições conforme legislação vigente, consolidando informações de documentos fiscais, apurações, créditos e demais operações sujeitas à tributação PIS/COFINS.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Escrituração Fiscal Digital
- Auditoria Fiscal

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens de documentos fiscais
- `Abb01` - Cabeçalho de documentos
- `Aah01` - Tipos de documento
- `Abe01` - Entidades
- `Abm01` - Itens (produtos/serviços)
- `Edb10` - Período de apuração
- `Edb11` - Operações de crédito
- `Edb12` - Retenções na fonte
- `Edb13` - Deduções diversas
- `Edb14` - Créditos por incorporação/fusão/cisão
- `Ecb01` - Bens do ativo imobilizado

**Blocos Gerados:**
- **Bloco 0** - Abertura, identificação e referências
- **Bloco A** - Documentos fiscais - serviços
- **Bloco C** - Documentos fiscais I - mercadorias (ICMS/IPI)
- **Bloco D** - Documentos fiscais II - serviços (ICMS)
- **Bloco F** - Demais documentos e operações
- **Bloco M** - Apuração do PIS/COFINS
- **Bloco 1** - Operações extemporâneas
- **Bloco 9** - Controle e encerramento

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtInicial | LocalDate | Sim | Data inicial do período |
| dtFinal | LocalDate | Sim | Data final do período |
| situacao | Integer | Sim | Situação da escrituração (0-normal, 9-retificadora) |
| numRecibo | String | Não | Número do recibo (para retificação) |
| aliqPisF150 | BigDecimal | Não | Alíquota PIS para registro F150 |
| aliqCofinsF150 | BigDecimal | Não | Alíquota COFINS para registro F150 |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação de parâmetros e período
- Busca da empresa matriz e filiais
- Verificação da apuração de receita do período
- Seleção do alinhamento EFD (0051)

### 2. **Geração dos Blocos**
- **Bloco 0**: Dados da empresa, contabilista, regimes de apuração
- **Bloco A**: Notas fiscais de serviço
- **Bloco C**: Notas fiscais de mercadorias
- **Bloco D**: Documentos de transporte e comunicação
- **Bloco F**: Operações geradoras de créditos, bens do ativo, ajustes
- **Bloco M**: Apuração consolidada
- **Bloco 1**: Processos judiciais e administrativos
- **Bloco 9**: Encerramento e totais

### 3. **Processamento por Empresa**
- Consolidação por grupo centralizador (GC)
- Validação de documentos fiscais
- Cálculo de bases e valores do PIS/COFINS
- Agrupamento por CST e atividade econômica

### 4. **Formatação e Saída**
- Geração do arquivo texto no formato delimitado por "|"
- Formatação de valores, datas e códigos
- Controle de quantidade de linhas por registro
- Validação final do arquivo

## ⚠️ Regras de Negócio

### Validações de Documentos
- Apenas documentos com indicador EFD Contribuinte = 1
- Documentos não cancelados
- Período conforme datas informadas
- Situação do documento obrigatória

### Classificação de Receitas
- **Tributada**: CST 01-03, 05
- **Não-Tributada**: CST 04, 06-09, 49
- **Exportação**: CST específicos
- **Cumulativa**: Regime cumulativo

### Cálculos Financeiros
- **Base de Cálculo**: Conforme CST e movimento
- **Alíquotas**: Específicas por atividade/documento
- **Créditos**: Baseados em depreciação ou aquisição
- **Ajustes**: Conforme legislação específica

### Formatação de Dados
- **Datas**: Formato "ddMMyyyy"
- **Valores**: 2 casas decimais
- **CNPJ/CPF**: Apenas números (14/11 dígitos)
- **Códigos**: Preenchimento com zeros à esquerda

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração da EFD.

### Geração por Bloco
- `gerarAberturaBloco0()` - Dados iniciais da empresa
- `gerarBlocoA()` - Documentos de serviços
- `gerarBlocoC()` - Documentos de mercadorias
- `gerarBlocoD()` - Documentos de transporte/comunicação
- `gerarBlocoF()` - Operações diversas e créditos
- `gerarBlocoM()` - Apuração do PIS/COFINS
- `gerarBloco1()` - Processos extemporâneos
- `gerarBloco9()` - Encerramento

### Métodos Auxiliares
- `buscarDocumentosXXX()` - Consultas específicas por bloco
- `comporRegistroXXX()` - Montagem de registros
- `formatarValor()` - Formatação de valores monetários
- `selecionarCSTPis/Cofins()` - Definição de CST

## 📊 Estrutura de Saída

**Arquivo Texto:**
- Formato delimitado por pipe ("|")
- Codificação UTF-8
- Registros conforme layout oficial
- Blocos separados com totais

**Principais Registros:**
- **0000/0001**: Abertura do arquivo e bloco
- **0100**: Dados do contabilista
- **0140**: Estabelecimentos
- **0150**: Participantes
- **0200**: Produtos/serviços
- **A100/C100/D100**: Documentos fiscais
- **F100/F120/F130**: Operações de crédito
- **F550**: Consolidação para lucro presumido
- **9999**: Encerramento do arquivo

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários, datas e formatação
- `multiorm` - Criteria e consultas ao banco
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Configurações:**
- Alinhamento EFD (0051)
- Versão do leiaute (6)
- Modelos de documentos fiscais por bloco

## 📝 Observações Técnicas

### Otimizações
- Consultas paginadas para grandes volumes
- Cache de entidades frequentemente acessadas
- Agrupamento por chaves para evitar duplicidades
- Processamento por lotes de documentos

### Tratamento de Erros
- Validação de dados obrigatórios
- Tratamento de valores nulos
- Log de documentos com inconsistências
- Interrupção por cancelamento do processo

### Performance
- Uso de TableMap para agrupamentos
- Consultas específicas por bloco
- Processamento em memória com limites
- Controle de quantidade de registros

---

**Última Alteração:** 22/10/2025 às 17:15  
**Autor:** Bruno  
**Tipo:** Fórmula EFD Contribuições  
**Versão:** 1.0  
**Leiaute:** 15/2021