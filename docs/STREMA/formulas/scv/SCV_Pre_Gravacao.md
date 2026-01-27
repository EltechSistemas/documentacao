# SCV_Pre_Gravacao

📖 **Descrição**  
Fórmula para realizar a validação e bloqueio de condições de pagamento para a gravação de dados, verificando inconsistências como títulos a vencer, limite de crédito excedido e restrições financeiras, antes de permitir a gravação de uma entidade no sistema.

🎯 **Finalidade**  
Garantir que apenas dados consistentes sejam gravados no sistema, bloqueando entidades com inconsistências financeiras como títulos vencidos ou limite de crédito excedido.

👥 **Público-Alvo**
- Departamento Financeiro
- Controle de Crédito
- Equipe de Suporte

📊 **Dados e Fontes**  
**Tabelas Principais:**
- `Aac10` - Informações de entidades
- `Aae20` - Configurações de pagamento
- `Aag02` - Condições de pagamento
- `Aag0201` - Detalhes de pagamento
- `Abb01` - Cabeçalhos de documentos
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe0101` - Relacionamentos de entidades
- `Abe30` - Condições de pagamento
- `Daa01` - Títulos a receber
- `Eaa01` - Documentos de contas a pagar

⚙️ **Parâmetros da Fórmula**

| Parâmetro       | Tipo    | Obrigatório | Descrição                              |
|-----------------|---------|-------------|----------------------------------------|
| eaa01           | Objeto  | Sim         | Objeto representando o documento de contas a pagar |
| validaInconsistencia | Boolean | Sim       | Flag que indica se há inconsistências a serem validadas |
| gravar          | Inteiro | Sim         | Flag de controle para indicar se o documento pode ser gravado |

🔄 **Fluxo do Processo**
1. **Verificação de Condições de Pagamento**  
   O código começa verificando se a condição de pagamento está corretamente informada, e se não, interrompe o processo.
2. **Validação de Inconsistências**  
   Verifica as inconsistências financeiras, como títulos a vencer, limite de crédito excedido ou restrições financeiras. Se encontrado, o documento é bloqueado para gravação.
3. **Execução das Validações**
   - **Validação de Títulos Vencidos:** Verifica se existem títulos a vencer.
   - **Limite de Crédito Excedido:** Verifica se o total financeiro excede o limite de crédito da entidade.
   - **Restrição Financeira:** Verifica se a entidade possui restrições financeiras.
4. **Resultado Final**  
   Se todas as validações passarem, o documento é desbloqueado para gravação. Caso contrário, o bloqueio permanece ativo.

⚠️ **Regras de Negócio**

- **Validações:**
   - Se `eaa01cp` (condição de pagamento) não estiver informado, o processo é interrompido.
   - O bloqueio da entidade ocorre quando:
      - Existem títulos vencidos a receber.
      - O limite de crédito da entidade foi excedido.
      - A entidade tem restrições financeiras.

- **Formatação de Dados:**
   - As mensagens de bloqueio são armazenadas na lista `eaa0107s`, que contém os detalhes das inconsistências encontradas.
   - O valor de bloqueio (`eaa01bloqueado`) é definido como `1` quando o bloqueio é necessário.

🔧 **Métodos Principais**
- `obterTipoFormula()`  
  Retorna o tipo de fórmula `SCV_SRF_PRE_GRAVACAO`.

- `executar()`  
  Método principal que executa as validações e controle de gravação do documento.

- **Métodos de Validação (comentados no código):**
   - `validarInconsistenciaTituloAreceberVencido()`
   - `validarInconsistenciaLimiteCreditoExcedido()`
   - `validarInconsistenciaRestricao()`
   - Métodos auxiliares para verificar condições de títulos vencidos, limite de crédito e restrições financeiras.

🔧 **Dependências**  
**Bibliotecas:**
- `br.com.multiorm` - ORM para manipulação de dados no banco
- `sam.model.entities.*` - Entidades do sistema para manipulação de dados financeiros

**Entidades:**
- `FormulaBase` - Classe base para fórmulas
- `TableMap` - Mapeamento de tabelas e dados JSON
- `Eaa01`, `Abe30`, `Abe01`, etc. - Entidades específicas do sistema de contas a pagar e crédito

📝 **Observações Técnicas**
- As validações de inconsistência são feitas por meio de consultas SQL otimizadas para identificar registros vencidos ou com valores financeiros inconsistentes.
- As inconsistências são armazenadas no objeto `eaa0107s`, que contém a mensagem e o identificador do bloqueio.
