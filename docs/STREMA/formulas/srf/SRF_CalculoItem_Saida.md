# SCE_CalculoTributacao (Trecho de Código)

📖 Descrição  
Trecho responsável pelo cálculo de tributos e ajustes fiscais em itens de documento de saída, incluindo ICMS, PIS, COFINS, IPI, Zona Franca/Amazônia Ocidental, FCI, Diferencial de Alíquota, e aplicação da reforma tributária (CBS/IBS). Realiza validações de CST, ajustes de base de cálculo e alíquotas, e atualiza valores finais do documento.

🎯 Finalidade
- Calcular ICMS, PIS, COFINS, IPI e demais tributos conforme CST e regras fiscais.
- Ajustar CFOP e CST em operações específicas (ex.: ST, Zona Franca, Amazônia Ocidental).
- Gerar valores aproximados de impostos para consumidor final.
- Calcular diferencial de alíquota interestadual (partilha).
- Aplicar regras da reforma tributária (CBS/IBS).
- Atualizar valores totais do documento e financeiro.

👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria
- Desenvolvedores de ERP/Gestão Fiscal

⚙️ Parâmetros/Configurações  
| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| eaa0103 | Entidade | Sim | Item do documento (Eaa0103) |
| jsonEaa0103 | JSON | Sim | Campos calculados e valores auxiliares do item |
| aaj10_cstIcms | Entidade | Depende | CST de ICMS |
| aaj12_cstPis | Entidade | Depende | CST de PIS |
| aaj13_cstCof | Entidade | Depende | CST de COFINS |
| aaj11_cstIpi | Entidade | Depende | CST de IPI |
| jsonAbm1001_UF_Item | JSON | Sim | Configuração fiscal do item por UF |
| jsonAag02Ent | JSON | Sim | Configuração tributária da empresa |
| jsonAag0201Ent | JSON | Sim | Configuração tributária da empresa por UF |
| abm12 | Entidade | Sim | Grupo de inventário / produto |
| abg01 | Entidade | Não | NCM (aliquotas aproximadas) |
| aaj15_cfop | Entidade | Sim | CFOP do item |
| operacao | Integer | Sim | Tipo de operação (ex.: venda) |
| eaa0102 | Entidade | Sim | Cabeçalho do documento (cliente/emitente) |
| eaa01 | Entidade | Sim | Documento fiscal |
| abb10 | Entidade | Sim | Tipo de documento |
| abe01 | Entidade | Sim | Regime tributário |

📊 Estrutura de Processamento

### ICMS (CST 00, 10, 20, 30/40, 41, 50, 51, 60, 70, 90)
- **CST 00 (Tributada integralmente)**
    - Valida alíquota ICMS > 0
    - Calcula ICMS sobre base e zera outros campos (outras/isento).
- **CST 10 (Tributada com ICMS ST)**
    - Valida alíquota ICMS e IVA ST
    - Calcula ICMS e ICMS ST (base com IVA e redução ST se houver)
    - Define alíquota ST (padronizada por UF ou fixada no item)
- **CST 20 (Tributada com redução de BC)**
    - Define % de redução (item ou cadastro)
    - Valida alíquota e % redução
    - Calcula redução da base, ICMS e registra “outras”
- **CST 30 / 40 (Isenta / não tributada)**
    - Zera base e alíquota e marca como isento
- **CST 41, 50, 51 (Nacional não tributada / suspensão / diferimento)**
    - Zera ICMS e marca “outras” com base original
- **CST 60 (Cobrado anteriormente por ST)**
    - Ajusta CFOP para 102/202 e zera ICMS
- **CST 70 (Redução com ST)**
    - Calcula redução, IVA, base ST, ICMS ST e alíquotas
- **CST 90 (Outras)**
    - Se há redução, calcula; caso contrário zera ICMS

- Se CST inválido, lança `ValidacaoException`.

### PIS
- Calcula base (`pis_bc`) = total item + frete + seguro + outras - ICMS.
- Ajusta base para consumidor final/não contribuinte/consumo.
- Define alíquota PIS (pis_aliq).
- Calcula valor PIS para CST 01/02.
- Para CST 03, lança exceção (não contemplado).
- CST 04/05/06/07/08/09/49 zera base e valores.
- Se CST inválido, lança `ValidacaoException`.

