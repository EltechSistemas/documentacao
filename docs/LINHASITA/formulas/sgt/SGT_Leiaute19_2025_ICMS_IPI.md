# SGT_Leiaute19_2025_ICMS_IPI

## 📖 Descrição
Fórmula para geração do arquivo digital da EFD (Escrituração Fiscal Digital) - Blocos 0, B, C, D, E, G, H, K e 1, conforme Leiaute 19/2025 da SEFAZ. Responsável por consolidar informações fiscais de ICMS e IPI para envio ao SPED Fiscal.

## 🎯 Finalidade
Gerar arquivo digital da EFD contendo todas as operações fiscais (entradas, saídas, serviços, apurações, inventários, CIAP, produção/estoque) de um período determinado, atendendo às exigências do SPED Fiscal para empresas industriais e comerciais.

## 👥 Público-Alvo
- Departamento Fiscal
- Controladoria
- Contabilidade
- Desenvolvedores de fórmulas do sistema

## 📊 Dados e Fontes
**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos
- `Eaa0102` - Dados gerais dos documentos
- `Eaa0101` - Endereços dos documentos
- `Abb01` - Central de documentos
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Itens/produtos
- `Aac10` - Empresa
- `Edb01` - Apurações fiscais
- `Ecc01` - CIAP (Controle de Crédito de ICMS do Ativo Permanente)
- `Bcb10/Bcb11` - Inventários
- `Bcc01` - Controle de produção/estoque

**Entidades Envolvidas:**
- `Eaa01` - Documento fiscal
- `Eaa0103` - Item do documento
- `Abb01` - Central do documento
- `Abe01` - Entidade participante
- `Abm01` - Item/produto
- `Aac10` - Empresa
- `Aac13` - Dados fiscais da empresa
- `Edb01` - Apuração fiscal
- `Ecc01` - Ficha CIAP

## ⚙️ Parâmetros da Fórmula
1. **dtInicial** (Date): Data inicial do período de apuração
2. **dtFinal** (Date): Data final do período de apuração
3. **arqSubstituto** (Integer): Indicador de arquivo substituto (0=Original, 1=Substituto)
4. **dtInventario** (Date): Data do inventário (opcional)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração da EFD:
1. **Validação de dados** da empresa e período
2. **Seleção de alinhamentos** (0050, 0030, 0033, 0032, 0031)
3. **Inicialização de arquivos** e contadores
4. **Geração dos blocos** na ordem correta do SPED
5. **Consolidação do arquivo final**

### `gerarAberturaBloco0()`
Configura os registros iniciais do arquivo EFD:
- **Registro 0000**: Abertura e identificação da entidade
- **Registro 0001**: Abertura do Bloco 0
- **Registro 0005**: Dados complementares da empresa
- **Registro 0100**: Dados do contabilista
- **Inicialização** de estruturas de dados para registros posteriores

### `gerarBlocoC()`
Gera o Bloco C - Documentos Fiscais I - Mercadorias (ICMS/IPI):
- **Registro C100**: Notas fiscais (01, 1B, 04, 55)
- **Registro C300/C350**: NFC-e e cupons fiscais
- **Registro C500**: NF de energia elétrica, água, gás
- **Registro C800**: Cupom Fiscal Eletrônico (SAT)
- **Registros analíticos** correspondentes a cada documento

### `gerarBlocoD()`
Gera o Bloco D - Documentos Fiscais II - Serviços (ICMS):
- **Registro D100**: Conhecimentos de transporte
- **Registro D500**: Notas fiscais de serviço
- **Registro D700**: Notas fiscais de serviço eletrônica

### `gerarBlocoE()`
Gera o Bloco E - Apuração do ICMS e do IPI:
- **Registro E100/E110**: Apuração do ICMS próprio
- **Registro E200/E210**: Apuração do ICMS ST
- **Registro E300/E310**: Apuração do diferencial de alíquota
- **Registro E500/E520**: Apuração do IPI

### `gerarBlocoG()`
Gera o Bloco G - CIAP (Controle de Crédito de ICMS do Ativo Permanente):
- **Registro G110**: Totais do CIAP
- **Registro G125**: Movimentação de bens
- **Registro G130/G140**: Identificação de documentos fiscais relacionados

### `gerarBlocoH()`
Gera o Bloco H - Inventário Físico:
- **Registro H005**: Totais do inventário
- **Registro H010**: Itens do inventário
- **Registro H020**: Informações complementares do inventário

