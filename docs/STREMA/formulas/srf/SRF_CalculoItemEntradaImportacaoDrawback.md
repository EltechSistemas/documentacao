# SRF_CalculoItemEntradaImportacaoDrawback.md

## 📖 Descrição
Sistema de cálculo de itens de entrada por importação e drawback para a Linhasita, responsável pelo processamento de valores fiscais e tributários em documentos de importação.

## 🎯 Finalidade
Realizar o cálculo completo de valores fiscais, tributários e comerciais para itens de entrada via importação e drawback, garantindo a correta apuração de impostos e custos.

## 👥 Público-Alvo
- Departamento Fiscal
- Compras/Importação
- Contabilidade
- Controladoria

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SRF_CalculoItemEntradaImportacaoDrawback` - Cálculo de itens de importação

**Localização:** `strema/formulas/srf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` - Documentos fiscais
- `EAA0103` - Itens de documentos
- `ABB01` - Cabeçalho de documentos
- `ABE01` - Entidades/Clientes
- `ABM01` - Itens/Cadastro de produtos
- `ABM0101` - Configurações de itens por empresa
- `AAC10` - Empresas/Filiais

**Entidades Envolvidas:**
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens de documentos
- `Abb01` - Cabeçalho documentos
- `Abe01` - Entidades
- `Abm01` - Itens
- `Aac10` - Empresas

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa0103 | Eaa0103 | Sim | Item do documento a ser calculado |

## 📋 Campos Calculados

| Campo | Descrição | Tipo |
|-------|-----------|------|
| vlrfcientrada | Valor fiscal da entrada | BigDecimal |
| ii_aliq | Alíquota do Imposto de Importação | BigDecimal |
| vlr_vlme | Valor do volume | BigDecimal |
| vlr_pl | Peso líquido | BigDecimal |
| vlr_pb | Peso bruto | BigDecimal |
| vlr_desc | Valor do desconto | BigDecimal |
| ipi_aliq | Alíquota do IPI | BigDecimal |
| ipi_ipi | Valor do IPI | BigDecimal |
| ipi_bc | Base de cálculo do IPI | BigDecimal |
| vlr_outras | Valor de outras despesas | BigDecimal |
| icm_isento | Valor ICMS isento | BigDecimal |
| custo_aquisicao | Custo de aquisição | BigDecimal |
| cif_imp | Valor CIF da importação | BigDecimal |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
   - Obtém item do documento (EAA0103)
   - Valida existência do documento e entidade

2. **Carregamento de Dados**
   - Documento fiscal (EAA01)
   - Central de documento (ABB01)
   - Entidade/cliente (ABE01)
   - Empresa (AAC10)
   - Configurações do item (ABM0101)

3. **Cálculos Fiscais**
   - Valor total do item
   - Imposto de Importação (II)
   - IPI (Imposto sobre Produtos Industrializados)
   - ICMS (Imposto sobre Circulação de Mercadorias)
   - PIS/COFINS

4. **Cálculos Comerciais**
   - Conversão de quantidades
   - Cálculo de pesos (bruto e líquido)
   - Aplicação de descontos
   - Custo de aquisição

5. **Consolidação de Valores**
   - Total do documento
   - Valor financeiro
   - Custo final da importação

## ⚠️ Regras de Negócio

### Validações
- Item deve ter quantidade comercial maior que zero
- Configuração fiscal do item é obrigatória
- Endereço principal da entidade deve existir no documento
- Empresa deve estar ativa e com município cadastrado

### Cálculos Específicos
- **Imposto de Importação:** Aplicado sobre alíquota específica do item
- **IPI:** Calculado sobre base de cálculo com alíquota do NCM
- **ICMS:** Cálculo considerando regime de tributação
- **PIS/COFINS:** Tratamento diferenciado para operações com exterior

### Conversões
- Quantidade comercial convertida para quantidade de uso
- Quantidade convertida para volume quando aplicável
- Cálculo de pesos baseado no cadastro do item

## 🎨 Saídas Geradas

| Saída | Descrição | Local |
|-------|-----------|--------|
| Campos JSON | Valores calculados no JSON do item | eaa0103.eaa0103json |
| Totais | Valores totais do documento | eaa0103.eaa0103totDoc |
| Valores Fiscais | Bases de cálculo e impostos | Campos específicos |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `multitec.utils` - Utilitários e validações
- `java.math` - Cálculos com BigDecimal

**Serviços:**
- Acesso ao banco de dados
- Gestão de sessões

## 📝 Observações Técnicas

- Implementa cálculos complexos de tributação de importação
- Suporte a múltiplas unidades de medida e conversões
- Integração com cadastro fiscal de itens
- Tratamento de JSON para campos customizados
- Cálculos precisos com arredondamento decimal
- Validações robustas para evitar erros de cálculo