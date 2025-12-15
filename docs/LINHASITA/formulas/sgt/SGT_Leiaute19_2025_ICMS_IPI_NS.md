# Documentação da Fórmula: SGT_Leiaute19_2025_ICMS_IPI_NS

## Informações Gerais
- **Última Alteração**: 09/12/2025 às 08:20
- **Autor**: Bruno
- **Tipo de Fórmula**: SGT_EFD (Escrituração Fiscal Digital)
- **Versão do Leiaute**: 19

## Descrição
Esta fórmula gera o arquivo da Escrituração Fiscal Digital (EFD) conforme o leiaute 19 para apuração de ICMS e IPI. A fórmula processa documentos fiscais, inventários, CIAP (Controle de Crédito de ICMS do Ativo Permanente) e outras informações fiscais relevantes para o período especificado.

## Parâmetros de Entrada
| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `dtInicial` | LocalDate | Data inicial do período de apuração |
| `dtFinal` | LocalDate | Data final do período de apuração |
| `arqSubstituto` | Integer | Indicador de arquivo substituto (0=Original, 1=Substituto) |
| `dtInventario` | LocalDate | Data do inventário fiscal (opcional) |

## Estrutura da Fórmula

### 1. Inicialização e Configuração
- Validação dos dados da empresa cadastrada
- Verificação de informações fiscais obrigatórias
- Inicialização de contadores e estruturas de dados
- Criação dos arquivos de texto para geração da EFD

### 2. Blocos Gerados

#### **Bloco 0: Abertura, Identificação e Referências**
- **Registro 0000**: Abertura do arquivo digital e identificação da entidade
- **Registro 0001**: Abertura do Bloco 0
- **Registro 0002**: Classificação do estabelecimento industrial (se aplicável)
- **Registro 0005**: Dados complementares da entidade
- **Registro 0015**: Dados do contribuinte substituto
- **Registro 0100**: Dados do contabilista
- **Registros 0150/0175**: Tabela de cadastro do participante
- **Registro 0190**: Identificação das unidades de medida
- **Registros 0200-0221**: Tabela de identificação de itens
- **Registros 0300-0305**: Cadastro de bens do CIAP
- **Registro 0400**: Tabela de natureza da operação/prestação
- **Registro 0450**: Tabela de informação complementar
- **Registro 0460**: Tabela de observações do lançamento fiscal
- **Registro 0500**: Plano de contas contábeis
- **Registro 0600**: Centros de custo

#### **Bloco B: Escrituração e Apuração do ISS**
- **Registro B001**: Abertura do Bloco B
- **Registro B990**: Encerramento do Bloco B

#### **Bloco C: Documentos Fiscais I - Mercadorias (ICMS/IPI)**
- **Registro C001**: Abertura do Bloco C
- **Documentos do Tipo C100**: Nota Fiscal (01), Nota Fiscal Avulsa (1B), Nota Fiscal de Produtor (04) e NF-e (55)
- **Documentos do Tipo C300**: Resumo diário de NFC-e (02)
- **Documentos do Tipo C500**: NF de energia elétrica (06, 66), NF de fornecimento de água (29) e NF de gás (28)
- **Documentos do Tipo C800**: Cupom Fiscal Eletrônico - SAT (59)
- **Registro C990**: Encerramento do Bloco C

#### **Bloco D: Documentos Fiscais II - Serviços (ICMS)**
- **Registro D001**: Abertura do Bloco D
- **Documentos do Tipo D100**: Nota Fiscal de Serviço de Transporte (07) e Conhecimentos de Transporte
- **Documentos do Tipo D500**: Nota Fiscal de Serviço de Comunicação (21) e Telecomunicação (22)
- **Registro D990**: Encerramento do Bloco D

#### **Bloco E: Apuração do ICMS e do IPI**
- **Registro E001**: Abertura do Bloco E
- **Registro E100**: Período da apuração do ICMS
- **Registro E110**: Apuração do ICMS - Operações Próprias
- **Registros E111-E116**: Ajustes, informações adicionais e obrigações do ICMS
- **Registros E200-E250**: Apuração do ICMS - Substituição Tributária
- **Registros E300-E316**: Apuração do ICMS Diferencial de Alíquota
- **Registros E500-E531**: Apuração do IPI (para empresas industriais)
- **Registro E990**: Encerramento do Bloco E

#### **Bloco G: CIAP (Controle de Crédito de ICMS do Ativo Permanente)**
- **Registro G001**: Abertura do Bloco G
- **Registro G110**: ICMS - Ativo Permanente - CIAP
- **Registros G125-G140**: Movimentação de bens do ativo imobilizado
- **Registro G990**: Encerramento do Bloco G

#### **Bloco H: Inventário Físico**
- **Registro H001**: Abertura do Bloco H
- **Registro H005**: Totais do inventário
- **Registros H010-H020**: Itens do inventário
- **Registro H990**: Encerramento do Bloco H

#### **Bloco K: Controle da Produção e Estoque**
- **Registro K001**: Abertura do Bloco K
- **Registro K010**: Tipo de leiaute
- **Registro K100**: Período de apuração
- **Registros K200-K302**: Controle de estoque e produção
- **Registro K990**: Encerramento do Bloco K

