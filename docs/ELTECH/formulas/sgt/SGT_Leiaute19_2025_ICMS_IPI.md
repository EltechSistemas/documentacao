# SGT_Leiaute19_2025_ICMS_IPI

## 📖 Descrição
Gerador da Escrituração Fiscal Digital (EFD) para ICMS e IPI conforme Leiaute 19/2025. Implementa todos os blocos e registros necessários para atender às obrigações acessórias fiscais.

## 🎯 Finalidade
Gerar arquivo texto no formato EFD-ICMS/IPI contendo todas as operações fiscais do período, incluindo documentos fiscais, apurações, inventários e demais informações exigidas pela legislação.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Consultores Tributários

## ⚙️ Parâmetros/Configurações

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| dtInicial | LocalDate | Sim | Data inicial do período | Data no formato dd/MM/yyyy |
| dtFinal | LocalDate | Sim | Data final do período | Data no formato dd/MM/yyyy |
| arqSubstituto | Integer | Sim | Indicador de arquivo substituto | 0=Original, 1=Substituto |
| dtInventario | LocalDate | Não | Data do inventário | Data no formato dd/MM/yyyy |

## 📊 Estrutura de Blocos

### Bloco 0 - Abertura, Identificação e Referências
- **0000**: Abertura do arquivo digital
- **0001**: Abertura do Bloco 0
- **0002**: Classificação do Estabelecimento Industrial
- **0005**: Dados Complementares da Entidade
- **0015**: Dados do Contribuinte Substituto
- **0100**: Dados do Contabilista
- **0150**: Tabela de Cadastro do Participante
- **0190**: Unidades de Medida
- **0200**: Identificação do Item
- **0300**: Cadastro de Bens do CIAP
- **0400**: Natureza da Operação/Prestação
- **0450**: Informação Complementar do Documento Fiscal
- **0460**: Observações do Lançamento Fiscal
- **0500**: Plano de Contas Contábeis
- **0600**: Centros de Custo

### Bloco B - Escrituração e Apuração do ISS
- **B001**: Abertura do Bloco B
- **B990**: Encerramento do Bloco B

### Bloco C - Documentos Fiscais I - Mercadorias (ICMS/IPI)
- **C001**: Abertura do Bloco C
- **C100**: Nota Fiscal, Nota Fiscal Avulsa, Nota Fiscal de Produtor e NF-e
- **C300**: Resumo Diário de NFC-e
- **C500**: NF de Energia Elétrica, Água e Gás
- **C800**: Cupom Fiscal Eletrônico - SAT

### Bloco D - Documentos Fiscais II - Serviços (ICMS)
- **D001**: Abertura do Bloco D
- **D100**: Nota Fiscal de Serviço de Transporte e Conhecimentos de Transporte
- **D500**: Nota Fiscal de Serviço de Comunicação e Telecomunicação

### Bloco E - Apuração do ICMS e do IPI
- **E001**: Abertura do Bloco E
- **E100**: Período da Apuração do ICMS
- **E110**: Apuração do ICMS - Operações Próprias
- **E200**: Apuração do ICMS - Substituição Tributária
- **E300**: Apuração do ICMS Diferencial de Alíquota
- **E500**: Apuração do IPI

### Bloco G - CIAP (Controle de Crédito de ICMS do Ativo Permanente)
- **G001**: Abertura do Bloco G
- **G110**: ICMS - Ativo Permanente - CIAP
- **G125**: Movimentação de Bem ou Componente do Ativo Imobilizado
- **G130**: Identificação do Documento Fiscal
- **G140**: Identificação do Item do Documento Fiscal

### Bloco H - Inventário Físico
- **H001**: Abertura do Bloco H
- **H005**: Totais do Inventário
- **H010**: Inventário
- **H020**: Informação Complementar do Inventário

### Bloco K - Controle da Produção e Estoque
- **K001**: Abertura do Bloco K
- **K100**: Período de Apuração
- **K200**: Estoque Escriturado
- **K220**: Outras Movimentações Internas
- **K230**: Itens Produzidos
- **K250**: Industrialização por Terceiros
- **K270**: Correção de Apontamento
- **K290**: Produção Conjunta

### Bloco 1 - Outras Informações
- **1001**: Abertura do Bloco 1
- **1010**: Obrigatoriedade de Registros
- **1100**: Informações sobre Exportação
- **1400**: Informação sobre Valores Agregados
- **1601**: Operações com Instrumentos de Pagamento Eletrônico

### Bloco 9 - Controle e Encerramento
- **9001**: Abertura do Bloco 9
- **9900**: Registros do Arquivo
- **9990**: Encerramento do Bloco 9
- **9999**: Encerramento do Arquivo Digital

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica datas do período
   - Valida dados da empresa
   - Configura alinhamentos fiscais

2. **Inicialização**
   - Cria arquivos texto de saída
   - Inicializa contadores de registros
   - Prepara estruturas de dados

3. **Geração dos Blocos**
   - Bloco 0: Dados da empresa e cadastros
   - Bloco B: ISS (quando aplicável)
   - Bloco C: Documentos fiscais de mercadorias
   - Bloco D: Documentos fiscais de serviços
   - Bloco E: Apurações de impostos
   - Bloco G: CIAP
   - Bloco H: Inventário
   - Bloco K: Controle de produção
   - Bloco 1: Informações complementares

4. **Finalização**
   - Gera totais e encerramentos
   - Consolida arquivo final
   - Atualiza controles de envio

## ⚠️ Regras de Negócio

### Validações Obrigatórias
- Empresa deve ter município cadastrado
- Informações fiscais devem estar completas
- Perfil da empresa deve ser definido
- Apurações do período devem existir

### Tratamento de Documentos
- Considera documentos com data de entrada/saída no período
- Aplica filtros por modelo de documento
- Trata documentos cancelados adequadamente
- Processa apenas documentos marcados para EFD

### Cálculos Fiscais
- Campos fiscais são obtidos do JSON do documento
- Valores são formatados com precisão específica por registro
- CSTs e CFOPs são validados e formatados

### Perfis de Apuração
- **Perfil A**: Indústria e equiparado
- **Perfil B**: Comércio
- **Perfil C**: Prestador de serviços

## 🎨 Saídas/Retornos

| Tipo | Descrição | Formato |
|------|-----------|---------|
| Arquivo EFD | Arquivo texto com todos os blocos | Texto delimitado por pipe |
| Log de Processamento | Contadores e status | Console/Interface |

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários gerais
- `java.time` - Manipulação de datas
- `jasperreports` - Relatórios (não utilizado neste código)

**Entidades do Sistema:**
- `Eaa01` - Documentos fiscais
- `Abb01` - Cabeçalho de documentos
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Itens (produtos/serviços)
- `Edb01` - Apurações fiscais
- `Ecc01` - Fichas CIAP

## 📝 Observações Técnicas

### Performance
- Utiliza paginação na consulta de documentos
- Implementa cache de entidades para evitar consultas repetidas
- Otimiza processamento com estruturas Map e Set

### Tratamento de Dados
- Campos JSON são acessados dinamicamente via `jGet()`
- Formatação específica para cada tipo de campo
- Normalização de textos para remover acentos

### Controles
- Contadores individuais por tipo de registro
- Verificação de cancelamento do processo
- Atualização de status durante execução

### Validações Específicas
- Regime especial de apuração
- Operações com diferencial de alíquota
- Documentos de exportação
- Industrialização por terceiros

**Última Atualização:** 07/11/2025
**Versão do Leiaute:** 19