### COFINS
- Calcula base (`cofins_bc`) = total item + frete + seguro + outras - ICMS.
- Ajusta base para consumidor final/não contribuinte/consumo.
- Define alíquota COFINS (cofins_aliq).
- Calcula valor COFINS para CST 01/02.
- Para CST 03, lança exceção (não contemplado).
- CST 04/05/06/07/08/09/49 zera base e valores.
- Se CST inválido, lança `ValidacaoException`.

### Zona Franca / Amazônia Ocidental
- Inicializa valores de ICMS ZF.
- Se ALC = 3 (Amazônia Ocidental):
    - Define CST IPI = 55
    - Zera valores de IPI e define isenção com base total.
- Se ALC = 1 ou 2 (Zona Franca / Área Livre Comércio):
    - Aplica alíquota e base ICMS ZF para itens nacionais.
    - Calcula desconto ICMS ZF e zera valores originais de ICMS.
    - Ajusta CFOP conforme tipo de produto (revenda/acabado).
    - Ajusta CST ICMS para 040.
    - Suspensão de IPI (IPI isento e zerado).
    - Para regime tributário diferente de Lucro Real, zera PIS e COFINS (CST 06).

### Total do Documento
- Calcula `totDoc` = total item + IPI + frete + seguro + outras + ICMS ST - desconto.
- Arredonda para 2 casas.
- Define `totFinanc` dependendo de `retInd`.

### Ajustes Adicionais
- Ajusta classificação de receita (PIS/Cofins) para CFOP específicos.
- Para CFOP 902, zera peso e volume.

### Impostos aproximados para consumidor final
- Se operação = 1 e não industrialização:
    - Calcula impostos aproximados (FEDERAL/ESTADUAL) com base no NCM (abg01).
    - Soma valores para `impaprx_vlr`.

### Diferencial de Alíquota (Interestadual)
- Se não contribuinte e CFOP de venda (6101,6102,6122,6251,6124,6125,6107,6949):
    - Define percentual de partilha por ano (2016..2019+).
    - Calcula bases e valores de ICMS UF destino e origem.
    - Aplica FCP se configurado.

### FCI (Valor FCI - Saída)
- Para venda ou transferência (tipo 1 ou 7):
    - Se origem não definida, busca último cálculo de FCI do item.
    - Ajusta CST ICMS conforme CI (classificação de importação).
    - Se CFOP corresponde a venda específica, define valor FCI e quantidade.
    - Se origem definida, ajusta CST ICMS com base no código de origem.

### Reforma Tributária (CBS/IBS)
- Executa `aplicarReformaTributaria()` ao final do cálculo.

⚠️ Regras de Negócio
- CST inválido gera `ValidacaoException`.
- CST 03 de PIS/COFINS não é contemplado (lança exceção).
- Em Zona Franca/Amazônia, ICMS e IPI podem ser zerados e ajustados.
- Operações de consumidor final podem gerar imposto aproximado.
- Diferencial de alíquota só aplica a partir de 2016 e para não contribuintes.

🎨 Saídas/Retornos  
| Tipo | Descrição | Formato |
|---|---|---|
| eaa0103 | Item atualizado com tributos e valores finais | Entidade Eaa0103 |
| jsonEaa0103 | JSON com campos calculados e valores auxiliares | JSON interno |

🔧 Dependências
- Entidades:
    - Aaj10 (CST ICMS), Aaj11 (CST IPI), Aaj12 (CST PIS), Aaj13 (CST COFINS)
    - Aaj15 (CFOP)
    - Abm12, Abm01, Abm1001 (configuração fiscal)
    - Eaa01, Eaa02, Eaa03, Abb10, Abe01
    - Abg01 (NCM)
    - Aag02, Aag0201 (configuração tributária)
    - Eab0101 (FCI)
- Funções auxiliares:
    - `buscarUltimoCalculo()`
    - `aplicarReformaTributaria()`
    - Métodos de cálculo CBS/IBS (ex.: `red_bc_aliq()`)

📝 Observações Técnicas
- Valores fiscais são armazenados e manipulados em `jsonEaa0103` como BigDecimal.
- O cálculo de ICMS ST utiliza IVA e redução de base conforme configuração do item por UF.
- A reforma tributária é aplicada dinamicamente via JSON de configuração (`jsonAaj07`) e execução de métodos por reflexão.
- O método `red_bc_aliq()` ajusta alíquotas efetivas de CBS/IBS conforme percentuais de redução.  