### `gerarBlocoK()`
Gera o Bloco K - Controle da Produção e Estoque:
- **Registro K100**: Período de apuração
- **Registro K200**: Estoque escriturado
- **Registros K220-K302**: Movimentações de produção

### `gerarBloco1()`
Gera o Bloco 1 - Outras Informações:
- **Registro 1100**: Informações sobre exportação
- **Registro 1400**: Informações sobre valores agregados
- **Registro 1601**: Operações com instrumentos de pagamento eletrônico

### `gerarFechamentoBloco0()`
Completa o Bloco 0 com registros que dependem do processamento dos outros blocos:
- **Registro 0150**: Cadastro de participantes
- **Registro 0200**: Cadastro de itens
- **Registro 0300**: Cadastro de bens do CIAP
- **Registros 0400-0600**: Tabelas auxiliares

### `gerarBloco9()`
Gera o Bloco 9 - Controle e Encerramento:
- **Registro 9001**: Abertura do Bloco 9
- **Registro 9900**: Controle de registros
- **Registro 9999**: Encerramento do arquivo

## 📝 Fluxo de Execução

### 1. **Inicialização e Validação**
- Carrega dados da empresa e valida configurações fiscais
- Verifica período informado e parâmetros obrigatórios
- Inicializa estruturas de dados e arquivos de saída

### 2. **Geração dos Blocos em Ordem Estruturada**
1. **Bloco 0 (Abertura)**: Registros iniciais
2. **Bloco B**: Escrituração do ISS (se aplicável)
3. **Bloco C**: Documentos fiscais de mercadorias
4. **Bloco D**: Documentos fiscais de serviços
5. **Bloco E**: Apurações de ICMS e IPI
6. **Bloco G**: CIAP
7. **Bloco H**: Inventário
8. **Bloco K**: Produção e estoque
9. **Bloco 1**: Outras informações
10. **Bloco 0 (Complemento)**: Registros dependentes
11. **Bloco 9**: Controle e encerramento

### 3. **Processamento de Documentos Fiscais**
- Busca documentos por modelo e período
- Processa cada documento e seus itens
- Gera registros principais e analíticos
- Atualiza estruturas de dados para registros posteriores

### 4. **Processamento de Apurações**
- Busca apurações de ICMS, ICMS-ST, Diferencial e IPI
- Calcula totais e gera registros de apuração
- Inclui ajustes e obrigações a recolher

### 5. **Finalização**
- Completa registros que dependem de dados consolidados
- Gera registros de controle e totais
- Fecha arquivo com registro 9999

## ⚠️ Regras de Negócio

### Validações Críticas
1. **Empresa**: Deve ter município, endereço e informações fiscais configuradas
2. **Período**: Data final não pode ser anterior à data inicial
3. **Documentos**: Devem ter situação fiscal definida para processamento
4. **Itens**: Devem ter configuração fiscal cadastrada para a empresa
5. **Apurações**: Devem existir para o período informado (ICMS obrigatório, IPI se industrial)

### Regras de Perfil (A, B, C)
- **Perfil A**: Industrial/equiparado, gera todos os registros
- **Perfil B**: Comércio, restringe alguns registros
- **Perfil C**: Outros, restringe mais registros
- A geração de cada registro depende do perfil e tipo de operação

### Tratamento Específico por Modelo de Documento
- **Modelo 01, 1B, 04, 55**: Bloco C, registros completos
- **Modelo 02**: NFC-e, registros simplificados
- **Modelo 06, 28, 29, 66**: Energia, água, gás
- **Modelo 07-11, 26-27, 57, 67**: Transportes
- **Modelo 21-22, 62**: Serviços diversos

### Cálculos Especiais
1. **CIAP**: Cálculo de apropriação de crédito de ICMS
2. **Inventário**: Valoração e classificação de itens
3. **Produção**: Controle de entrada/saída de estoque
4. **Exportação**: Tratamento específico para operações de exportação

## 🔄 Dependências

**Classes:**
- `FormulaBase` - Classe base para fórmulas do sistema
- Todas as entidades do modelo SAM mencionadas na seção "Entidades Envolvidas"

**Bibliotecas:**
- `br.com.multiorm` - ORM e consultas ao banco
- `br.com.multitec.utils` - Utilitários diversos
- `sam.dicdados.FormulaTipo` - Tipos de fórmula
- `sam.server.samdev.utils.Parametro` - Parâmetros de consulta

