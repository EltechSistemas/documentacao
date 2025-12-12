# SGT - Leiaute 19/2025 (EFD ICMS/IPI)

## 📖 Descrição
Fórmula para geração do arquivo **EFD-ICMS/IPI** (Escrituração Fiscal Digital) conforme Leiaute versão 19 (2025), contendo registros dos blocos 0, B, C, D, E, G, H, K, 1 e 9, com suporte a múltiplos modelos de documentos fiscais e apurações de ICMS, IPI, ST, DIFAL, CIAP, inventário e controle de produção.

## 🎯 Finalidade
Gerar arquivo digital no formato exigido pela Receita Federal (EFD-ICMS/IPI) para entrega obrigatória, consolidando operações fiscais, apurações, inventário, CIAP e movimentações de estoque do período.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Faturamento
- Controladoria

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` – Documentos fiscais
- `Eaa0102` – Dados complementares do documento fiscal
- `Eaa0103` – Itens do documento fiscal
- `Eaa01031` – Lançamentos fiscais
- `Eaa01034` – Declarações de importação
- `Abb01` – Central de documentos
- `Aah01` – Tipo de documento fiscal
- `Abe01` – Entidades (clientes/fornecedores)
- `Abm01` – Itens (produtos/serviços)
- `Aac10` – Empresa
- `Edb01` – Apurações fiscais
- `Ecc01` – Fichas do CIAP
- `Bcb10` / `Bcb11` – Inventário físico
- `Bcc01` – Movimentações de estoque/produção

## ⚙️ Parâmetros da Fórmula

| Parâmetro        | Tipo        | Obrigatório | Descrição                                      |
|------------------|-------------|-------------|------------------------------------------------|
| dtInicial        | LocalDate   | Sim         | Data inicial do período de apuração            |
| dtFinal          | LocalDate   | Sim         | Data final do período de apuração              |
| arqSubstituto    | Integer     | Sim         | Indicador de arquivo substituto (0 = Não, 1 = Sim) |
| dtInventario     | LocalDate   | Não         | Data do inventário físico (opcional)           |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento dos dados da empresa (Aac10)
- Verificação das informações fiscais e perfil da empresa
- Seleção de alinhamentos (EFD, ICMS, ST, IPI, DIFAL)

### 2. **Geração dos Blocos da EFD**
- **Bloco 0** – Abertura, identificação e referências
- **Bloco B** – Escrituração e apuração do ISS
- **Bloco C** – Documentos fiscais I – Mercadorias (ICMS/IPI)
- **Bloco D** – Documentos fiscais II – Serviços (ICMS)
- **Bloco E** – Apuração do ICMS e do IPI
- **Bloco G** – CIAP (Controle de Crédito de ICMS do Ativo Permanente)
- **Bloco H** – Inventário Físico
- **Bloco K** – Controle da Produção e Estoque
- **Bloco 1** – Outras Informações
- **Bloco 9** – Controle e encerramento do arquivo digital

### 3. **Consolidação e Validação**
- Contagem de linhas por registro
- Geração dos registros totais (0990, C990, D990, E990, etc.)
- Montagem final do arquivo texto com delimitador "|"

## ⚠️ Regras de Negócio

### Documentos Fiscais (Bloco C e D)
- Filtragem por modelo (01, 1B, 04, 55, 02, 06, 59, etc.)
- Tratamento por situação do documento (cancelado, regular, etc.)
- Cálculo de valores (ICMS, IPI, PIS, COFINS, descontos, frete)
- Geração de registros analíticos (C170, C190, C590, etc.)

### Apurações Fiscais (Bloco E)
- Busca das apurações de ICMS (011), ST (012), IPI (013) e DIFAL (014)
- Cálculo de débitos, créditos, ajustes e saldos
- Geração de obrigações a recolher (E116, E250, E316)

### CIAP (Bloco G)
- Identificação de bens do ativo imobilizado
- Cálculo de apropriação de créditos de ICMS
- Movimentações (entrada, baixa, alienação, etc.)

### Inventário (Bloco H)
- Geração a partir de data de inventário informada
- Valores unitários, totais e classificação (próprio/terceiros)

### Controle de Produção (Bloco K)
- Registro de produção própria, terceiros, correções e inventário
- Movimentações internas entre itens (K220)

### Outras Informações (Bloco 1)
- Exportação (1100, 1105)
- Valores agregados (1400)
- Instrumentos de pagamento eletrônico (1601)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra a geração completa da EFD.

### `gerarAberturaBloco0()`
Inicializa os contadores e gera registros de abertura (0000, 0001, 0005, 0015, 0100).

### `gerarBlocoC()`
Processa documentos fiscais de mercadorias (C100, C300, C500, C800) e seus registros filhos.

### `gerarBlocoE()`
Gera apurações de ICMS, ST, DIFAL e IPI com seus ajustes e obrigações.

### `gerarBlocoG()`
Processa o CIAP com movimentações de bens e créditos apropriados.

### `gerarBlocoH()`
Gera o inventário físico com itens e valores.

### `gerarBlocoK()`
Controle de produção e estoque com registros de movimentações e correções.

### `gerarFechamentoBloco0()`
Completa o bloco 0 com registros cadastrais (0150, 0190, 0200, 0300, 0400, 0450, etc.).

### `gerarBloco9()`
Gera o bloco 9 com totais de registros e encerramento do arquivo.

## 📊 Estrutura de Saída

**Arquivo de Texto:**
- Dois arquivos concatenados (`txt1` e `txt2`)
- Formato delimitado por "|"
- Codificação UTF-8
- Linhas no padrão EFD: `REG|CAMPO1|CAMPO2|...|CAMPON`

**Blocos Gerados:**
- 0, B, C, D, E, G, H, K, 1, 9

**Saída no Parâmetro:** `dadosArquivo`

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Criteria e consultas ao banco
- `multitec.utils` – Utilitários de data, texto e validação
- `sam.model` – Entidades do sistema
- `java.time` – Manipulação de datas
- `java.math` – Cálculos com BigDecimal

**Módulos:** Sistema Fiscal SAM

## 📝 Observações Técnicas

### Tratamento de Perfis (A, B, C)
- Perfil A: Completo, todas as operações
- Perfil B: Limitado, algumas operações de saída
- Perfil C: Simplificado, apenas entradas e saídas essenciais

### Campos JSON
- Valores fiscais armazenados em campos JSON nas tabelas (`eaa01json`, `eaa0103json`, `edb01json`, etc.)
- Acesso via `jGet(campo)::numeric`

### Controle de Paginação
- Consultas paginadas para documentos fiscais (evita estouro de memória)

### Validações
- Empresa com município e informações fiscais
- Documentos com entidade e situação definida
- Apurações obrigatórias conforme perfil

### Performance
- Uso de `Set` e `Map` para evitar duplicidades (ex: 0150, 0200, 0300)
- Consultas otimizadas com filtros por período e empresa

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula EFD-ICMS/IPI  
**Versão:** Leiaute 19/2025  
**Sistema:** SAM