#### **Bloco 1: Outras Informações**
- **Registro 1001**: Abertura do Bloco 1
- **Registro 1010**: Obrigatoriedade de registros
- **Registros 1100-1105**: Informações sobre exportação
- **Registro 1400**: Informações sobre valores agregados
- **Registro 1601**: Operações com instrumentos de pagamentos eletrônicos
- **Registro 1990**: Encerramento do Bloco 1

#### **Bloco 9: Controle e Encerramento**
- **Registro 9001**: Abertura do Bloco 9
- **Registro 9900**: Quantificação de registros
- **Registro 9990**: Encerramento do Bloco 9
- **Registro 9999**: Encerramento do arquivo digital

## Funcionalidades Principais

### 1. **Processamento de Documentos Fiscais**
- Processa documentos de entrada e saída
- Aplica validações específicas por perfil (A, B ou C)
- Gera registros detalhados por item (C170, C370, etc.)
- Trata documentos cancelados e ajustes

### 2. **Apuração de Tributos**
- Calcula ICMS próprio e substituição tributária
- Processa apuração de IPI para indústrias
- Calcula diferencial de alíquota (EC 87/15)
- Gera obrigações a recolher

### 3. **Controle de CIAP**
- Gerencia créditos de ICMS do ativo permanente
- Calcula apropriação de créditos
- Processa baixa e alienação de bens

### 4. **Inventário Físico**
- Gera registros de inventário quando aplicável
- Calcula valores totais do estoque
- Relaciona itens com suas respectivas contas contábeis

### 5. **Controle de Produção e Estoque**
- Registra movimentações internas
- Controla produção própria e por terceiros
- Gerencia correções de apontamento

## Validações e Tratamentos Especiais

### Validações Obrigatórias
1. Empresa com município cadastrado
2. Informações fiscais completas
3. Perfil da empresa definido
4. Situação dos documentos informada
5. Configurações dos itens para a empresa

### Tratamento por Perfil
- **Perfil A**: Gera todos os registros
- **Perfil B**: Restrições específicas para alguns registros
- **Perfil C**: Registros mais limitados

### Tratamento de Modelos de Documentos
- Modelos específicos para cada bloco (C100, C300, etc.)
- Formatação de séries conforme modelo
- Validações específicas por tipo de documento

## Estruturas de Dados Utilizadas

### Mapas e Conjuntos Principais
- `entidades`: Lista de participantes para registro 0150
- `abe01s`: Entidades para processamento
- `set0190`: Unidades de medida utilizadas
- `set0200`: Itens processados
- `map0220`: Fatores de conversão de unidades
- `map0221`: Correlação entre códigos de itens
- `abc10s`: Contas contábeis utilizadas
- `map0400`: Naturezas de operação
- `map0450`: Informações complementares
- `map0460`: Observações do lançamento fiscal

### Contadores
- Contagem individual por tipo de registro
- Contagem por bloco para encerramento
- Totalização geral para registro 9999

## Métodos Auxiliares

### Busca de Dados
- `buscarDocumentosPorModelo()`: Busca documentos por modelos específicos
- `buscarDocumentosPorMovimentoModelo()`: Busca por movimento e modelo
- `buscarApuracoes()`: Busca apurações de tributos
- `buscarCIAPPorPeriodoOuBaixados()`: Busca fichas CIAP

### Processamento
- `gerarRegByPerfil()`: Define se registro deve ser gerado conforme perfil
- `comporRegistro0200()`: Monta registro de identificação de item
- `comporRegistro0220()`: Monta fatores de conversão
- `comporRegistro0221()`: Monta correlação entre itens

### Formatação
- `formatarValor()`: Formata valores numéricos
- `formatarSerie()`: Formata série do documento
- `inscrEstadual()`: Formata inscrição estadual
- `gerarCodigoEntidade()`: Gera código único para entidade

## Dependências e Imports
A fórmula utiliza diversas classes do sistema SAM, incluindo:
- Entidades de modelos (Eaa01, Abb01, Abm01, etc.)
- Utilitários (StringUtils, DateUtils, ValidacaoException)
- Bibliotecas Java (LocalDate, BigDecimal, HashMap, etc.)

## Considerações Técnicas

### Performance
- Processamento paginado de documentos para evitar estouro de memória
- Uso de estruturas de dados otimizadas (HashMap, HashSet)
- Limpeza de referências para coleta de lixo

### Manutenibilidade
- Métodos organizados por funcionalidade
- Comentários em pontos críticos
- Separação clara entre lógica de negócio e formatação

### Tratamento de Erros
- Validações prévias com mensagens claras
- Tratamento de nulos em campos opcionais
- Continuação do processamento após erros não-críticos

## Observações Finais
Esta fórmula é essencial para a geração da EFD-ICMS/IPI e deve ser executada mensalmente para cumprimento das obrigações fiscais. A correta configuração dos parâmetros e a validação dos dados de entrada são cruciais para a geração de um arquivo válido.
