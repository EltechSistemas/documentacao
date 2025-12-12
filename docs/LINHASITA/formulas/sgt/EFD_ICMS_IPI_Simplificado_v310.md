# EFD ICMS IPI - Simplificado v3.10

## 📖 Descrição
Fórmula para geração da Escrituração Fiscal Digital (EFD ICMS/IPI) no formato simplificado, conforme versão 3.10 do leiaute. O processo consolida todos os dados fiscais do período (mês/ano) e gera os registros dos blocos 0, B, C, D, E, G, H, K, 1 e 9 conforme as regras da Receita Federal.

## 🎯 Finalidade
Gerar o arquivo digital da EFD ICMS/IPI contendo:
- Identificação da empresa e períodos
- Documentos fiscais de entrada e saída
- Apurações de ICMS, IPI, ICMS-ST e DIFAL
- Controle de produção e estoque (CIAP)
- Inventário físico
- Outras informações complementares

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Faturamento

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Eaa0102` - Informações complementares dos documentos
- `Eaa01031` - Lançamentos fiscais
- `Edb01` - Apurações fiscais (ICMS, IPI, ST, DIFAL)
- `Edb0101` - Ajustes das apurações
- `Edb0102` - Obrigações a recolher
- `Ecc01` - Controle de CIAP (Crédito de ICMS do Ativo Permanente)
- `Bcb11` - Itens de inventário
- `Bcc01` - Movimentações de estoque
- `Aac10` - Empresa
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Itens (produtos/serviços)
- `Abb20` - Bens do ativo imobilizado
- Entre outras 50+ tabelas relacionadas

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtInicial | LocalDate | Sim | Data inicial do período (geralmente primeiro dia do mês) |
| dtFinal | LocalDate | Sim | Data final do período (geralmente último dia do mês) |
| arqSubstituto | Integer | Não | Indicador de arquivo substituto (0 ou 1) |
| dtInventario | LocalDate | Não | Data do inventário físico (último dia do mês) |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos dados da empresa (Aac10)
- Verificação das informações fiscais (Aac13)
- Definição do perfil (A, B ou C)
- Configuração dos alinhamentos (0050, 0030, 0033, 0032, 0031)
- Inicialização dos contadores de registros

### 2. **Geração do Bloco 0 - Abertura e Identificação**
- Registro 0000: Abertura do arquivo e identificação da empresa
- Registro 0001: Abertura do bloco 0
- Registro 0002: Classificação industrial (se aplicável)
- Registro 0005: Dados complementares da empresa
- Registro 0015: Dados do contribuinte substituto
- Registro 0100: Dados do contabilista
- Inicialização das estruturas para registros 0150, 0190, 0200, etc.

### 3. **Geração do Bloco B - Escrituração do ISS**
- Registro B001: Abertura do bloco B
- Registro B990: Encerramento do bloco B

### 4. **Geração do Bloco C - Documentos Fiscais I (ICMS/IPI)**
- **Registro C001**: Abertura do bloco C
- **Seção C100**: Notas fiscais (modelos 01, 1B, 04, 55)
  - Registros C101, C110-C115, C120, C130, C140-C141, C160
  - Registros C170-C179: Itens dos documentos
  - Registros C190, C195, C197: Análises e observações
- **Seção C300**: Notas fiscais de venda a consumidor (modelo 02)
  - Registros C310, C320-C321
- **Seção C350**: Notas fiscais de venda a consumidor (modelo 02 - individual)
  - Registros C370, C390
- **Seção C500**: Notas de energia, gás, água (modelos 06, 66, 28, 29)
  - Registro C590
- **Seção C800**: Cupons fiscais eletrônicos SAT (modelo 59)
  - Registros C850, C860, C890
- **Registro C990**: Encerramento do bloco C

### 5. **Geração do Bloco D - Documentos Fiscais II (Serviços - ICMS)**
- **Registro D001**: Abertura do bloco D
- **Seção D100**: Notas de serviço de transporte (modelos 07, 08, 09, etc.)
  - Registros D101, D190, D195, D197
- **Seção D500**: Notas de comunicação/telecomunicação (modelos 21, 22)
  - Registro D590
- **Registro D990**: Encerramento do bloco D

### 6. **Geração do Bloco E - Apuração do ICMS e IPI**
- **Registro E001**: Abertura do bloco E
- **Seção E100**: Período da apuração do ICMS
  - Registros E110-E116: Apuração, ajustes e obrigações
- **Seção E200**: Apuração do ICMS-ST
  - Registros E210, E220-E230, E250
- **Seção E300**: Apuração do DIFAL
  - Registros E310-E316
- **Seção E500**: Apuração do IPI (se industrial)
  - Registros E510, E520, E530
- **Registro E990**: Encerramento do bloco E

### 7. **Geração do Bloco G - CIAP (Crédito de ICMS do Ativo Permanente)**
- **Registro G001**: Abertura do bloco G
- **Registro G110**: Totais do CIAP
- **Registro G125**: Movimentação de bens
- **Registro G126**: Outros créditos
- **Registro G130**: Identificação dos documentos
- **Registro G140**: Itens dos documentos
- **Registro G990**: Encerramento do bloco G

### 8. **Geração do Bloco H - Inventário Físico**
- **Registro H001**: Abertura do bloco H
- **Registro H005**: Totais do inventário
- **Registro H010**: Itens do inventário
- **Registro H020**: Informações complementares
- **Registro H990**: Encerramento do bloco H

### 9. **Geração do Bloco K - Controle de Produção e Estoque**
- **Registro K001**: Abertura do bloco K
- **Registro K100**: Período de apuração
- **Registros K200-K302**: Estoque, produções, correções
- **Registro K990**: Encerramento do bloco K

### 10. **Geração do Bloco 1 - Outras Informações**
- **Registro 1001**: Abertura do bloco 1
- **Registro 1010**: Obrigatoriedade de registros
- **Registro 1100**: Informações sobre exportação
- **Registro 1400**: Informações sobre valores agregados
- **Registro 1990**: Encerramento do bloco 1

### 11. **Completamento do Bloco 0**
- Registros 0150-0175: Cadastro de participantes
- Registro 0190: Unidades de medida
- Registros 0200-0220: Identificação de itens
- Registros 0300-0305: Bens do CIAP
- Registros 0400, 0450, 0460: Tabelas auxiliares
- Registros 0500, 0600: Plano de contas e centros de custo
- **Registro 0990**: Encerramento do bloco 0

### 12. **Geração do Bloco 9 - Controle e Encerramento**
- **Registro 9001**: Abertura do bloco 9
- **Registro 9900**: Quantificação de registros por tipo
- **Registro 9990**: Encerramento do bloco 9
- **Registro 9999**: Encerramento do arquivo

## ⚠️ Regras de Negócio

### Perfis de Apuração
- **Perfil A**: Completo (todas as operações)
- **Perfil B**: Simplificado (algumas exclusões)
- **Perfil C**: Super simplificado (apenas essenciais)

### Filtros de Documentos
- Considera apenas documentos com `eaa01iEfdIcms = 1`
- Ignora documentos cancelados (situação 02, 03)
- Processa por data de entrada/saída conforme configuração

### Validações Obrigatórias
- Empresa deve ter município e endereço cadastrados
- Informações fiscais (Aac13) devem estar completas
- Apurações (Edb01) devem existir para o período
- Itens devem ter configurações fiscais (Abm0101)

### Tratamento de Campos
- Formatação de CNPJ/CPF (somente números)
- Inscrições estaduais "ISENTO" são convertidas para null
- Valores monetários com 2 casas decimais
- Datas no formato DDMMYYYY
- Série da nota limitada a 3 caracteres

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração da EFD.

### `gerarAberturaBloco0()`
Configura os registros iniciais do bloco 0.

### `gerarBlocoC()`, `gerarBlocoD()`, `gerarBlocoE()`, etc.
Métodos específicos para cada bloco da EFD.

### `buscarDocumentosPorModelo()`, `buscarDocumentosPorMovimentoModelo()`
Consultas para recuperar documentos por modelo e movimento.

### `comporRegistro0200()`, `comporRegistro0220()`
Montam os registros de identificação de itens.

### `gerarCodigoEntidade()`
Gera código único para entidades (considera múltiplas IEs).

### `gerarRegByPerfil()`
Define se um registro deve ser gerado conforme o perfil.

## 📊 Estrutura de Saída

**Arquivo de Texto:**
- Dois arquivos concatenados (txt1 e txt2)
- Formato delimitado por pipe (`|`)
- Codificação UTF-8
- Linhas no formato: `REG|CAMPO1|CAMPO2|...|CAMPOn`

**Conteúdo Principal:**
- Bloco 0: Identificação e cadastros
- Bloco C: Documentos de mercadorias
- Bloco D: Documentos de serviços
- Bloco E: Apurações
- Bloco G: CIAP
- Bloco H: Inventário
- Bloco K: Produção/estoque
- Bloco 1: Informações complementares
- Bloco 9: Controle

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data, strings, coleções
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema (100+ classes)
- `java.time` - Manipulação de datas
- `java.math` - Precisão decimal

**Módulo:** SGT (Sistema de Gestão Tributária)

## 📝 Observações Técnicas

### Performance
- Processamento paginado para grandes volumes
- Cache de entidades em memória
- Agrupamento de registros similares
- Uso de TableMap para estruturas dinâmicas

### Contadores
- Contagem precisa de cada tipo de registro
- Utilizado no registro 9900 para validação
- Atualizado incrementalmente durante o processamento

### Campos Dinâmicos (JSON)
- Valores fiscais armazenados em campos JSON (eaa01json, eaa0103json, etc.)
- Chaves configuráveis via alinhamentos
- Flexibilidade para diferentes configurações por empresa

### Validações
- Verificação de cancelamento do processo
- Atualização de status durante execução
- Validações de integridade dos dados

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de EFD ICMS/IPI  
**Versão:** 3.10  
**Leiaute:** Versão 16 (PVA 3.1.0)
