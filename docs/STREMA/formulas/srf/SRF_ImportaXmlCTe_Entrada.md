# SRF_ImportaXmlCTe_Entrada.md

## 📖 Descrição
Sistema de importação de XML de Conhecimento de Transporte Eletrônico (CT-e) para documentos de entrada, responsável por extrair e processar dados fiscais, valores e informações de transporte do arquivo XML.

## 🎯 Finalidade
Automatizar o processo de importação de dados fiscais de CT-e de entrada, garantindo a correta extração e mapeamento de informações tributárias, valores da prestação de serviço e dados de transporte para o sistema.

## 👥 Público-Alvo
- Departamento Fiscal
- Logística/Transportes
- Compras
- Contabilidade

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_ImportaXmlCTe_Entrada` - Processamento de XML CT-e entrada

**Localização:** `strema/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens do documento
- `EAA0102` - Dados gerais do documento
- `EAA0101` - Endereços do documento
- `AAG0201` - Municípios

**Entidades Envolvidas:**
- `Eaa01` - Documento fiscal
- `Eaa0103` - Item do documento
- `ElementXml` - Parser de XML

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |
| elementXmlCte | ElementXml | Sim | Estrutura XML do CT-e a ser importada |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Verifica estrutura do XML (cteProc > CTe > infCte)
   - Obtém primeiro item do documento para processamento
   - Valida nó principal do documento fiscal

2. **Processamento de Dados Básicos**
   - Define tipo do CT-e (tpCTe) nos dados gerais
   - Configura quantidades padrão (1 unidade)
   - Extrai valores da prestação de serviço

3. **Processamento Tributário**
   - **ICMS 00:** Tributação normal
   - **ICMS 20:** Com redução de base de cálculo
   - **ICMS 60:** Substituição tributária
   - **ICMS 90:** Outros casos
   - **ICMS Outra UF:** Operações interestaduais

4. **Gestão de Endereços**
   - Remove endereços de saída e entrega existentes
   - Cria novos endereços baseados no XML
   - Busca municípios por código IBGE
   - Define endereços de saída e entrega

5. **Dados Complementares**
   - Informações adicionais do fisco
   - Valores da carga e averbação
   - Observações fiscais

## ⚠️ Regras de Negócio

### Validações de Estrutura
- XML deve conter estrutura cteProc > CTe > infCte
- Pelo menos um item deve existir no documento
- Campos numéricos são convertidos com casas decimais específicas

### Processamento de Valores
- **Quantidade:** Fixa em 1 unidade para serviços de transporte
- **Valores:** Extraídos da prestação de serviço (vRec)
- **Totais:** Replicados para totais do documento e financeiro

### Tributação do CT-e
- **ICMS 00:** Base de cálculo, alíquota e valor normal
- **ICMS 20:** Redução de base de cálculo aplicada
- **ICMS 60:** Valores de retenção por substituição tributária
- **ICMS 90:** Tributação com redução e crédito
- **ICMS Outra UF:** Tratamento específico para operações interestaduais

### Gestão de Endereços
- Endereços de saída e entrega são recriados a cada importação
- Municípios são buscados exclusivamente por código IBGE
- Endereços são marcados com flags específicas (saida, entrega)
- Endereço principal permanece inalterado

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| eaa01 | Documento fiscal com dados importados | Eaa01 |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `br.com.multitec.utils.xml` - Processamento de XML

**Estruturas de Dados:**
- `ElementXml` - Parser e navegação de XML
- `TableMap` - Armazenamento de dados JSON

## 📝 Observações Técnicas

- **Processamento:** Síncrono, execução completa do XML
- **Alinhamento:** Utiliza alinhamento "0013" específico para CT-e
- **Flexibilidade:** Suporte a múltiplos regimes de ICMS
- **Precisão:** Arredondamentos específicos por tipo de campo

### Mapeamento de Campos XML
- **Dados Básicos:** Tipo do CT-e, valores da prestação
- **Tributação:** Todos os grupos de ICMS aplicáveis ao CT-e
- **Endereços:** Códigos IBGE de municípios de saída e entrega
- **Complementares:** Informações fiscais e valores da carga

### Tratamento de Dados
- Conversão automática de valores monetários
- Busca otimizada de municípios por código IBGE
- Validação de consistência de dados municipais
- Preservação de dados originais do XML

### Integração com Sistema
- Atualização em tempo real do documento fiscal
- Gestão dinâmica de endereços de transporte
- Compatibilidade com estrutura de itens de serviço
- Manutenção de relacionamentos entre entidades