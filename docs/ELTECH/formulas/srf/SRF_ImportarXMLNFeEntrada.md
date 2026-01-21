# SRF_ImportarXMLNFeEntrada

## 📖 Descrição
Fórmula responsável pela importação e interpretação de arquivos XML de **NF-e de Entrada**, realizando a leitura completa das informações fiscais, tributárias, logísticas, financeiras e de estoque, vinculando os dados ao documento fiscal (`Eaa01`) e seus itens (`Eaa0103`).

## 🎯 Finalidade
Processar XMLs de NF-e de entrada, garantindo a correta integração das informações do XML com o sistema, incluindo impostos, lotes, tributações, totais, duplicatas, transporte e dados de protocolo da NF-e.

## 👥 Público-Alvo
- Fiscal
- Contabilidade
- Compras
- Estoque
- Financeiro
- Controladoria
- TI / Desenvolvimento

**Tipo de Fórmula:**
- `IMPORTAR_ARQUIVOS_XML_DE_NFE`

## 📊 Dados e Fontes
**Entidades Principais:**
- `Eaa01` – Documento fiscal
- `Eaa0103` – Itens do documento
- `Eaa01033` – Referência entre itens
- `Eaa01038` – Lotes do item
- `Eaa0113` – Duplicatas financeiras
- `Abm01 / Abm0101` – Item e configurações
- `Abm13 / Abm1301` – Comercial e fatores de conversão
- `Abd01 / Abd04` – PCD e configurações fiscais/estoque
- `Aam04 / Aam06` – Status e unidades
- `Abg01` – NCM
- `Aaj07` – Classificação tributária IBS/CBS

**Fontes de Dados:**
- XML de NF-e (layout SEFAZ)
- Parâmetros internos do SAM / SRF

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|----------|------|-------------|----------|
| eaa01 | Eaa01 | Sim | Documento fiscal de entrada |
| elementXmlNfe | ElementXml | Sim | XML da NF-e processada |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|------|-----------|------|
| eaa01 | Documento fiscal atualizado | Eaa01 |
| eaa0103 | Itens atualizados com dados fiscais | Eaa0103 |
| eaa01038 | Lotes associados aos itens | List |
| eaa0113 | Duplicatas financeiras | List |
| JSONs | Dados fiscais e tributários mapeados | TableMap |

## 🔄 Fluxo do Processo

1. **Inicialização**
  - Obtém documento (`Eaa01`) e XML
  - Seleciona alinhamento fiscal
  - Valida estrutura básica do XML (`nfeProc`)

2. **Leitura da Estrutura da NF-e**
  - Identifica `NFe`, `infNFe` e `ide`
  - Inicializa JSON do documento

3. **Processamento dos Itens**
  - Percorre tags `<det>`
  - Localiza item pelo número sequencial (`nItem`)
  - Associa notas fiscais referenciadas quando aplicável

4. **Dados do Produto**
  - Unidade tributável
  - NCM
  - Quantidades, valores unitários, frete, seguro e descontos
  - Informações adicionais do item

5. **Controle de Lotes**
  - Processa tag `<rastro>`
  - Aplica fator de conversão de unidade
  - Define lote, validade, fabricação e série
  - Configura status e controles de estoque conforme PCD

6. **Processamento de Tributos**
  - ICMS (todos os grupos)
  - IPI
  - II
  - PIS / PIS ST
  - COFINS / COFINS ST
  - ISSQN
  - IBS / CBS (Reforma Tributária)

7. **Totais da NF-e**
  - Totais fiscais (ICMS, ST, IPI, PIS, COFINS, ISS)
  - Totais IBS/CBS
  - Atualiza totais financeiros do documento

8. **Transporte**
  - Quantidade de volumes
  - Peso líquido e bruto
  - Replica dados para o primeiro item

9. **Informações Adicionais**
  - Observações do fisco
  - Observações do contribuinte

10. **Protocolo da NF-e**
  - Chave de acesso
  - Data e hora de autorização
  - Protocolo
  - Código e descrição do status

11. **Duplicatas Financeiras**
  - Processa cobranças (`<cobr>`)
  - Cria parcelas financeiras (`Eaa0113`)
  - Define vencimento, valor e tipo financeiro

## ⚠️ Regras de Negócio

### Validações Obrigatórias
- XML deve estar no padrão `nfeProc`
- Documento e itens devem existir previamente
- Item é localizado pelo número sequencial do XML
- Campos só são gravados se existirem no JSON configurado

### Regras Fiscais
- Valores monetários são convertidos conforme casas decimais
- Tributos seguem o grupo informado no XML
- Suporte completo aos grupos de ICMS e Simples Nacional
- Compatível com IBS/CBS

### Estoque e Lotes
- Controle de estoque depende do PCD
- Status e controles herdados da configuração
- Quantidade ajustada por fator de conversão

### Financeiro
- Duplicatas criadas apenas se SCF ativo no PCD
- Parcelas criadas com base no XML (`dup`)

## 🎨 Inconsistências Possíveis

| Situação | Descrição |
|--------|-----------|
| XML inválido | Estrutura fora do padrão |
| Item não encontrado | Sequência do XML não localizada |
| Configuração ausente | Falta de parâmetros de estoque ou fiscal |
| Dados incompletos | Tags obrigatórias não informadas |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – ORM e critérios
- `multitec.utils` – Utilitários, JSON e XML
- `java.time` – Datas e horários

**Infraestrutura:**
- Parâmetros fiscais configurados
- Estrutura de JSON fiscal previamente definida

## 📝 Observações Técnicas

- Importação extremamente detalhada e completa
- Compatível com NF-e de entrada modelo 55
- Suporte a múltiplos cenários tributários
- Integração direta com estoque, fiscal e financeiro
- Uso intensivo de JSON para flexibilidade fiscal
- Preparada para evolução da Reforma Tributária (IBS/CBS)
- Fórmula tolerante a dados opcionais do XML
