# SRF - Cálculo de Itens de Saída

### 📖 Descrição
Fórmula para processamento fiscal completo de itens em documentos de saída, realizando precificação automática via tabela de preços, determinação dinâmica de CFOP e apuração de impostos tradicionais (ICMS, IPI, ST) em conjunto com a nova estrutura da **Reforma Tributária (CBS e IBS)**.

### 🎯 Finalidade
Automatizar o faturamento de itens, garantindo a aplicação correta de preços e o cálculo exato da carga tributária vigente, incluindo a lógica de alíquotas efetivas, reduções de base e gatilhos dinâmicos da Reforma Tributária.

### 👥 Público-Alvo
- Departamento de Faturamento
- TI / Desenvolvedores de Regras de Negócio
- Departamento Fiscal
- Controladoria

### 📊 Dados e Fontes
**Tabelas Principais:**
- **Eaa0103** - Itens do documento de saída
- **Abm01** - Cadastro de itens (Produtos)
- **Abe4001** - Itens da tabela de preços
- **Abm1001** - Valores do item por UF (ICMS/IVA)
- **Aaj07** - Classificação tributária CBS/IBS (Reforma)
- **Aag02** - Cadastro de Estados (CBS)
- **Aag0201** - Cadastro de Municípios (IBS)

**Entidades Envolvidas:**
- **Eaa01** - Documento (Cabeçalho)
- **Abe01** - Entidade (Cliente)
- **Aac10** - Empresa ativa
- **Abm12** - Dados fiscais do item
- **Abm13** - Dados comerciais e comissões

---

### ⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| **eaa0103** | Eaa0103 | Sim | Item do documento de saída a ser processado |

---

### 🔄 Fluxo do Processo

#### 1. Inicialização e Validações
- Recupera o item (`eaa0103`) e os dados do documento.
- **Validação Crítica:** Impede faturamento se Pessoa Física for marcada como contribuinte de ICMS.
- Verifica se o PCD permite operação de **Saída**.

#### 2. Busca de Preço e Comissões
- Valida vigência da tabela de preços.
- Busca o preço unitário por hierarquia: *Item > Tabela > Condição Pgto > Qtde > Desconto*.
- Atribui as 5 faixas de comissão do item.

#### 3. Inteligência de CFOP e Logística
- Determina se a operação é Interna ou Interestadual.
- Define o CFOP dinamicamente (Venda/Revenda/Consumidor Final).
- Calcula **Pesos (Líquido/Bruto)** e **Volume**.

#### 4. Reforma Tributária (CBS/IBS)
- Localiza a classificação tributária (`Aaj07`) vinculada ao item.
- **Base de Cálculo:** Define a BC unificada baseada no total do item.
- **Alíquotas Efetivas:** Executa o método `red_bc_aliq()` para aplicar os percentuais de redução sobre as alíquotas nominais de CBS, IBS Estadual e IBS Municipal.
- **Gatilhos Dinâmicos:** Percorre o JSON da `Aaj07` e executa via reflexão métodos como `monofasica_cbsibs`, `cred_presumido`, entre outros, caso estejam ativos.
- **Totalização:** Calcula os valores finais de `vlr_cbs`, `vlr_ibsuf` e o somatório total de `vlr_ibs`.

#### 5. Impostos Tradicionais (IPI/ICMS)
- **IPI:** Baseado no NCM e CST de IPI.
- **ICMS:** Trata Substituição Tributária (IVA), Reduções de Base e Diferimento de ICMS.

---

### ⚠️ Regras de Negócio

#### Reforma Tributária
- **Exigibilidade:** Se `exige_tributacao` for diferente de 1, todos os valores de CBS/IBS são zerados.
- **Prioridade de Regra:** Busca primeiro a classificação direta do item; se nula, busca a classificação regional.
- **Cálculo Efetivo:** As alíquotas efetivas são calculadas antes da aplicação sobre a base de cálculo.

#### Validações Gerais
- **Arredondamento:** Valores monetários arredondados para 2 casas decimais; Pesos para 4 casas.
- **Falha de Execução:** Métodos dinâmicos da reforma tributária possuem tratamento de exceção (`try-catch`) para evitar interrupção do cálculo em caso de erro em fórmulas específicas.

---

### 🔧 Métodos Principais

#### aplicarReformaTributaria()
Orquestra a incidência da CBS/IBS, define as bases de cálculo e dispara os métodos dinâmicos baseados no cadastro `Aaj07`.

#### red_bc_aliq()
Realiza o cálculo matemático das alíquotas efetivas:
`Aliq. Efetiva = Aliq. Nominal - (Aliq. Nominal * Perc. Redução / 100)`.

#### executar()
Método principal que coordena a precificação, logística e chamadas fiscais (tradicionais e reforma).

---

### 📊 Estrutura de Saída (JSON eaa0103)
- **cbs_ibs_bc**: Base de cálculo para a reforma.
- **aliq_efet_cbs / aliq_efet_ibs**: Alíquotas após reduções.
- **vlr_cbs / vlr_ibs**: Valores calculados de CBS e IBS Total.
- **vlr_pl / vlr_pb**: Peso líquido e bruto totais.

---

### 📝 Observações Técnicas
- **Reflexão Dinâmica:** O uso de `"$campo"()` permite que novas funcionalidades da Reforma Tributária sejam acionadas apenas configurando o JSON da `Aaj07`.
- **Performance:** As alíquotas de CBS e IBS são recuperadas das entidades de UF (`Aag02`) e Município (`Aag0201`) previamente carregadas.