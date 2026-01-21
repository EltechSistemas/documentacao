# SGT_Sintegra

## 📄 Descrição
A fórmula **SGT_Sintegra** é responsável pela geração do arquivo **SINTEGRA** a partir dos dados fiscais da empresa, consolidando informações de documentos fiscais, itens, inventário, GNRE, exportações e dados cadastrais, conforme layout oficial do SINTEGRA.

O processamento percorre diversos tipos de registros (10, 11, 50, 51, 53, 54, 55, 70, 74, 75, 85, 88 e 90), compondo um arquivo texto estruturado, pronto para entrega aos fiscos estaduais.

---

## 🎯 Finalidade
- Gerar o arquivo SINTEGRA conforme legislação vigente
- Consolidar dados fiscais de entradas, saídas, inventário e obrigações acessórias
- Atender fiscalizações e obrigações legais estaduais
- Padronizar a escrituração fiscal em layout SINTEGRA

---

## 👥 Público-Alvo
- Área Fiscal
- Contabilidade
- Compliance Tributário
- Suporte Fiscal / ERP

---

## 📊 Dados e Fontes
- **Empresa:** `Aac10`
- **Documentos Fiscais:** `Eaa01`, `Abb01`, `Aah01`
- **Itens de Documentos:** `Eaa0103`
- **Endereços:** `Eaa0101`, `Eaa0102`
- **Inventário:** `Bcb11`
- **GNRE:** `Edd01`
- **Itens / Produtos:** `Abm01`, `Abm0101`, `Abm10`
- **NCM:** `Abg01`
- **Unidade de Medida:** `Aam06`
- **Transportadores e Veículos:** `Abe01`, `Aah20`
- **Tabelas auxiliares fiscais:** `Aaj03`, `Aaj04`, `Aaj15`

As informações são obtidas via **Session ORM** e consultas SQL customizadas.

---

## 🔢 Parâmetros do Processo
- `dataInicial` — Data inicial do período
- `dataFinal` — Data final do período
- `finalidade` — Finalidade do arquivo SINTEGRA
- `natOper` — Natureza da operação
- `aag02s` — Lista de UFs envolvidas no processamento

---

## 📤 Saídas do Processo
- Arquivo texto SINTEGRA (`TextFile`)
- Conteúdo disponibilizado na variável de retorno:
   - `dadosArquivo`

---

## 🔄 Fluxo do Processo
1. Inicialização dos parâmetros e contexto da empresa
2. Seleção do alinhamento do arquivo
3. Geração sequencial dos registros:
   - **Tipo 10 / 11:** Dados cadastrais da empresa
   - **Tipo 50 / 51 / 53:** Documentos fiscais
   - **Tipo 54:** Itens dos documentos e despesas acessórias
   - **Tipo 55:** GNRE
   - **Tipo 70:** Documentos específicos por modelo
   - **Tipo 74:** Inventário
   - **Tipo 75:** Cadastro de produtos
   - **Tipo 85:** Exportações
   - **Tipo 88:** Complementos (C, D, E, M, T)
   - **Tipo 90:** Totalizadores e encerramento
4. Totalização dos registros
5. Finalização do arquivo SINTEGRA

---

## 📜 Regras de Negócio
- Apenas documentos dentro do período informado são considerados
- Documentos cancelados são desconsiderados
- Quantidades e valores são normalizados conforme layout SINTEGRA
- Valores monetários são convertidos para centavos
- Inscrição Estadual “ISENTO/ISENTA” é tratada como literal
- Inventário considera mês e ano da data final
- Caso não existam registros Tipo 88, é gerado o **Tipo 88M**
- Totalizadores do Tipo 90 refletem todos os registros gerados

---

## ⚠️ Inconsistências
- Documentos sem JSON fiscal recebem valores zerados
- Itens sem NCM ou valores fiscais são tratados com defaults
- Caso o totalizador exceda o limite, o Tipo 90 é dividido em múltiplos registros
- UFs “EX” ignoram número de inscrição

---

## 🔗 Dependências
- Framework ORM `multiorm`
- Utilitários `multitec`
- Infraestrutura de fórmulas SAM
- Banco de dados fiscal consistente
- Layout SINTEGRA vigente

---

## 🛠️ Observações Técnicas
- Implementa `FormulaBase`
- Utiliza intensivamente `TableMap` para composição dinâmica
- Possui controle de cancelamento de processo
- Geração do arquivo é sequencial e sensível à ordem
- Métodos auxiliares centralizam regras fiscais e formatações
- Código preparado para grandes volumes de dados