## 🎨 Saída da Fórmula
A fórmula gera um arquivo texto no formato delimitado por pipe (|) conforme layout do SPED EFD:

### Arquivo de Saída
- **Formato**: Texto com delimitador "|"
- **Codificação**: UTF-8
- **Estrutura**: Blocos e registros conforme manual do SPED
- **Conteúdo**: Todas as operações fiscais do período

### Campos Gerados (Exemplos)
- **Bloco 0**: Identificação da empresa, participantes, itens, bens
- **Bloco C/D**: Documentos fiscais com valores de impostos
- **Bloco E**: Apurações de impostos
- **Bloco G/H/K**: Controles específicos
- **Bloco 9**: Totais e controle do arquivo

## 📌 Observações Técnicas

### Arquitetura
- Fórmula extensa com aproximadamente 3000 linhas de código
- Processamento em lotes (paginado) para evitar estouro de memória
- Uso intensivo de `TableMap` para manipulação de dados
- Separação de arquivos txt1 e txt2 para organização do fluxo

### Performance
- Consultas paginadas para documentos fiscais
- Cache de entidades frequentemente acessadas
- Processamento em memória com estruturas otimizadas
- Validações para evitar processamento desnecessário

### Manutenibilidade
- Métodos organizados por blocos do SPED
- Constantes para códigos fixos
- Funções auxiliares para cálculos repetitivos
- Comentários indicando alterações importantes

### Metadados
- Código identificado por metadados no final do arquivo: `meta-sis-eyJ0aXBvIjoiZm9ybXVsYSIsImZvcm11bGF0aXBvIjoiMDYifQ==`
- Tipo de fórmula: `FormulaTipo.SGT_EFD`
- Última alteração: 15/12/2025 11:16 por NAGYLA

## 🔧 Configurações Necessárias

### Pré-requisitos do Sistema
1. **Cadastro Completo da Empresa** com dados fiscais (perfil, atividade, IE)
2. **Documentos Fiscais** lançados e processados no período
3. **Apurações Fiscais** realizadas para o período
4. **Cadastro de Itens** com configurações fiscais e comerciais
5. **CIAP** configurado para empresas industriais
6. **Inventário** lançado se necessário

### Configurações Específicas
1. **Alinhamentos**: Configurações de campos JSON para cada registro
2. **Perfil da Empresa**: A, B ou C conforme atividade
3. **Campos JSON**: Estrutura de campos personalizados para cálculos
4. **Unidades de Medida**: Cadastro completo para conversões

## ⚠️ Considerações de Implementação

### Complexidade Fiscal
- Implementa layout complexo do SPED EFD
- Trata múltiplos modelos de documentos fiscais
- Considera diferentes perfis de empresa
- Atende legislação fiscal brasileira atual

### Testes
- Necessário testar com diferentes perfis de empresa
- Validar com períodos com/sem inventário
- Testar cenários com/sem operações de exportação
- Verificar cálculos de apuração

### Monitoramento
- Logs de processamento por documento
- Controle de progresso durante execução
- Validações de consistência de dados
- Tratamento de exceções específicas

### Atualizações
- Acompanhamento de mudanças no layout do SPED
- Atualização de códigos e alíquotas
- Manutenção das regras por perfil
- Adaptação para novos modelos de documentos

## 🎨 Estrutura do Código
- **Package**: `linhasita.formulas.sgt`
- **Imports**: Extensos, cobrindo ORM, utilitários, entidades SAM
- **Constantes**: Versão do leiaute, códigos de alinhamento, modelos de documentos
- **Variáveis**: Contadores por registro, estruturas de cache, arquivos de saída
- **Métodos**: Organizados por blocos do SPED com responsabilidades específicas

## 🔧 Métodos Auxiliares
- `buscarDocumentosPorModelo()`: Consulta documentos por modelo
- `formatarValor()`: Formata valores monetários
- `gerarCodigoEntidade()`: Gera código único para participantes
- `comporRegistro0200()`: Prepara cadastro de itens
- `gerarRegByPerfil()`: Define se registro deve ser gerado conforme perfil

## 📊 Contadores
A fórmula mantém contadores detalhados por registro para:
- Controle interno do processamento
- Geração do registro 9900 (controle de registros)
- Validação do total de linhas do arquivo

## ⚠️ Limitações Conhecidas
- Processamento pode ser lento para períodos muito extensos
- Consumo de memória em períodos com muitos documentos
- Dependência de estrutura de JSON fields configurada
- Necessidade de apurações prévias para geração do bloco E