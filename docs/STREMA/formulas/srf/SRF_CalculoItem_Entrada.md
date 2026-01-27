# SRF_CalculoItem_Entrada
📖 Descrição
Fórmula para o processamento, cálculo de impostos (ICMS, IPI, PIS, COFINS) e determinação automática de CFOP para itens de documentos de entrada no sistema SRF.

🎯 Finalidade
Automatizar a apuração fiscal dos itens, garantindo que as bases de cálculo, alíquotas e códigos fiscais (CST/CFOP) sejam aplicados corretamente com base na origem, destino, tipo de item e regime tributário dos envolvidos.

👥 Público-Alvo
- Departamento Fiscal
- Recebimento / Almoxarifado
- TI / Suporte de Sistemas
- Controladoria

📊 Dados e Fontes
Tabelas Principais:
- Eaa0103 - Itens do documento de entrada
- Eaa01 - Cabeçalho do documento
- Aac10 - Cadastro da empresa/filial
- Abe01 - Entidades (Fornecedores)
- Abm0101 - Configuração do item por empresa
- Abm12 - Parâmetros fiscais do item
- Abm1001 - Valores e IVA por UF
- Aaj15 - Cadastro de CFOP

Entidades Envolvidas:
- Eaa0103 - Item do Documento
- Eaa01 - Documento Central
- Aac10 - Empresa Ativa
- Abe01 - Fornecedor
- Abm01 - Cadastro de Produto
- Aaj10/11/12/13 - Tabelas de CST (ICMS, IPI, PIS, COFINS)

⚙️ Parâmetros da Fórmula
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| eaa0103 | Eaa0103 | Sim | O objeto do item do documento a ser calculado |

🔄 Fluxo do Processo
1. Inicialização
   - Recupera o item (Eaa0103) e o documento pai (Eaa01).
   - Valida se o documento é de ENTRADA (rejeita documentos de saída).
2. Busca de Dados Relacionados
   - Identifica a empresa logada e sua localização (UF/Município).
   - Localiza o fornecedor e seu endereço principal no documento.
   - Carrega as configurações fiscais do item por empresa (Abm0101) e por UF (Abm1001).
3. Determinação de Localidade
   - Compara a UF da Empresa com a UF do Fornecedor para definir se a operação é Interna ou Interestadual.
4. Ajuste de CFOP
   - Define o primeiro dígito (1 ou 2) baseado na localidade.
   - Define o sufixo baseado no tipo do item (Revenda, Matéria-Prima, Uso/Consumo) e presença de IVA (ST).
5. Cálculo de Impostos
   - IPI: Define base de cálculo somando despesas e aplica CST conforme regime tributário.
   - ICMS: Determina alíquota (Item, Estado ou Importação), calcula Redução de BC e apura ICMS ST (Retenção) se houver IVA.
   - PIS/COFINS: Inicia a montagem da base de cálculo com fretes e despesas.
6. Atualização do Item
   - Grava os resultados nos campos nativos e no objeto JSON (eaa0103json).

⚠️ Regras de Negócio
Validações
- Bloqueia processamento se o documento não for de entrada (Tipo Saída no PCD).
- Exige endereço principal da entidade cadastrado no documento.
- Exige configuração fiscal (Abm12) e fator de conversão de unidade (Abm1301) ativos para o item.

Regras de CFOP (Entrada)
- Uso e Consumo: Força CFOP x556 e ajusta CST de IPI para '02'.
- Revenda com ST: Se houver IVA de compra, altera o sufixo para x403.
- Matéria-Prima: Utiliza sufixo x101 ou x401 (se houver ST).

Cálculo de Base (BC)
- BC ICMS/PIS/COFINS = Valor Total + Frete + Seguro + Outras Despesas - Desconto Incondicional.
- BC IPI = Valor Total + Frete + Seguro + Outras Despesas.

🔧 Métodos Principais
executar()
Método de entrada que realiza as buscas no banco de dados, validações de segurança e orquestra a chamada do cálculo.

calcularItem()
Executa a lógica matemática e fiscal: compara UFs, seleciona CFOP, calcula bases de impostos e realiza arredondamentos.

📊 Estrutura de Saída
- eaa0103total: Valor bruto calculado (Qtd * Unitário).
- eaa0103qtUso: Quantidade convertida pelo fator de unidade de medida.
- eaa0103json: Mapa contendo:
   - icm_bc / icm_icm: Base e Valor do ICMS.
   - st_bc / st_icm: Base e Valor do ICMS ST.
   - ipi_bc / ipi_obs: Base e Valor (em observação se houver) do IPI.

🔧 Dependências
Bibliotecas:
- sam.server.samdev - Base da fórmula.
- br.com.multitec.utils - Validações e exceções.
- java.time.format - Datas (se aplicável).
  Entidades:
- FormulaBase - Classe pai.
- TableMap - Estrutura para campos dinâmicos JSON.

📝 Observações Técnicas
- A fórmula utiliza round(2) para valores financeiros e round(4) para pesos/quantidades.
- Suporte para alíquota de 4% automática em casos de fornecedores do tipo "Importação" (tp_empresa = 3).
- Tratamento de campos nulos utilizando getBigDecimal_Zero() para evitar NullPointerException.