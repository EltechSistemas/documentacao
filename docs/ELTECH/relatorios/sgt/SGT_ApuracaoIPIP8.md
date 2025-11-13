# SGT_ApuracaoIPIP8.md

## 📖 Descrição
Sistema de geração do Livro de Apuração do IPI (P8) para a Linhasita, responsável pela apuração e registro dos valores de IPI no período.

## 🎯 Finalidade
Fornecer relatório completo de apuração do IPI conforme exigências fiscais, incluindo cálculos de débitos, créditos e saldos a recolher ou compensar.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Controladoria
- Auditores fiscais

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SGT_ApuracaoIPIP8_R2` - Layout principal do IPI
- Template Jasper `SGT_ApuracaoIPIP8_R1` - Termos de abertura/encerramento
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
| apuracao | Long | Não | Tipo de apuração específica | ID do tipo |
| rascunho | Boolean | Sim | Modo rascunho | true/false |

## 📋 Estrutura do Relatório

### Seção 1 - Dados da Empresa
- Razão social e CNPJ
- Inscrição Estadual
- Período de apuração

### Seção 2 - Lançamentos por CFOP (Entradas)
- Agrupamento por grupo de CFOP (1.x, 2.x, 3.x)
- Valores contábeis e fiscais
- Bases e valores de IPI
- Isenções e outras situações

### Seção 3 - Lançamentos por CFOP (Saídas)
- Agrupamento por grupo de CFOP (5.x, 6.x, 7.x)
- Valores contábeis e fiscais
- Bases e valores de IPI
- Isenções e outras situações

### Seção 4 - Apuração do IPI
- Débitos e créditos do período
- Estornos e compensações
- Créditos anteriores
- Deduções aplicáveis
- Saldo a recolher

### Seção 5 - Ocorrências
- Detalhamento por tipo de ocorrência
- Valores específicos por subitem
- Observações fiscais

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica período de referência
   - Valida tipo de apuração selecionado
   - Confirma existência de dados fiscais

2. **Busca de Dados Fiscais**
   - Lançamentos agrupados por CFOP
   - Cálculos de bases e valores de IPI
   - Dados de entradas e saídas

3. **Processamento da Apuração**
   - Cálculo de débitos e créditos
   - Aplicação de estornos e deduções
   - Cálculo do saldo final

4. **Composição do Relatório**
   - Múltiplos sub-relatórios especializados
   - Formatação fiscal específica para IPI

5. **Controle de Livros**
   - Numeração sequencial de livros
   - Termos de abertura e encerramento
   - Controle de páginas

## ⚠️ Regras de Negócio

### Validações
- Período deve ser válido e consistente
- Dados fiscais devem estar integrados
- Livro não pode estar encerrado para novas apurações
- Documentos cancelados são excluídos

### Cálculos Fiscais
- **Base de Cálculo:** Soma das bases de IPI dos lançamentos
- **Valor do IPI:** Soma dos valores de IPI
- **Débitos/Créditos:** Cálculo conforme regras fiscais
- **Saldo Final:** Débitos - Créditos - Deduções

### Campos de Alinhamento
- **0040** - Apuração LF ICMS/IPI (campos gerais)
- **0083** - Resumo de Apuração IPI P8 (campos específicos)

## 🎨 Sub-relatórios Especializados

| Sub-relatório | Descrição |
|---------------|-----------|
| R2S1 | Lançamentos de entrada por CFOP |
| R2S2 | Lançamentos de saída por CFOP |
| R2S3 | Apuração principal do IPI |
| R2S3S1-R2S3S5 | Ocorrências detalhadas por tipo |
| R2S4 | Resumo final da apuração |

## 🔧 Dependências

**Parâmetros de Alinhamento:**
- `0040` - Apuração LF ICMS/IPI
- `0083` - Resumo de Apuração IPI P8

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Acesso a dados
- `java.time` - Manipulação de datas

## 📝 Observações Técnicas

- Suporte a múltiplos sub-relatórios especializados
- Cálculos fiscais precisos baseados em JSON fields
- Controle rigoroso de livros fiscais (P8)
- Formatação específica para apuração de IPI
- Integração com sistema de alinhamento de campos
- Validações de consistência dos dados fiscais
- Suporte a termos de abertura e encerramento
- Modo rascunho para testes e verificações