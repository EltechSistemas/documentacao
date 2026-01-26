# SRF_CalculoItem_Saida

📖 **Descrição**  
Fórmula responsável por calcular e atualizar valores fiscais de um item de saída (documento de saída). Inclui ajuste de CFOP, cálculo de impostos (IPI, ICMS, PIS, COFINS), aplicação de regras de reforma tributária (CBS/IBS), cálculo de descontos especiais (Zona Franca/Amazônia Ocidental) e geração de valores aproximados de impostos para venda ao consumidor final. Atualiza campos JSON do item.

🎯 **Finalidade**
- Ajustar CFOP conforme operação, estado e tipo de item.
- Calcular totais do item (total, totDoc e totFinanc).
- Calcular impostos: IPI, ICMS, PIS e COFINS.
- Aplicar regras da reforma tributária (CBS/IBS) quando aplicável.
- Aplicar regras especiais para Zona Franca / Área de Livre Comércio e Amazônia Ocidental.
- Calcular diferencial de alíquota interestadual para não contribuintes e pessoa física.
- Atualizar campos JSON do item e persistir alterações.

👥 **Público-Alvo**
- Fiscal
- Contabilidade
- Auditoria
- Desenvolvedores do sistema SRF

⚙️ **Parâmetros/Configurações**  
| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| eaa0103 | Eaa0103 | Sim | Item do documento a ser processado |
| eaa01 | Eaa01 | Sim | Documento associado ao item |
| aac10 | Aac10 | Sim | Empresa ativa |
| abm01 | Abm01 | Sim | Item de cadastro |
| abm0101 | Abm0101 | Sim | Configuração do item por empresa |
| abm10 | Abm10 | Não | Valores do item (geral) |
| abm1001 | Abm1001 | Não | Valores do item por estado |
| abm1003 | Abm1003 | Não | Valores do item por entidade |
| abm12 | Abm12 | Sim | Configuração fiscal do item |
| abm13 | Abm13 | Não | Dados comerciais do item |
| abm1301 | Abm1301 | Não | Fatores de conversão de unidade de compra para estoque |
| abb01 | Abb01 | Sim | Central do documento |
| abb10 | Abb10 | Não | Operação comercial |
| abe01 | Abe01 | Sim | Entidade do documento |
| abd01 | Abd01 | Sim | PCD (tipo de documento) |
| aag01 | Aag01 | Não | País da entidade |
| ufEnt / ufEmpr | Aag02 | Não | UF do destinatário e da empresa |
| municipioEnt / municipioEmpr | Aag0201 | Não | Município do destinatário e da empresa |
| jsonEaa0103 | TableMap | Sim | JSON do item (Eaa0103) |
| jsonAbm0101 | TableMap | Não | JSON de configuração do item |
| jsonAbm1001_UF_Item | TableMap | Não | JSON de estado do item |
| jsonAbm1003_Ent_Item | TableMap | Não | JSON do item por entidade |
| jsonAbe01 | TableMap | Não | JSON da entidade |
| jsonAag02Ent / jsonAag02Empr | TableMap | Não | JSON de UF |
| jsonAag0201Ent | TableMap | Não | JSON de município |
| jsonAac10 | TableMap | Não | JSON da empresa |
| jsonAaj07 | TableMap | Não | JSON de regras de reforma tributária |

📊 **Estrutura de Processamento**

### Inicialização
- Recupera o item (`eaa0103`) e o documento (`eaa01`).
- Valida que o documento é de saída (PCD).
- Recupera dados da entidade, endereço principal e localização (UF/município).
- Carrega empresa ativa e configurações do item.
- Valida existência da configuração fiscal do item (`abm12`).
- Carrega CST/CFOP/NCM e outros dados fiscais.
- Carrega JSONs do item, estado, entidade e configurações.
- Executa `calcularItem()`.

### Cálculo do Item
- Determina operação comercial e se é dentro ou fora do estado.
- Ajusta CFOP conforme operação, tipo de item, estado e presença de IVA.
- Calcula total do item (quantidade * unitário).
- Ajusta quantidade de uso (conversão de unidade de venda para estoque).
- Calcula volume e peso bruto/líquido conforme configurações do item.
- Calcula desconto incondicional (se informado).
- Aplica reforma tributária (CBS/IBS).

#### IPI
- Calcula base (total + frete + seguro + outras despesas).
- Busca alíquota do NCM.
- Ajusta CST conforme alíquota informada.
- Calcula valor do IPI para CST 50.
- Ajusta campos para CST 51/53/54/55/99 e CST 52.
- Valida CST.

#### ICMS
- Ajusta CST conforme IVA, redução, operação, regime especial e estado.
- Define alíquota de ICMS considerando UF da entidade, UF da empresa, e configurações do item/entidade.
- Calcula base de ICMS (valor do item + frete + seguro + outras despesas - desconto).
- Adiciona IPI à base quando destinatário não é contribuinte.
- Calcula ICMS conforme CST:
    - CST 00: tributado integralmente
    - CST 10: com ICMS ST
    - CST 20: redução de base
    - CST 30/40: isento/não tributado
    - CST 41/50/51: outras operações
    - CST 60: ICMS ST cobrado anteriormente (ajusta CFOP para 5405/6403 se aplicável)
    - CST 70: redução de base + ICMS ST
    - CST 90: outras (com ou sem redução)
