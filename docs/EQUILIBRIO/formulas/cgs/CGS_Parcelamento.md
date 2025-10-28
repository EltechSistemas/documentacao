# CGS - Parcelamento (Equilibrio)

## 📖 Descrição
Fórmula para cálculo e geração de parcelas de condições de pagamento, considerando datas de vencimento, ajustes por dias da semana, feriados, valores mínimos por parcela e configurações específicas do módulo Equilibrio.

## 🎯 Finalidade
Calcular automaticamente as parcelas de uma condição de pagamento, aplicando regras de vencimento, descontos, juros, multas e validações de valores mínimos por parcela.

## 👥 Público-Alvo
- Departamento Financeiro
- Faturamento
- Crédito e Cobrança

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abe30` - Condições de pagamento
- `Abe3001` - Parcelas da condição de pagamento
- `Abe3002` - Dias complementares (ajustes de vencimento)
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtBase | LocalDate | Sim | Data base para cálculo das parcelas |
| abe30id | Long | Sim | ID da condição de pagamento |
| valor | BigDecimal | Sim | Valor total a ser parcelado |
| abe01id | Long | Não | ID da entidade (cliente) |
| eaa01 | Eaa01 | Não | Documento fiscal relacionado |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento da condição de pagamento (Abe30)
- Definição da data de emissão (do documento ou atual)

### 2. **Cálculo dos Ajustes de Data**
- Aplicação de dias adicionais conforme dia da semana da data base
- Cálculo de vencimentos nominais para cada parcela
- Ajustes por vencimentos fixos e dias específicos
- Tratamento de meses com menos dias (ex: fevereiro)

### 3. **Cálculo dos Valores das Parcelas**
- Distribuição percentual do valor total
- Cálculo do saldo para última parcela
- Aplicação de juros, multas e encargos
- Cálculo de descontos por antecipação

### 4. **Validações e Regras Especiais**
- Verificação de valor mínimo por parcela
- Opção de agrupamento em parcela única
- Cálculo de comissões na última parcela
- Validação de documentos financeiros

## ⚠️ Regras de Negócio

### Configuração de Vencimentos
- **Dias base**: Ajustes por dia da semana na data base
- **Vencimentos fixos**: Datas específicas por parcela
- **Dias do mês**: Vencimento em dia específico do mês
- **Ajustes complementares**: Regras por faixa de dias

### Cálculo Financeiro
- **Juros**: Percentual aplicado sobre o valor da parcela
- **Multa**: Percentual ou valor fixo
- **Desconto**: Percentual com limite temporal
- **Encargos**: Valores adicionais fixos

### Validações de Parcelas
- **Valor mínimo**: Configurável por condição de pagamento
- **Opções**: 0 - Agrupar em uma parcela, 1 - Validar valor mínimo
- **Comissões**: Calculadas apenas na última parcela

### Documentos Financeiros
- **Tipo 1**: Gera documento na data de vencimento
- **Tipo 2**: Gera documento na data de emissão

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de parcelamento.

### `obterDiasAdicionaisAData(LocalDate data, Abe30 abe30, int qualData)`
Calcula dias adicionais baseado no dia da semana:
- 0: Dias para data base
- 1: Dias para vencimento nominal

### `buscarCondicaoPagamentoPorId(Long abe30id)`
Busca a condição de pagamento pelo ID.

### `buscarParcelasPeloIdCondicaoPagamento(Long abe30id)`
Busca as parcelas configuradas para a condição de pagamento.

### `buscarDiaComplementarPeloIdCondicaoPagamento(Long abe30id)`
Busca ajustes complementares de dias.

### `montarParcelaDto(LocalDate data, BigDecimal valor, Integer docFinanc, Abe3001 abe3001, TableMap cposLivres)`
Monta o DTO da parcela com todos os dados calculados.

## 📊 Estrutura de Saída

**ParcelaDto:**
- `vctoN` - Data de vencimento nominal
- `valor` - Valor da parcela
- `criaDoc` - Tipo de documento financeiro (1 ou 2)
- `abf15id` - ID do portador
- `abf16id` - ID da operação
- `abf40id` - ID da forma de pagamento
- `cposLivres` - Campos livres (juros, multa, desconto, etc.)

**Lista de Parcelas:**
- Retornada no parâmetro `listaParcelas`

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data
- `sam.dicdados` - Tipos de fórmula
- `sam.dto.cgs` - DTOs do módulo CGS
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Módulo:** Equilibrio

## 📝 Observações Técnicas

### Tratamento de Datas
- Ajustes automáticos para finais de semana
- Tratamento especial para fevereiro (meses com menos dias)
- Suporte a vencimentos fixos e variáveis

### Campos Livres (JSON)
- `juros` - Valor calculado de juros
- `multa` - Valor calculado de multa
- `encargos` - Encargos adicionais
- `desconto` - Valor de desconto por antecipação
- `dtLimiteDesc` - Data limite para desconto
- `nomevcto` - Nome personalizado do vencimento
- `comissao_parcela` - Comissão na última parcela

### Validações de Negócio
- Interrupção se valor mínimo não for atingido
- Cálculo preciso de saldos para evitar diferenças
- Suporte a condições complexas de pagamento

---

**Última Alteração:** 27/10/2025 às 10:00  
**Autor:** Bruno  
**Tipo:** Fórmula de Condição de Pagamento  
**Versão:** 1.0