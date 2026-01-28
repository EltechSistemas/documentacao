# SCV_Pre_Gravacao.md

### 📖 Descrição
Fórmula de pré-gravação para o módulo SCV (Sistema de Controle de Vendas) da Strema. Atua como um gatilho de validação e consistência de dados antes da persistência de pedidos ou documentos de venda no banco de dados.

### 🎯 Finalidade
Validar condições obrigatórias para a gravação de documentos, gerenciar bloqueios por inconsistências financeiras (crédito, títulos vencidos) e garantir a integridade das regras de negócio do departamento comercial antes da finalização do registro.

### 👥 Público-Alvo
* Departamento de TI / Desenvolvedores
* Administradores de Dados (DBAs)
* Suporte Técnico de Sistemas

### ⚙️ Configuração
* **Recursos Necessários:** Classe `SCV_Pre_Gravacao`, Motor de fórmulas Multiorm.
* **Localização:** `strema/formulas/scv/`

### 📊 Dados e Fontes
**Tabelas Principais:**
* `EAA01` - Cabeçalho do pedido/documento de venda
* `ABE30` - Condições de pagamento
* `ABE01` - Entidades (Clientes/Fornecedores)
* `DAA01` - Títulos a receber (Financeiro)
* `EAA0107` - Inconsistências do documento

**Entidades Envolvidas:**
* `Eaa01` - Pedido de Venda
* `Abe30` - Condição de Pagamento
* `Abe01` - Cliente
* `Daa01` - Título Financeiro
* `Eaa0107` - Registro de Inconsistência

---

### ⚙️ Parâmetros do Processo
| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| eaa01 | Eaa01 | Sim | Objeto do documento de venda sendo processado |
| gravar | Integer | Sim | Flag que define se o registro deve ser persistido (0-Não, 1-Sim) |
| validaInconsistencia | Boolean | Interno | Flag de controle para sinalizar bloqueio do documento |

### 📋 Saídas do Processo
| Campo | Descrição | Tipo |
| :--- | :--- | :--- |
| eaa01.eaa01bloqueado | Define se o pedido nasce bloqueado (0-Liberado, 1-Bloqueado) | Integer |
| eaa01.eaa0107s | Lista de mensagens de erro/bloqueio anexadas ao documento | List<Eaa0107> |
| gravar | Retorno para o sistema permitindo ou não a operação final | Integer |

---

### 🔄 Fluxo do Processo
1.  **Inicialização:**
    * Recupera o objeto `eaa01` do contexto de execução.
    * Define `validaInconsistencia` como `false` por padrão.
2.  **Validação de Condição de Pagamento:**
    * Verifica se existe uma condição de pagamento associada.
    * Caso nulo, utiliza `interromper()` para travar a execução com mensagem de erro.
3.  **Análise de Operação Comercial (Código 201):**
    * Se a operação for do tipo "201", inicia o ciclo de validações financeiras.
    * Carrega dados da entidade (Cliente) incluindo campos customizados via JSON.
4.  **Validações Financeiras (Regras de Negócio):**
    * Verifica existência de títulos vencidos.
    * Calcula se o limite de crédito foi excedido.
    * Verifica flags de restrição manual no cadastro.
5.  **Finalização:**
    * Atualiza o status de bloqueio (`eaa01bloqueado`) com base nos resultados.
    * Limpa a política de uso temporária (`eaa01psUso`).
    * Define o parâmetro `gravar` para retorno ao core do sistema.

---

### ⚠️ Regras de Negócio
* **Condição de Pagamento:** É um campo mandatório. A ausência deste dado impede qualquer gravação.
* **Bloqueio Automático:** Sempre que uma inconsistência financeira é detectada, o documento é marcado como bloqueado (`1`), exigindo liberação posterior.
* **Operação 201:** Regras rigorosas de crédito e inadimplência são aplicadas especificamente para este código de operação comercial.
* **Títulos Vencidos:** O sistema bloqueia pedidos se o cliente possuir qualquer título (`DAA01`) com data de vencimento inferior à data atual e que não esteja quitado.
* **Limite de Crédito:** O cálculo considera: `(Pedidos em Aberto + Títulos a Receber + Valor do Pedido Atual)`. Se o total superar o `lim_cred` definido no JSON da entidade, o pedido é bloqueado.

---

### 🔧 Dependências
* **Bibliotecas:** `br.com.multiorm`, `sam.server.samdev.formula.FormulaBase`, `java.time`.
* **Serviços:** Motor de execução de fórmulas SAM (Server Application Manager).

### 📝 Observações Técnicas
* **Otimização de Query:** Utiliza projeções (`addFields`) para cálculos de soma (`SUM`), evitando overhead de memória.
* **Tratamento de JSON:** Faz uso intensivo da classe `TableMap` para acessar dados dinâmicos em `eaa01json` e `abe01json`.
* **Segurança:** O status de bloqueio é garantido como `0` (liberado) apenas se nenhuma inconsistência for encontrada durante o fluxo.

---

### 🔄 Métodos Principais
#### obterTipoFormula()
Identifica a fórmula no dicionário de dados como `SCV_SRF_PRE_GRAVACAO`.

#### executar()
Método mestre que coordena a sequência de validações e a atribuição de status ao objeto `Eaa01`.

#### validarInconsistenciaTituloAreceberVencido()
Executa critéria na `DAA01` para somar valores de títulos vencidos não quitados.

#### validarInconsistenciaLimiteCreditoExcedido()
Realiza o balanço financeiro entre o limite contratual do cliente e sua exposição atual no sistema.

#### validarInconsistenciaRestricao()
Valida se o campo `lim_restricao` no JSON da entidade está ativo (valor 1).