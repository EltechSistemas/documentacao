# SRF_CalculoItem_Saida

## Descrição
Fórmula responsável por calcular os valores do item em documentos de **saída**, incluindo impostos (ICMS, IPI, PIS, COFINS), ajustes de CFOP, valores totais, conversões de quantidade, pesos, volumes, e regras específicas de **Zona Franca/Amazônia Ocidental** e **Diferencial de Alíquota (2016+)**. Também aplica regras da **Reforma Tributária (CBS/IBS)** quando aplicável.

---

## Finalidade
Automatizar o cálculo fiscal e comercial do item no documento, definindo:
- CFOP correto conforme operação, tipo do item e destinatário.
- CST de ICMS/IPI/PIS/COFINS conforme regras e configurações do item.
- Bases de cálculo, alíquotas e valores de impostos.
- Total do documento e valor financeiro.
- Regras especiais (Zona Franca, Amazônia, diferencial de alíquota, reforma tributária).

---

## Público Alvo
- Usuários fiscais/contábeis que utilizam o sistema para emissão de documentos de saída.
- Desenvolvedores que implementam regras fiscais e tributárias no ERP.
- Analistas de tributação e compliance.

---

## Parâmetros / Variáveis de Entrada (principais)

| Nome | Tipo | Origem | Descrição |
|------|------|--------|-----------|
| `eaa0103` | `Eaa0103` | Parâmetro da fórmula | Item do documento |
| `eaa01` | `Eaa01` | Documento | Dados do documento |
| `abb01` | `Abb01` | Central de documento | Centro/Filial |
| `abe01` | `Abe01` | Entidade | Cliente/Fornecedor |
| `abm01` | `Abm01` | Item | Cadastro do item |
| `abm0101` | `Abm0101` | Config item x empresa | Configuração comercial/fiscal |
| `abm12` | `Abm12` | Config fiscal | Tipo fiscal do item |
| `abm13` | `Abm13` | Config comercial | Unidade de venda |
| `abg01` | `Abg01` | NCM | Dados fiscais do NCM |
| `aaj10_cstIcms` | `Aaj10` | CST ICMS | CST do item |
| `aaj11_cstIpi` | `Aaj11` | CST IPI | CST do item |
| `aaj12_cstPis` | `Aaj12` | CST PIS | CST do item |
| `aaj13_cstCof` | `Aaj13` | CST COFINS | CST do item |
| `aaj15_cfop` | `Aaj15` | CFOP | CFOP do item |
| `jsonEaa0103` | `TableMap` | Campos livres | Armazena valores calculados e complementares |

---

## Estrutura de Processamento

### 1. Inicialização e validações iniciais
- Obtém item do documento (`eaa0103`) e documento (`eaa01`).
- Valida se a entidade é PF e contribuinte de ICMS → **exceção**.
- Verifica se o documento é de saída (PCD) → **exceção se não for**.
- Busca dados do cliente, empresa, endereço principal e UF/Município.
- Carrega configurações do item (comercial, fiscal, valores, UF e entidade).
- Valida existência do `abm12` e do tipo fiscal.

---

### 2. Ajustes e cálculos iniciais
- Ajusta valor unitário com base na tabela de preço (`abe40` / `abe4001`).
- Define se a operação é interna ou interestadual.
- Ajusta CFOP de acordo com:
    - Tipo de operação (venda/revenda)
    - Tipo do item
    - Contribuinte de ICMS ou não
    - IVA no item
    - Pessoa física x jurídica
- Atualiza CFOP final (com o primeiro dígito 5 ou 6 conforme estado).

---

### 3. Cálculo de valores do item
Quando quantidade e unitário são válidos:
- Total do item
- Qtde de uso (considerando fator de conversão UV → Estoque)
- Volume, peso bruto e líquido
- Desconto incondicional
- Aplica Reforma Tributária (CBS/IBS)

---

### 4. Cálculo de IPI
- Base = total + frete + seguro + outras - desconto
- Alíquota do NCM quando aplicável
- Ajuste de CST (50,51,52,53,54,55,99)
- Calcula valor do IPI conforme CST e alíquota

---

### 5. Cálculo de ICMS
- Determina CST ajustado conforme:
    - IVA, redução de BC, substituição tributária, regime especial, etc.