- Valida CST.

#### PIS
- Calcula base (valor do item + frete + seguro + outras - desconto - ICMS).
- Busca alíquota no cadastro do item.
- Calcula PIS para CST 01/02.
- Valida e trata CST 03 (não suportado).
- Zera valores para CST 04/05/06/07/08/09/49.
- Valida CST.

#### COFINS
- Calcula base (valor do item + frete + seguro + outras - desconto - ICMS).
- Busca alíquota no cadastro do item.
- Calcula COFINS para CST 01/02.
- Valida e trata CST 03 (não suportado).
- Zera valores para CST 04/05/06/07/08/09/49.
- Valida CST.

### Zona Franca / Área de Livre Comércio e Amazônia Ocidental
- Zera valores de ICMS ZF e demais campos antes do cálculo.
- Se Amazônia Ocidental (alc=3):
    - Define CST IPI 55
    - Zera base e alíquota de IPI
    - Calcula IPI isento (total + frete + seguro + outras despesas)
- Se Zona Franca/Área Livre Comércio (alc=1 ou 2):
    - Aplica alíquota de ICMS ZF conforme UF
    - Calcula base e desconto de ICMS ZF
    - Zera valores de ICMS normal e ajusta isento
    - Ajusta CFOP (6110 para revenda, 6109 para produto acabado)
    - Ajusta CST ICMS para 040
    - Suspende IPI (IPI isento)
    - Se regime tributário diferente de Lucro Real, aplica CST 06 para PIS/COFINS (alíquota zero)

### Total do Documento
- Calcula total do documento:  
  `Total Item + IPI + Frete + Seguro + Outras Despesas + ICMS ST - Desconto`
- Arredonda para 2 casas.
- Define total financeiro (zero se item de retorno).

### Outras Informações
- Ajusta classificação de receita (PIS/COFINS) para determinados CFOPs.
- Para itens retornados (CFOP 902), zera volume e peso.

### Impostos Aproximados para Consumidor Final
- Se operação de venda (operacao=1) e consumidor final ou pessoa física, calcula valores aproximados:
    - Imposto federal e estadual com base em alíquotas do NCM
    - Soma para valor aproximado total de impostos

### Diferencial de Alíquota Interestadual (a partir de 01/01/2016)
- Aplicável para venda a pessoa física ou não contribuinte, fora do estado e país Brasil (1058).
- Define % de partilha conforme ano (2016-2019+).
- Calcula ICMS devido ao estado destino e origem, e valor de FCP se aplicável.

⚠️ **Regras de Negócio**
- Fórmula só pode ser usada em documentos de saída.
- Se o documento for de entrada, gera exceção.
- Se não existir endereço principal da entidade no documento, gera exceção.
- Se não existir configuração fiscal do item, gera exceção.
- Ajustes de CST (IPI/ICMS/PIS/COFINS) validam obrigatoriedade de alíquota e campos necessários.
- CST inválido gera exceção.
- Ajustes de CFOP conforme tipo de item, estado, operação e IVA.
- Operações em Zona Franca/Amazônia Ocidental alteram CFOP, CST e impostos conforme regras.
- Regime especial da entidade força CST de ICMS para “00”.
- Para venda a consumidor final, gera valor aproximado de impostos com base no NCM.

🎨 **Saídas/Retornos**  
| Tipo | Descrição | Formato |
|---|---|---|
| Item atualizado | Atualiza campos do item com valores calculados | Objeto `Eaa0103` com JSON atualizado |

🔧 **Dependências**
- Bibliotecas:
    - `sam.server.samdev.formula.FormulaBase`
    - `sam.dicdados.FormulaTipo`
    - `br.com.multitec.utils.ValidacaoException`
    - `br.com.multitec.utils.collections.TableMap`
    - `br.com.multiorm.criteria.criterion.Criterions`
- Entidades:
    - `Eaa01`, `Eaa0101`, `Eaa0102`, `Eaa0103`
    - `Aac10`, `Aag01`, `Aag02`, `Aag0201`, `Aaj07`, `Aaj10`, `Aaj11`, `Aaj12`, `Aaj13`, `Aaj14`, `Aaj15`, `Aam06`
    - `Abb01`, `Abb10`, `Abd01`, `Abe01`, `Abg01`, `Abm01`, `Abm0101`, `Abm10`, `Abm1001`, `Abm1003`, `Abm12`, `Abm13`, `Abm1301`

📝 **Observações Técnicas**
- Campos JSON acessados e atualizados via `TableMap`.
- Aplicação de reforma tributária é dinâmica (executa métodos conforme flags do JSON `Aaj07`).
- Base de cálculo e alíquotas podem ser substituídas por valores do cadastro do item, entidade ou estado.
- Valores são arredondados conforme regras do sistema (ex.: 2 casas decimais para totais).
- O cálculo do diferencial de alíquota usa o ano da emissão do documento e regras de partilha progressiva (2016-2019+).
