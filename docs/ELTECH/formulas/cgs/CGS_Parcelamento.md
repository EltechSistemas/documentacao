# CGS - Parcelamento (Eltech)

## 📖 Descrição
Fórmula para cálculo e geração de parcelas de condições de pagamento, considerando datas de vencimento, ajustes por dias da semana, feriados, valores mínimos por parcela e configurações específicas do módulo CGS.

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
- `Abb01` - Documentos da central (data de emissão)

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtBase | LocalDate | Sim | Data base para cálculo das parcelas |
| abe30id | Long | Sim | ID da condição de pagamento |
| valor | BigDecimal | Sim | Valor total a ser parcelado |
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
- Aplicação de ajustes complementares (Abe3002)

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
- **Dias base**: Ajustes por dia da semana na data base (abe30diasDtBase1-7)
- **Vencimentos fixos**: Datas específicas por parcela (dt_vcto_fixo)
- **Dias do mês**: Vencimento em dia específico do mês (diavcto)
- **Ajustes complementares**: Regras por faixa de dias (Abe3002)
- **Mês referência**: Ajustes para mês corrente ou seguinte

### Cálculo Financeiro
- **Juros**: Percentual aplicado sobre o valor da parcela
- **Multa**: Percentual ou valor fixo (vlr_fixo)
- **Desconto**: Percentual com limite temporal (dias_dtlimite)
- **Encargos**: Valores adicionais fixos

### Validações de Parcelas
- **Valor mínimo**: Configurável por condição de pagamento (abe30vmpValor)
- **Opções**: 
  - 0 - Agrupar em uma parcela quando valor mínimo não for atingido
  - 1 - Validar valor mínimo e interromper se não atingido
- **Documentos financeiros**: 
  - Tipo 1 - Gera na data de vencimento
  - Tipo 2 - Gera na data de emissão

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de parcelamento.

### `obterDiasAdicionaisAData(LocalDate data, Abe30 abe30, int qualData)`
Calcula dias adicionais baseado no dia da semana:
- 0: Dias para data base (abe30diasDtBase1-7)
- 1: Dias para vencimento nominal (abe30diasVctoN1-7)

### `buscarCondicaoPagamentoPorId(Long abe30id)`
Busca a condição de pagamento pelo ID.

### `buscarParcelasPeloIdCondicaoPagamento(Long abe30id)`
Busca as parcelas configuradas para a condição de pagamento.

### `buscarDiaComplementarPeloIdCondicaoPagamento(Long abe30id)`
Busca ajustes complementares de dias (Abe3002).

### `montarParcelaDto(LocalDate data, BigDecimal valor, Integer docFinanc, Abe3001 abe3001, TableMap cposLivres)`
Monta o DTO da parcela com todos os dados calculados.

## 📊 Estrutura de Saída

**ParcelaDto:**
- `vctoN` - Data de vencimento nominal
- `valor` - Valor da parcela
- `criaDoc` - Tipo de documento financeiro (1 ou 2)
- `abf15id` - ID do portador
- `abf16id` - ID da operação
- `abf01id` - ID do banco
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

**Módulo:** CGS (Condições Gerais do Sistema)

## 📝 Observações Técnicas

### Tratamento de Datas
- Ajustes automáticos para finais de semana
- Tratamento especial para fevereiro (meses com menos dias)
- Suporte a vencimentos fixos e variáveis
- Ajustes por faixas de dias (inicial-final)

### Campos Livres (JSON)
- `juros` - Valor calculado de juros
- `multa` - Valor calculado de multa
- `encargos` - Encargos adicionais
- `desconto` - Valor de desconto por antecipação
- `dtLimiteDesc` - Data limite para desconto
- `nomevcto` - Nome personalizado do vencimento

### Validações de Negócio
- Interrupção se valor mínimo não for atingido (opção 1)
- Cálculo preciso de saldos para evitar diferenças
- Suporte a condições complexas de pagamento
- Opção de agrupamento em parcela única quando valor mínimo não é atingido

### JSON da Abe30 (Configurações)
- `juros` - Percentual de juros
- `multa` - Percentual de multa
- `vlr_fixo` - Valor fixo de multa
- `vlr_desc_tx` - Taxa de desconto
- `dias_dtlimite` - Dias limite para desconto
- `encargos` - Encargos adicionais
- `desconto` - Desconto padrão

### JSON da Abe3001 (Parcelas)
- `vcto_nome` - Nome do vencimento
- `refparc` - Referência da parcela
- `diavcto` - Dia específico do mês
- `dt_vcto_fixo` - Data fixa de vencimento

### JSON da Abe3002 (Ajustes Complementares)
- `diai` - Dia inicial da faixa
- `diaf` - Dia final da faixa
- `diavcto` - Dia de vencimento
- `refmes` - Referência de mês (0=corrente, 1=seguinte)
- `dia_inicial` - Dia inicial para ajuste
- `dia_final` - Dia final para ajuste
- `dia_data` - Dia de ajuste
- `mes_data` - Mês de ajuste (0=corrente, 1=seguinte)

### Lógica de Agrupamento em Parcela Única
- Quando `abe30vmpOpcao = 0` e valor da parcela < valor mínimo
- Soma todas as parcelas em uma única
- Usa data da primeira parcela (ou emissão se docFinan = 2)
- Mantém campos livres da primeira parcela
- Aplica validação de valor mínimo ao total agrupado

---

**Última Alteração:** 01/12/2025 às 15:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Condição de Pagamento  
**Versão:** 1.0