- Calcula alíquota de ICMS conforme:
    - Estado de destino
    - Configurações do item (UF/Entidade)
    - Cliente final / interno / externo
- Calcula BC ICMS e valores:
    - `icm_icm`, `icm_outras`, `icm_isento`
    - CST 00,10,20,30,40,41,50,51,60,70,90
- Calcula ICMS ST quando aplicável
- Ajusta CFOP em CST 60 quando necessário

---

### 6. Cálculo de PIS
- Base = total + frete + seguro + outras - desconto
- Alíquota do cadastro do item
- CSTs válidos:
    - 01,02 (tributado)
    - 04,05,06,07,08,09,49 (não tributado)
- CST 03 → **exceção (não contemplado)**

---

### 7. Cálculo de COFINS
- Base = total + frete + seguro + outras - desconto
- Alíquota do cadastro do item
- CSTs válidos:
    - 01,02 (tributado)
    - 04,05,06,07,08,09,49 (não tributado)
- CST 03 → **exceção (não contemplado)**

---

### 8. Zona Franca / Amazônia Ocidental
- Se `alc == 3` (Amazônia Ocidental):
    - CST IPI → 55
    - IPI isento
- Se `alc == 1 ou 2` (ZFM/ALC):
    - Ajusta CFOP para 6110 ou 6109 conforme tipo do item
    - CST ICMS → 040
    - Calcula desconto ICMS ZF e zera ICMS normal
    - Suspensão de IPI

---

### 9. Total do documento e financeiro
- Total documento:
    - `total item + IPI + frete + seguro + outras + ICMS ST - desconto`
- Valor financeiro:
    - `totDoc` quando `retInd == 0`, caso contrário `0`

---

### 10. Outras regras
- Ajuste de classificação de receita para CFOPs específicos (901, 905, 910…)
- Itens retornados (CFOP 902) → zera volume e peso
- Geração de valor aproximado de impostos para venda a consumidor final

---

### 11. Diferencial de Alíquota (Interestadual)
- Aplicável quando:
    - Pessoa física ou não contribuinte
    - Fora do estado
    - Brasil (código país 1058)
    - Operação de venda
    - Ano >= 2016
- Calcula:
    - `partilha_aliq`
    - `icms_ufdest`, `uforig_icms`, `icms_fcp`

---

### 12. Reforma Tributária (CBS/IBS)
- Executa se `exige_tributacao == 1`
- Define:
    - `cbs_ibs_bc`
    - `cbs_aliq`
    - `ibs_uf_aliq`
- Executa métodos dinâmicos conforme flags do JSON (ex.: `red_bc_aliq()`)
- Calcula:
    - `vlr_ibsuf`, `vlr_ibs`, `vlr_cbs`

---

## Regras de Negócio / Validações
- PF contribuinte de ICMS → erro
- Documento não é de saída → erro
- Endereço principal não encontrado → erro
- Tipo fiscal do item não informado → erro
- CSTs e alíquotas obrigatórias conforme CST → erro
- CFOP inválido → erro
- CST ICMS inválido → erro
- CST PIS/COFINS inválidos → erro
- CST 03 PIS/COFINS → não suportado (erro)

---

## Saídas / Retornos
- `eaa0103` atualizado com:
    - `cfop`, `cstIcms`, `cstIpi`, `cstPis`, `cstCofins`
    - Valores fiscais calculados (BC, alíquotas, impostos)
    - `totDoc`, `totFinanc`
    - Campos livres atualizados em `eaa0103json`

---

## Dependências
- `FormulaBase` (framework do SAM)
- Entidades do modelo: `Eaa01`, `Eaa0103`, `Abm01`, `Abm10`, `Abm12`, `Abm13`, `Abe01`, etc.
- `TableMap` (campos livres)
- `Criterions` (queries)
- `ValidacaoException` (validações)

---

## Observações Técnicas
- O cálculo de `unit` via custo unitário está comentado e pode ser utilizado futuramente.
- Uso de campos livres (`json*`) para armazenar dados adicionais e cálculos.
- A fórmula é **altamente dependente** de configurações fiscais do item, estado e entidade.
- Método `aplicarReformaTributaria()` usa execução dinâmica de métodos via strings.
