# SRF_CalculoItem_DevolucaoDeCompra

---

## 📖 Descrição
Esta fórmula calcula os valores fiscais e comerciais de um **item de devolução de compra** no sistema, considerando impostos como ICMS, IPI, PIS, COFINS, CFOP e CSOSN, além de fatores de unidade, pesos e descontos.

A fórmula faz consultas a diversas tabelas do sistema para obter dados de empresa, entidade, item, endereço, NCM, CFOP e CST, permitindo ajustar os valores de acordo com regras tributárias e comerciais.

---

## 🎯 Finalidade
Calcular automaticamente os valores fiscais e comerciais de um item de devolução de compra, garantindo que:
- O CFOP esteja correto conforme estado da empresa e destino.
- Os impostos sejam calculados conforme CST e alíquotas.
- Pesos, quantidades e descontos sejam atualizados.
- Regras de ICMS ST, redução de base e IVA sejam aplicadas corretamente.

---

## 👥 Público-Alvo
- Contadores e analistas fiscais.
- Equipes de controle de estoque e financeiro.
- Desenvolvedores que implementam fórmulas fiscais no sistema SAM.

---

## Tipo de Fórmula
`SCV_SRF_ITEM_DO_DOCUMENTO` — fórmula aplicada a **itens de documento**.

---

## 📊 Dados e Fontes
- **Documento:** `Eaa01`
- **Item do Documento:** `Eaa0103`
- **Dados Gerais do Documento:** `Eaa0102`
- **Endereço Principal da Entidade:** `Eaa0101`
- **Empresa:** `Aac10`
- **Entidade:** `Abe01`
- **Item:** `Abm01` e configurações por empresa `Abm0101`
- **Valores do Item:** `Abm10`, `Abm1001`, `Abm1003`
- **Configuração Fiscal:** `Abm12`
- **Configuração Comercial:** `Abm13`
- **Fatores de Conversão de Unidade:** `Abm1301`
- **Unidade de Medida:** `Aam06`
- **Operação Comercial:** `Abb10`
- **CFOP:** `Aaj15`
- **CST ICMS:** `Aaj10`
- **CST IPI:** `Aaj11`
- **CST PIS:** `Aaj12`
- **CST COFINS:** `Aaj13`
- **CSOSN:** `Aaj14`
- **NCM:** `Abg01`

---

## ⚙️ Parâmetros do Processo
- `eaa0103`: Item do documento (entrada da fórmula)
- Sessão (`getSession()`): utilizada para consultas das entidades
- Empresa ativa (`obterEmpresaAtiva()`): referência para configuração do item

---

## 📋 Saídas do Processo
- `eaa0103.eaa0103json`: objeto `TableMap` atualizado com todos os valores calculados, incluindo:
    - Total do item (`total`)
    - Quantidade utilizada (`qtUso`)
    - Peso líquido e bruto (`vlr_pl`, `vlr_pb`)
    - Desconto (`vlr_desc`)
    - Base e valor de ICMS, ICMS ST, IPI (`icm_bc`, `icm_icm`, `st_bc`, `st_icm`, `ipi_bc`, `ipi_ipi`)
    - Alíquotas aplicadas (`icm_aliq`, `st_aliq`, `ipi_aliq`)
    - Valores isentos ou não tributados (`icm_isento`, `ipi_isento`)

---

## 🔄 Fluxo do Processo
1. Recupera o **item do documento** (`eaa0103`) e o **documento** (`eaa01`).
2. Obtém dados do **PCD** (`Abd01`) e valida se o documento é de saída.
3. Obtém dados da **entidade** e do **endereço principal**.
4. Recupera dados da **empresa** e endereço da empresa.
5. Obtém o **item** (`Abm01`) e suas configurações fiscais (`Abm12`), comerciais (`Abm13`) e por estado/entidade (`Abm1001`, `Abm1003`).
6. Recupera CST, CSOSN, CFOP e NCM.
7. Inicializa campos livres (`TableMap`) para armazenar valores temporários.
8. Calcula valores do item:
    - Total (`qtComl * unit`)
    - Quantidade de uso ajustada por fator de conversão
    - Pesos líquidos e brutos
    - Desconto
9. Aplica **reforma tributária**.
10. Calcula **IPI** de acordo com CST e alíquota.
11. Calcula **ICMS**, incluindo:
    - Base de cálculo
    - Redução da base
    - ICMS próprio e ST
    - Ajuste para regimes especiais
12. Atualiza o objeto `eaa0103` com todos os valores calculados.

---

## ⚠️ Regras de Negócio
- CFOP ajustado de acordo com operação (venda/revenda) e estado de destino.
- CST ICMS ajustado considerando IVA, redução de BC, ST, operação e regime da entidade.
- CST IPI validado com cálculo de base e alíquota.
- Se o documento não for de saída (`abd01.abd01es == 0`), a fórmula lança exceção.
- Peso e quantidade podem ser ajustados por unidade comercial ou fatores de conversão.
- Desconto incondicional aplicado sobre o total do item.
- ICMS ST e redução de BC calculados conforme configurações do estado do item e da entidade.

---

## 🎨 Inconsistências Possíveis
- CFOP não encontrado no sistema.
- CST inválido ou alíquota de ICMS/IPI não informada.
- Endereço principal da entidade não definido.
- Configuração fiscal do item ausente (`Abm12`).
- Valores de IVA, redução de ICMS ST ou ST BC não informados.

---

## 🔧 Dependências
- Sessão Hibernate (`getSession()`)
- Entidades SAM (`Eaa01`, `Eaa0103`, `Abm01`, `Abm0101`, `Abm10`, `Abm12`, `Abm13`, `Abm1301`, `Aac10`, `Abe01`, `Abb10`, `Aaj10-Aaj15`, etc.)
- Método `obterEmpresaAtiva()`
- Classe base `FormulaBase`
- Classe utilitária `TableMap`
- Critérios de consulta (`Criterions`)

---

## 📝 Observações Técnicas
- O cálculo do ICMS e IPI segue regras complexas do sistema fiscal brasileiro, incluindo:
    - Ajustes por IVA (ICMS ST)
    - Redução de base de cálculo
    - CSTs especiais
- Campos JSON (`TableMap`) são utilizados para armazenar valores temporários e cálculos intermediários.
- Toda exceção de validação lança `ValidacaoException`.
- Unidade de medida comercial pode alterar a quantidade utilizada (`eaa0103qtUso`) via `Abm1301`.
- A fórmula depende fortemente de dados fiscais por estado, entidade e item para determinar corretamente alíquotas e CST.

