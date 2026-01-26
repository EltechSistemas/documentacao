# SRF_CalculoItem_DevolucaoDeVenda

📖 Descrição  
Fórmula responsável por calcular e atualizar valores fiscais de um item de devolução de venda (documento de entrada). Inclui ajuste de CFOP, cálculo de impostos (IPI, ICMS, PIS, COFINS) e aplicação de regras de reforma tributária (CBS/IBS), atualizando campos JSON do item.

🎯 Finalidade
- Ajustar CFOP conforme operação e destino (dentro/fora do estado).
- Calcular totais do item (total, totDoc e totFinanc).
- Calcular impostos: IPI, ICMS, PIS e COFINS.
- Aplicar regras da reforma tributária (CBS/IBS) quando aplicável.
- Atualizar campos JSON do item e persistir alterações.

👥 Público-Alvo
- Fiscal
- Contabilidade
- Auditoria
- Desenvolvedores do sistema SRF

⚙️ Parâmetros/Configurações  
| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| eaa0103 | Eaa0103 | Sim | Item do documento a ser processado |
| eaa01 | Eaa01 | Sim | Documento associado ao item |
| aac10 | Aac10 | Sim | Empresa ativa |
| abm01 | Abm01 | Sim | Item de cadastro |
| abm0101 | Abm0101 | Sim | Configuração do item por empresa |
| abm12 | Abm12 | Sim | Configuração fiscal do item |
| abm13 | Abm13 | Não | Dados comerciais do item |
| abb01 | Abb01 | Sim | Central do documento |
| abb10 | Abb10 | Não | Operação comercial |
| abe01 | Abe01 | Sim | Entidade do documento |
| ufEnt / ufEmpr | Aag02 | Não | UF do destinatário e da empresa |
| municipioEnt / municipioEmpr | Aag0201 | Não | Município do destinatário e da empresa |
| jsonEaa0103 | TableMap | Sim | JSON do item (Eaa0103) |
| jsonAbm0101 | TableMap | Não | JSON de configuração do item |
| jsonAbm1001 | TableMap | Não | JSON de estado do item |
| jsonAbm1003_Ent_Item | TableMap | Não | JSON do item por entidade |
| jsonAbe01 | TableMap | Não | JSON da entidade |
| jsonAag02Ent / jsonAag02Empr | TableMap | Não | JSON de UF |
| jsonAag0201Ent | TableMap | Não | JSON de município |

📊 Estrutura de Processamento

### Inicialização
- Recupera o item (`eaa0103`) e o documento (`eaa01`).
- Verifica se o documento é de entrada (PCD).
- Carrega dados da entidade, endereço principal e localização (UF/município).
- Carrega empresa ativa e configurações do item.
- Valida existência da configuração fiscal do item (`abm12`).
- Carrega CST/CFOP/NCM e outros dados fiscais.
- Carrega JSONs do item, estado, entidade e configurações.

### Cálculo do Item
- Ajusta CFOP conforme operação e estado (dentro/fora).
- Calcula total do item (quantidade * unitário).
- Ajusta quantidade de uso (conversão de unidade de venda para estoque).
- Calcula peso bruto e líquido (se configurado).
- Aplica reforma tributária (CBS/IBS).
- Calcula impostos:

#### IPI
- Calcula base e alíquota.
- Ajusta CST conforme tipo de item e regime tributário.
- Calcula valor do IPI e valores isentos/outras.

#### ICMS
- Ajusta CST conforme IVA, redução de base, operação e regime especial.
- Define alíquota de ICMS conforme UF e configurações.
- Calcula base de ICMS e valor do ICMS.
- Calcula ICMS ST quando aplicável.

#### PIS
- Calcula base e alíquota.
- Aplica regras de CST (tributado ou não).

#### COFINS
- Calcula base e alíquota.
- Aplica regras de CST (tributado ou não).

### Total do Documento
- Calcula total do documento considerando IPI, frete, seguro, outras despesas, ICMS ST e desconto.
- Ajusta total financeiro igual ao total do documento.

⚠️ Regras de Negócio
- Fórmula só pode ser usada em documentos de entrada.
- Se não existir endereço principal da entidade, gera exceção.
- Se não existir configuração fiscal do item, gera exceção.
- Ajustes de CST (IPI/ICMS/PIS/COFINS) validam obrigatoriedade de alíquota e campos necessários.
- CST inválido gera exceção.
- CFOP ajustado conforme IVA, estado e tipo do item.
- Regime especial da entidade força CST de ICMS para “00”.

🎨 Saídas/Retornos  
| Tipo | Descrição | Formato |
|---|---|---|
| Item atualizado | Atualiza campos do item com valores calculados | Objeto `Eaa0103` com JSON atualizado |

🔧 Dependências
- Bibliotecas:
    - `sam.server.samdev.formula.FormulaBase`
    - `sam.dicdados.FormulaTipo`
    - `br.com.multitec.utils.ValidacaoException`
    - `br.com.multitec.utils.collections.TableMap`
    - `br.com.multiorm.criteria.criterion.Criterions`
- Entidades:
    - `Eaa01`, `Eaa0102`, `Eaa0103`, `Eaa0101`
    - `Aac10`, `Abe01`, `Abb01`, `Abb10`, `Abm01`, `Abm0101`, `Abm10`, `Abm12`, `Abm13`, `Abm1301`, `Abm1001`, `Abm1003`
    - `Aag01`, `Aag02`, `Aag0201`
    - `Aaj07`, `Aaj10`, `Aaj11`, `Aaj12`, `Aaj13`, `Aaj14`, `Aaj15`

📝 Observações Técnicas
- Campos JSON acessados e atualizados via `TableMap`.
- Aplicação de reforma tributária é dinâmica (executa métodos conforme flags do JSON `Aaj07`).
- Base de cálculo e alíquotas podem ser substituídas por valores do cadastro do item, entidade ou estado.
- Valores são arredondados conforme regras do sistema (ex.: 2 casas decimais para totais).

