# SCF_Movimento.md

## 📖 Descrição
Sistema de cálculo de movimento financeiro para o SCF (Sistema de Controle Financeiro) da Linhasita, responsável pelo processamento de juros, multas, encargos e descontos em títulos financeiros.

## 🎯 Finalidade
Calcular automaticamente os valores líquidos de títulos financeiros considerando encargos, descontos e condições de pagamento, garantindo a correta apuração financeira.

## 👥 Público-Alvo
- Departamento Financeiro
- Tesouraria
- Crédito e Cobrança
- Contabilidade

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SCF_Movimento` - Cálculo de movimento financeiro

**Localização:** `eltech/formulas/scf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA1001` - Movimentos financeiros
- `DAA01` - Títulos a receber/pagar

**Entidades Envolvidas:**
- `Daa1001` - Movimento financeiro
- `Daa01` - Título principal
- `SCFService` - Serviço de cálculos financeiros

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| daa1001 | Daa1001 | Sim | Movimento financeiro a ser calculado |

## 📋 Campos Calculados

| Campo | Descrição | Tipo |
|-------|-----------|------|
| daa1001liquido | Valor líquido do movimento | BigDecimal |
| jurosq | Juros quitados | BigDecimal |
| multaq | Multa quitada | BigDecimal |
| encargosq | Encargos quitados | BigDecimal |
| descontoq | Desconto aplicado | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Obtém movimento financeiro (DAA1001)
   - Recupera título principal relacionado (DAA01)
   - Inicializa mapa JSON para campos livres

2. **Cálculo de Dias em Atraso**
   - Utiliza serviço SCF para calcular dias de atraso
   - Aplica juros somente se houver atraso
   - Aplica multa somente se houver atraso

3. **Processamento de Encargos**
   - Encargos aplicados independente de atraso
   - Considera valor fixo cadastrado

4. **Aplicação de Descontos**
   - Verifica data limite para desconto
   - Aplica desconto somente se pagamento dentro do prazo
   - Desconsidera desconto se fora do prazo

5. **Ajuste para Pagamento Parcial**
   - Calcula fator proporcional para pagamentos parciais
   - Aplica proporção aos valores de JMED
   - Mantém consistência nos cálculos

6. **Cálculo do Valor Líquido**
   - Soma valor principal com encargos quitados
   - Subtrai descontos aplicados
   - Arredondamento preciso para 2 casas decimais

## ⚠️ Regras de Negócio

### Condições de Aplicação
- **Juros:** Aplicados apenas quando há dias em atraso (diasAtraso > 0)
- **Multa:** Aplicada apenas quando há dias em atraso
- **Encargos:** Aplicados independente da situação
- **Desconto:** Aplicado apenas se pagamento até data limite

### Cálculos Específicos
- **Juros:** `juros * diasAtraso` (acumulativo)
- **Multa:** Valor fixo conforme cadastro
- **Desconto:** Valor fixo, condicionado à data
- **Pagamento Parcial:** Proporcionalidade nos encargos

### Tratamento de Valores
- **Desconto Quitado:** Convertido para negativo (desconto é redução)
- **Arredondamento:** Sempre para 2 casas decimais
- **Valor Líquido:** Soma algébrica de todos os componentes

## 🔧 Dependências

**Serviços:**
- `SCFService` - Serviço especializado em cálculos financeiros
- `calculaDiasDeAtraso()` - Método para cálculo preciso de dias

**Bibliotecas:**
- `java.time` - Manipulação de datas e períodos
- `multitec.utils` - Utilitários e cálculos

## 📝 Observações Técnicas

- Suporte a pagamentos parciais com proporcionalidade
- Cálculo automático de dias de atraso
- Validação de datas para aplicação de descontos
- Campos livres (JSON) para flexibilidade nas configurações
- Arredondamento preciso em todas as etapas
- Integração com serviço especializado SCF
- Tratamento de valores nulos e casos extremos