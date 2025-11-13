# SGT_LivroSaidasP2P2A.md

## 📖 Descrição
Sistema de geração do Livro de Saídas (P2/P2A) para a Linhasita, responsável pelo registro fiscal de todas as operações de saída no período.

## 🎯 Finalidade
Fornecer relatório completo de saídas fiscais conforme exigências legais, incluindo dados de ICMS, IPI e demais tributos incidentes nas operações.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria
- Auditores fiscais

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SGT_LivroSaidasP2P2A_R2` - Layout P2 (sem IPI)
- Template Jasper `SGT_LivroSaidasP2P2A_R3` - Layout P2A (com IPI)
- Template Jasper `SGT_LivroSaidasP2P2A_R1` - Termos de abertura/encerramento
- Múltiplos sub-relatórios para demonstrações e resumos

**Localização:** `eltech/relatorios/sgt/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos
- `ABB01` - Cabeçalho de documentos
- `AAJ15` - CFOPs
- `EAA0102` - Dados gerais do documento
- `EAA0101` - Endereços
- `AAG02` - Estados/UF
- `EDD10` - Livros fiscais

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens fiscais
- `Abb01` - Cabeçalho documentos
- `Eaa0102` - Dados do destinatário
- `Edd10` - Controle de livros

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| imprimir | Integer | Sim | Tipo de impressão | 0-Livro, 1-Termo Abertura, 2-Termo Encerramento |
| dataInicial | LocalDate | Sim | Data inicial do período | dd/MM/yyyy |
| dataFinal | LocalDate | Sim | Data final do período | dd/MM/yyyy |
| modelo | Integer | Sim | Modelo do livro | 0-P2, 1-P2A |
| consEntNosDocDeSaida | Boolean | Sim | Considerar entradas em docs de saída | true/false |
| rascunho | Boolean | Sim | Modo rascunho | true/false |
| resumo | Boolean | Sim | Exibir resumo por CFOP e alíquota | true/false |

## 📋 Modelos do Livro

### P2 (0)
- Livro de Saídas sem IPI
- Foco nas operações de ICMS
- Demonstrações por estado e CFOP

### P2A (1)
- Livro de Saídas com IPI
- Inclui dados de IPI além do ICMS
- Formatação específica para indústria

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica período informado
   - Valida modelo selecionado
   - Confirma configurações fiscais

2. **Busca de Dados Fiscais**
   - Documentos de saída no período
   - Itens fiscais com tributos
   - Dados de destinatários e CFOPs

3. **Processamento dos Dados**
   - Agrupamento por diversos critérios
   - Cálculos de bases e valores tributários
   - Formatação específica por modelo

4. **Composição do Relatório**
   - Dados principais dos documentos
   - Demonstrações por estado (contribuintes/não contribuintes)
   - Resumos por CFOP e alíquotas
   - Sub-relatórios especializados

5. **Controle de Livros**
   - Numeração sequencial
   - Termos de abertura e encerramento
   - Controle de páginas

## ⚠️ Regras de Negócio

### Validações
- Período deve ser válido e consistente
- Documentos cancelados têm valores zerados
- Apenas documentos fiscais integrados
- Livro não pode estar encerrado

### Cálculos Fiscais
- **Valor Contábil:** Soma dos totais dos itens
- **Base ICMS:** Soma das bases de cálculo
- **Valor ICMS:** Soma dos valores do imposto
- **Base IPI:** Soma das bases de IPI (P2A)
- **Valor IPI:** Soma dos valores de IPI (P2A)

### Campos de Alinhamento
- Configuração específica para campos JSON
- Mapeamento de campos fiscais padronizados (0080)
- Suporte a múltiplos tipos de dados tributários

## 🎨 Estrutura do Relatório

### Seção Principal - Documentos de Saída
- Data e número do documento
- CFOP e descrição
- Destinatário (código, nome, IE)
- Valores contábeis e fiscais
- Tributos incidentes

### Sub-relatórios Especializados

**R2S1:** Demonstrativo por estado - Operações com contribuinte
**R2S4:** Demonstrativo por estado - Operações com não contribuinte  
**R2S2:** Resumo por CFOP
**R2S3:** Resumo por CFOP e alíquota de ICMS

## 🔧 Dependências

**Parâmetros de Alinhamento:**
- `0080` - Livro de Saídas P2/P2A

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Acesso a dados
- `java.time` - Manipulação de datas

## 📝 Observações Técnicas

- Suporte a múltiplos sub-relatórios especializados
- Zeramento automático de valores para documentos cancelados
- Controle rigoroso de livros fiscais (P2/P2A)
- Formatação específica para cada modelo
- Integração com sistema de alinhamento de campos
- Validações de consistência dos dados fiscais
- Suporte a entradas em documentos de saída (opcional)
- Resumos agrupados por diversos critérios