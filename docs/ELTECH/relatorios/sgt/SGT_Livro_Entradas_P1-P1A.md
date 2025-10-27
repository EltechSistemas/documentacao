## 📖 Descrição
Sistema de geração do Livro de Registro de Entradas modelos P1 e P1A para fins fiscais, incluindo termos de abertura e encerramento conforme legislação.

## 🎯 Finalidade
Atender às obrigações fiscais de registro de entradas de mercadorias, gerando livros fiscais P1/P1A com demonstrativos por estado, resumo por CFOP e controle de numeração sequencial.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Gestão Tributária

## ⚙️ Configuração
**Templates Jasper:**
- `SGT_LivroEntradasP1P1A_R1` - Termos de abertura/encerramento
- `SGT_LivroEntradasP1P1A_R2` - Livro P1
- `SGT_LivroEntradasP1P1A_R3` - Livro P1A
- `SGT_LivroEntradasP1P1A_R2S1` - Demonstrativo por estado
- `SGT_LivroEntradasP1P1A_R2S2` - Resumo por CFOP
- `SGT_LivroEntradasP1P1A_R2S3` - Resumo por CFOP e alíquota

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos
- `ABB01` - Cabeçalho de documentos
- `AAJ15` - CFOP
- `EDD10` - Controle de livros fiscais
- `AAC10` - Dados da empresa
- `AAC1002` - Inscrições estaduais

**Entidades Envolvidas:**
- `Edd10` - Controle de livros
- `Eaa01` - Documentos fiscais
- `Aac10` - Empresa
- `Aag0201` - Municípios

## ⚙️ Parâmetros do Relatório

| Parâmetro   | Tipo      | Obrigatório | Descrição                | Valores Possíveis                               |     |
| ----------- | --------- | ----------- | ------------------------ | ----------------------------------------------- | --- |
| imprimir    | Integer   | Sim         | Tipo de impressão        | 0=Livro, 1=Termo Abertura, 2=Termo Encerramento |     |
| dataInicial | LocalDate | Sim         | Data inicial do período  | Formato dd/MM/yyyy                              |     |
| dataFinal   | LocalDate | Sim         | Data final do período    | Formato dd/MM/yyyy                              |     |
| modelo      | Integer   | Sim         | Modelo do livro          | 0=P1, 1=P1A                                     |     |
| entidade    | Boolean   | Sim         | Exibir dados da entidade | true/false                                      |     |
| rascunho    | Boolean   | Sim         | Modo rascunho            | true/false                                      |     |
| resumo      | Boolean   | Sim         | Exibir resumo CFOP       | true/false                                      |     |
| livro       | Integer   | Sim         | Número do livro          | Sequencial                                      |     |
| pagina      | Integer   | Sim         | Página inicial           | Número da página                                |     |

## 📋 Campos do Livro

| Campo | Descrição | Tipo |
|-------|-----------|------|
| eaa01esData | Data de entrada | Date |
| abb01num | Número do documento | Integer |
| aaj15codigo | CFOP | String |
| eaa0102nome | Nome do fornecedor | String |
| eaa0102ni | CNPJ/CPF | String |
| aag02uf | UF do fornecedor | String |
| vlr_contabil | Valor contábil | BigDecimal |
| bc_icms | Base cálculo ICMS | BigDecimal |
| icms | Valor ICMS | BigDecimal |
| bc_ipi | Base cálculo IPI | BigDecimal |
| ipi | Valor IPI | BigDecimal |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define campos JSON para impostos (ICMS, IPI)
   - Configura alinhamento de impressão
   - Obtém dados da empresa e responsáveis

2. **Validação de Parâmetros**
   - Verifica período fiscal
   - Valida número do livro e página
   - Configura termos de abertura/encerramento

3. **Busca de Dados Fiscais**
   - Consulta documentos de entrada no período
   - Agrupa por UF, CFOP e alíquotas
   - Calcula totais e impostos

4. **Geração do Relatório**
   - Seleciona template conforme modelo (P1/P1A)
   - Processa sub-relatórios (demonstrativos)
   - Gera PDF com numeração de páginas

5. **Controle de Numeração**
   - Grava/atualiza controle do livro
   - Valida sequência numérica
   - Controla termos de encerramento

## ⚠️ Regras de Negócio

### Validações Fiscais
- Apenas documentos de entrada (`ESMOV_ENTRADA`)
- Apenas documentos SRF (`CLASDOC_SRF`)
- Documentos com indicação para livro fiscal (`iLivroFisc = SIM`)
- CFOPs 1xxx, 2xxx e 3xxx

### Controle de Livros
- Numeração sequencial automática
- Impedimento de livro em aberto
- Bloqueio de livro já encerrado
- Controle de páginas por livro

### Campos de Impostos
- Configuração via parâmetros JSON
- Campos: alíquota, base cálculo, valor, isentas, outras
- Suporte a ICMS, IPI e ICMS ST

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Livro fiscal completo | `JasperExportManager.exportReportToPdf()` |
| PDF | Termos de abertura/encerramento | Template específico |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Persistência e queries
- `multitec.utils` - Utilitários e datas

**Configurações:**
- Parâmetros JSON para campos de impostos
- Templates Jasper para diferentes modelos
- Controle sequencial de livros (`EDD10`)

## 📝 Observações Técnicas

- Utiliza sub-relatórios para demonstrativos consolidados
- Processa campos fiscais via funções JSON (`jGet`)
- Controle rigoroso de numeração de livros e páginas
- Suporte a modo rascunho sem gravação
- Validação de estado do livro (aberto/encerrado)
- Configuração dinâmica de alinhamento de impressão