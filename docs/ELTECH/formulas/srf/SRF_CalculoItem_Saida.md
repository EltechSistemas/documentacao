# SRF_CalculoItem_Saida

📖 Descrição
Fórmula responsável pelo processamento fiscal de saída, incluindo precificação automática, apuração de impostos tradicionais (ICMS, IPI, ST) e a nova estrutura de tributação da **Reforma Tributária (CBS e IBS)**.

🎯 Finalidade
Centralizar a inteligência fiscal de faturamento, garantindo que além dos impostos atuais, as alíquotas de CBS (Federal) e IBS (Estadual/Municipal) sejam calculadas conforme a classificação tributária do item e localidade.

👥 Público-Alvo
- Departamento Fiscal / Faturamento
- TI / Desenvolvedores de Regras de Negócio
- Controladoria

📊 Dados e Fontes
Tabelas Adicionais (Reforma):
- Aaj07 - Classificação Tributária CBS/IBS
- Aag0201 - Alíquotas de IBS por Município
- Aag02 - Alíquotas de CBS por Estado

⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| eaa0103 | Eaa0103 | Sim | Objeto do item do documento de saída |

🔄 Fluxo do Processo
1. **Precificação e Impostos Tradicionais**: Executa a busca de preço em tabela e calcula IPI e ICMS (conforme regras padrão de saída).
2. **Identificação da Classificação CBS/IBS**:
    - Busca a regra na entidade `Aaj07` associada ao item ou ao registro principal.
3. **Validação de Exigibilidade**:
    - Se `exige_tributacao = 1`, inicia a montagem das bases de cálculo CBS/IBS.
4. **Cálculo de Alíquotas Efetivas**:
    - Aplica os percentuais de redução (`perc_red_cbs` e `perc_red_ibs`) sobre as alíquotas nominais para chegar à alíquota efetiva.
5. **Processamento Dinâmico**:
    - O sistema percorre o JSON da classificação tributária e executa métodos específicos (ex: monofásica, crédito presumido) se as chaves estiverem ativas.
6. **Totalização da Reforma**:
    - Calcula os valores finais de CBS, IBS Estadual e IBS Municipal.

⚠️ Regras de Negócio - Reforma Tributária


Exigibilidade
- Caso a classificação tributária (`Aaj07`) não exija tributação, todos os campos de CBS e IBS são zerados automaticamente.

Cálculo de Alíquota Efetiva
- **CBS Efetiva**: $AlíquotaCBS - (AlíquotaCBS \times \frac{PercReduçãoCBS}{100})$
- **IBS Efetiva**: $AlíquotaIBS - (AlíquotaIBS \times \frac{PercReduçãoIBS}{100})$

Base de Cálculo
- A base de cálculo para CBS e IBS (`cbs_ibs_bc`) é, por padrão, o **Valor Total do Item** (`eaa0103total`).

Composição do IBS
- O valor total do IBS é a somatória do **IBS Municipal** (`vlr_ibsmun`) com o **IBS Estadual** (`vlr_ibsuf`).

🔧 Métodos da Reforma Tributária
aplicarReformaTributaria()
Orquestra a leitura da classe tributária e decide se haverá incidência de novos impostos.

red_bc_aliq()
Método matemático que subtrai o percentual de redução das alíquotas de CBS, IBS Municipal e IBS Estadual.

Metódos Dinâmicos (Gatilhos JSON)
- `monofasica_cbsibs()`: Tratamento para regimes monofásicos.
- `cred_presumido()`: Cálculo de créditos presumidos.
- `dif_cbsibs()`: Tratamento de diferimento.

📊 Estrutura de Saída (JSON eaa0103json - Reforma)
| Campo | Descrição |
| :--- | :--- |
| cbs_ibs_bc | Base de cálculo unificada para CBS e IBS |
| aliq_efet_cbs | Alíquota final de CBS após reduções |
| aliq_efet_ibs | Alíquota final de IBS Estadual após reduções |
| vlr_cbs | Valor monetário da CBS |
| vlr_ibs | Valor total do IBS (Estadual + Municipal) |
| perc_red_cbs/ibs | Percentuais de redução aplicados |

📝 Observações Técnicas
- A fórmula utiliza reflexão de métodos (`"$campo"()`) para executar lógicas fiscais específicas baseadas em flags ativas no cadastro da Classificação Tributária.
- Garante integridade ao utilizar `getBigDecimal_Zero()` para evitar erros em campos não preenchidos nas tabelas de UF e Município.