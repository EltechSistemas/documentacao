# SGT_ApuracaoICMSP9.md

## 📖 Descrição
Sistema de geração do Livro de Apuração do ICMS (P9) para a Linhasita, responsável pela apuração e registro dos valores de ICMS e ICMS-ST no período.

## 🎯 Finalidade
Fornecer relatório completo de apuração do ICMS conforme exigências fiscais, incluindo cálculos de débitos, créditos e saldos a recolher ou compensar.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria
- Auditores fiscais

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SGT_ApuracaoICMSP9_R2` - Layout ICMS
- Template Jasper `SGT_ApuracaoICMSP9_R3` - Layout ICMS-ST
- Template Jasper `SGT_ApuracaoICMSP9_R1` - Termos de abertura/encerramento
- Múltiplos sub-relatórios para seções específicas

**Localização:** `eltech/relatorios/sgt/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos
- `EDB01` - Apurações fiscais
- `EDB0101` - Ocorrências da apuração
- `EDD10` - Livros fiscais
- `AAJ15` - CFOPs
- `AAG02` - Estados/UF

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens fiscais
- `Edb01` - Apuração principal
- `Edb0101` - Detalhes da apuração
- `Edd10` - Controle de livros

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| imprimir | Integer | Sim | Tipo de impressão | 0-Livro, 1-Termo Abertura, 2-Termo Encerramento |
| referencia | String | Sim | Período de referência | MM/aaaa |
| modelo | Integer | Sim | Tipo de apuração | 0-ICMS, 1-ICMS-ST |
| estados | List<Long> | Não | Estados para filtro | IDs dos estados |
| apuracao | Long | Não | Tipo de apuração específica | ID do tipo |
| rascunho | Boolean | Sim | Modo rascunho | true/false |

## 📋 Tipos de Apuração

### ICMS (0)
- Apuração normal do ICMS
- Entradas e saídas por CFOP
- Cálculo de débitos e créditos
- Saldo a recolher ou compensar

### ICMS-ST (1)
- Apuração do ICMS Substituição Tributária
- Filtro por estados específicos
- Cálculos específicos para ST
- Diferenciação por UF

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica período de referência
   - Valida tipo de apuração selecionado
   - Confirma existência de dados fiscais

2. **Busca de Dados Fiscais**
   - Lançamentos agrupados por CFOP
   - Cálculos de bases e valores de ICMS
   - Dados de entradas e saídas

3. **Processamento da Apuração**
   - Cálculo de débitos e créditos
   - Aplicação de estornos e deduções
   - Cálculo do saldo final

4. **Composição do Relatório**
   - Múltiplos sub-relatórios especializados
   - Agrupamento por UF (ICMS-ST)
   - Formatação fiscal específica

5. **Controle de Livros**
   - Numeração sequencial de livros
   - Termos de abertura e encerramento
   - Controle de páginas

## ⚠️ Regras de Negócio

### Validações
- Período deve ser válido e consistente
- Dados fiscais devem estar integrados
- Livro não pode estar encerrado para novas apurações
- Estados devem estar cadastrados para ICMS-ST

### Cálculos Fiscais
- **Base de Cálculo:** Soma das bases de ICMS dos lançamentos
- **Valor do ICMS:** Soma dos valores de ICMS
- **Débitos/Créditos:** Cálculo conforme regras fiscais
- **Saldo Final:** Débitos - Créditos - Deduções

### Campos de Alinhamento
- Configuração específica para campos JSON
- Mapeamento de campos fiscais padronizados
- Suporte a múltiplos tipos de apuração

## 🎨 Estrutura do Relatório

### Seção 1 - Dados da Empresa
- Razão social e CNPJ
- Inscrição Estadual
- Período de apuração

### Seção 2 - Lançamentos por CFOP
- Agrupamento por grupo de CFOP (1.x, 2.x, 3.x, 5.x, 6.x, 7.x)
- Valores contábeis e fiscais
- Bases e valores de ICMS

### Seção 3 - Apuração do ICMS
- Débitos e créditos do período
- Estornos e compensações
- Saldo a recolher

### Seção 4 - Ocorrências
- Detalhamento por tipo de ocorrência
- Valores específicos por subitem
- Observações fiscais

## 🔧 Dependências

**Parâmetros de Alinhamento:**
- `0040` - Apuração LF ICMS
- `0082` - Resumo de Apuração ICMS P9

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Acesso a dados
- `java.time` - Manipulação de datas

## 📝 Observações Técnicas

- Suporte a múltiplos sub-relatórios especializados
- Cálculos fiscais precisos baseados em JSON fields
- Controle rigoroso de livros fiscais
- Formatação específica para cada tipo de apuração
- Integração com sistema de alinhamento de campos
- Validações de consistência dos dados fiscais
- Suporte a filtros por UF para ICMS-ST