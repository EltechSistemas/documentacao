# SRF_ImportarXMLNFeEntrada.md

## 📖 Descrição
Sistema de importação de XML de Nota Fiscal Eletrônica (NF-e) para documentos de entrada, responsável por extrair e processar todos os dados fiscais, tributários e comerciais do arquivo XML.

## 🎯 Finalidade
Automatizar o processo de importação de dados fiscais de NF-e de entrada, garantindo a correta extração e mapeamento de informações tributárias, valores, produtos e dados complementares para o sistema.

## 👥 Público-Alvo
- Departamento Fiscal
- Compras
- Almoxarifado/Estoque
- Contabilidade

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_ImportarXMLNFeEntrada` - Processamento de XML NF-e entrada

**Localização:** `eltech/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens do documento
- `EAA01038` - Controle de lotes/séries
- `EAA0113` - Duplicatas
- `EAA01033` - Itens referenciados

**Entidades Envolvidas:**
- `Eaa01` - Documento fiscal
- `Eaa0103` - Item do documento
- `ElementXml` - Parser de XML

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| elementXmlNfe | ElementXml | Sim | Estrutura XML da NF-e a ser importada |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Verifica estrutura do XML (nfeProc > NFe > infNFe)
   - Valida nó principal do documento fiscal

2. **Processamento de Itens**
   - Percorre todos os itens (det) do XML
   - Busca item correspondente no documento por número de sequência
   - Processa dados básicos do produto (NCM, unidade tributária)

3. **Extração de Dados Tributários**
   - **ICMS:** Todos os grupos (00, 10, 20, 30, 40, 51, 60, 70, 90, Part, ST, SN)
   - **IPI:** Dados de impostos sobre produtos industrializados
   - **PIS/COFINS:** Tributos federais normal e ST
   - **II:** Imposto de importação
   - **ISSQN:** Imposto sobre serviços

4. **Controle de Lotes e Séries**
   - Processa informações de rastreamento (rastro)
   - Aplica fatores de conversão de unidades
   - Define status e controles conforme configuração do PCD

5. **Dados Complementares**
   - Totais da NF-e (valores, impostos consolidados)
   - Informações de transporte (volumes, pesos)
   - Observações fiscais e complementares
   - Duplicatas (quando aplicável)

## ⚠️ Regras de Negócio

### Validações de Estrutura
- XML deve conter estrutura nfeProc > NFe > infNFe
- Itens devem corresponder sequencialmente aos itens do documento
- Campos numéricos são convertidos com casas decimais específicas

### Processamento de Tributos
- **ICMS:** Suporte a todos os CSTs (00, 10, 20, 30, 40, 51, 60, 70, 90)
- **IPI:** Tratamento para cálculos por alíquota e por unidade
- **PIS/COFINS:** Diferenciação entre regime cumulativo e não-cumulativo
- **Substituição Tributária:** Cálculos de MVA, reduções e bases

### Controle de Estoques
- Aplicação automática de fatores de conversão
- Definição de status conforme configuração do PCD
- Controles específicos para operações com estoque

### Referências e Relacionamentos
- Vinculação automática de NF-e referenciadas
- Processamento de duplicatas para documentos financeiros
- Mapeamento de observações fiscais e complementares

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| eaa01 | Documento fiscal com dados importados | Eaa01 |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `br.com.multitec.utils.xml` - Processamento de XML
- `sam.model.entities` - Entidades do sistema

**Estruturas de Dados:**
- `ElementXml` - Parser e navegação de XML
- `TableMap` - Armazenamento de dados JSON

## 📝 Observações Técnicas

- **Processamento:** Síncrono, execução completa do XML
- **Performance:** Processamento otimizado por lotes de itens
- **Flexibilidade:** Suporte a múltiplos regimes tributários
- **Precisão:** Arredondamentos específicos por tipo de campo

### Mapeamento de Campos XML
- **Produtos:** NCM, unidades, quantidades, valores
- **Tributos:** Bases de cálculo, alíquotas, valores
- **Transporte:** Volumes, pesos líquido e bruto
- **Complementares:** Observações, protocolos, chaves de acesso

### Tratamento de Dados
- Conversão automática de datas e valores
- Aplicação de fatores de conversão de unidades
- Validação de consistência de dados
- Preservação de dados originais do XML

### Integração com Sistema
- Atualização em tempo real do documento fiscal
- Manutenção de relacionamentos entre entidades
- Suporte a operações de entrada com estoque
- Compatibilidade com diferentes tipos de PCD