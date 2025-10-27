## 📖 Descrição
Relatório abrangente de documentos financeiros (receber/pagar) com múltiplas opções de ordenação, filtros e formatações para análise financeira detalhada.

## 🎯 Finalidade
Fornecer visão completa da situação de documentos financeiros, permitindo análise por diferentes critérios como vencimento, entidade, portador e representante, com cálculos automáticos de juros, multas e descontos.

## 👥 Público-Alvo
- Departamento Financeiro
- Controladoria
- Cobrança
- Gestão de Contas a Pagar/Receber

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA01` - Documentos financeiros
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades (clientes/fornecedores)
- `ABF15` - Portadores
- `ABF16` - Operações
- `AAH01` - Tipos de documento
- `ABF20` - PLF (Plano de Livro Fiscal)

**Entidades Envolvidas:**
- `Daa01` - Documentos financeiros
- `Abe01` - Clientes/Fornecedores/Representantes
- `Abf15` - Portador
- `Abf16` - Operação
- `Aac10` - Empresa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| classe | Integer | Sim | Classe de documentos | 0=A Receber, 1=Recebidos, 2=A Pagar, 3=Pagos, 4=A Receber/Recebidos, 5=A Pagar/Pagos |
| ordem | Integer | Sim | Ordenação | 0=Número, 1=Vencimento, 2=Entidade, 3=Pagamento, 4=Portador, 5=Representante |
| numeroInicial | Integer | Sim | Número inicial | 000000000 |
| numeroFinal | Integer | Sim | Número final | 999999999 |
| tipoData | Integer | Sim | Tipo data pagamento | 0=Pagamento, 1=Recebimento |
| tipo | Integer | Sim | Tipo documento | 0=Real, 1=Previsão, 2=Todos |
| impressao | Integer | Sim | Formato saída | 0=PDF, 1=XLSX |
| vencimento | Integer | Sim | Tipo vencimento | 0=Real, 1=Nominal |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abb01num | Número documento | Integer |
| abb01parcela | Parcela | String |
| ent.abe01nome | Nome entidade | String |
| aah01codigo | Tipo documento | String |
| vencimento | Data vencimento | Date |
| daa01dtPgto | Data pagamento | Date |
| valor | Valor documento | BigDecimal |
| dias | Dias em atraso | Long |
| jm | Juros/Multas | BigDecimal |
| desconto | Valor desconto | BigDecimal |
| abf15nome | Portador | String |
| repNome | Representante | String |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define título conforme classe e ordenação
   - Configura período no cabeçalho
   - Processa parâmetros complexos

2. **Busca de Documentos**
   - Aplica múltiplos filtros dinâmicos
   - Define ordenação flexível
   - Processa joins com diversas entidades

3. **Cálculos Financeiros**
   - Calcula dias em atraso
   - Processa juros, multas e descontos via JSON
   - Define tipo de documento (Real/Previsão)

4. **Processamento de Dados**
   - Remove duplicatas por ID
   - Formata campos específicos
   - Prepara dados para relatório

5. **Seleção de Template**
   - Escolhe template conforme ordenação
   - Define sintético ou analítico
   - Gera PDF ou XLSX

## ⚠️ Regras de Negócio

### Classificação de Documentos
- **Classe 0/2:** Apenas documentos em aberto
- **Classe 1/3:** Apenas documentos quitados
- **Classe 4/5:** Todos os documentos

### Cálculos Financeiros
- **Dias:** Diferença entre data atual/pagamento e vencimento
- **JM:** Soma de juros e multas (calculados ou quitados)
- **Desconto:** Valor absoluto (sempre positivo)

### Filtros de Data
- **Vencimento:** Data nominal ou real conforme parâmetro
- **Emissão:** Data do documento
- **Pagamento/Recebimento:** Conforme tipoData

## 🎨 Saídas Disponíveis

| Template | Descrição | Ordem | Formato |
|----------|-----------|-------|---------|
| SCF_Documentos | Layout padrão | 0,1,3 | PDF/XLSX |
| SCF_DocumentosEntidade | Por entidade | 2 | PDF/XLSX |
| SCF_DocumentosPortador | Por portador | 4 | PDF/XLSX |
| SCF_DocumentosRepresentante | Por representante | 5 | PDF/XLSX |
| SCF_DocumentosSintetico2 | Sintético | 0 | PDF/XLSX |

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utilitários e datas
- `jasperreports` - Geração de relatórios

**Configurações:**
- Múltiplos templates Jasper para diferentes ordenações
- Parâmetros JSON para campos financeiros

## 📝 Observações Técnicas

- SQL extremamente dinâmico com múltiplos filtros opcionais
- Processamento complexo de campos JSON para cálculos
- Remoção de duplicatas baseada em ID do documento
- Flexibilidade total de ordenação e agrupamento
- Suporte a diferentes tipos de vencimento (real/nominal)
- Cálculo automático de dias em atraso e encargos