# SRF_CalculoItem_Saida

## 📖 Descrição
Fórmula responsável pelo cálculo completo de itens em documentos de saída no contexto do SCV/SRF, contemplando regras comerciais, fiscais e tributárias. A fórmula executa a apuração de valores do item, impostos (ICMS, IPI, PIS, COFINS), comissões, descontos, tabela de preços, CFOP, CSTs, partilhas interestaduais, Zona Franca de Manaus, FCI e demais regras fiscais aplicáveis à operação.

## 🎯 Finalidade
Automatizar e padronizar o cálculo de itens em documentos de saída, garantindo conformidade fiscal, correta formação de preços, aplicação de regras tributárias complexas e geração de valores financeiros e fiscais utilizados na emissão de documentos fiscais e controles internos.

## 👥 Público-Alvo
- Departamento Fiscal
- Departamento Contábil
- Departamento Comercial
- Equipe de Faturamento
- Suporte Técnico / Operacional

## 📊 Dados e Fontes

### Tabelas Principais
- Eaa01 – Documento SCV
- Eaa0103 – Itens do documento
- Eaa0102 – Dados gerais do documento
- Abb01 – Central de documentos
- Abb10 – Operação comercial
- Abd01 – PCD
- Abe01 – Entidade
- Abe30 – Condição de pagamento
- Abe40 – Tabela de preço
- Abe4001 – Itens da tabela de preço
- Abm01 – Cadastro de itens
- Abm0101 – Configuração do item por empresa
- Abm10 / Abm1001 / Abm1003 – Valores do item
- Abm12 – Configuração fiscal do item
- Abm13 / Abm1301 – Dados comerciais e fatores de conversão
- Abg01 – NCM
- Aaj10 – CST ICMS
- Aaj11 – CST IPI
- Aaj12 – CST PIS
- Aaj13 – CST COFINS
- Aaj14 – CSOSN
- Aaj15 – CFOP
- Aag01 – País
- Aag02 – Estado (UF)
- Aag0201 – Município

### Entidades Envolvidas
- Documento e Item (Eaa01 / Eaa0103)
- Empresa (Aac10)
- Entidade Destinatária (Abe01)
- Configurações fiscais, comerciais e tributárias do item

## ⚙️ Parâmetros da Fórmula

| Parâmetro     | Tipo     | Obrigatório | Descrição |
|---------------|----------|-------------|-----------|
| eaa0103       | Objeto   | Sim         | Item do documento de saída a ser calculado |
| eaa01         | Objeto   | Sim         | Documento ao qual o item pertence |
| jsonEaa0103   | TableMap | Sim         | Estrutura de campos livres utilizada para armazenar valores calculados |
| Sessão ORM    | Contexto | Sim         | Sessão ativa para carregamento das entidades relacionadas |

## 🔄 Fluxo do Processo

1. **Validação Inicial**
    - Recupera o item do documento.
    - Garante que o documento seja de saída.
    - Valida tipo de entidade e regras fiscais básicas.

2. **Carregamento de Dados**
    - Carrega dados do documento, empresa, entidade, endereço, item, tabela de preço e configurações fiscais.
    - Recupera informações de UF, município e país.

3. **Aplicação da Tabela de Preço**
    - Define valor unitário conforme tabela configurada.
    - Calcula descontos máximos e taxa de comissão.

4. **Cálculos Comerciais**
    - Calcula total do item, quantidades, peso bruto e líquido, volume e comissão.

5. **Cálculos Fiscais**
    - Determina CFOP e CSTs (ICMS, IPI, PIS, COFINS).
    - Apura bases de cálculo, alíquotas, valores de impostos e reduções.
    - Aplica regras de ICMS ST, partilha interestadual, FCP e diferencial de alíquota.

6. **Regras Especiais**
    - Zona Franca de Manaus, Área de Livre Comércio e Amazônia Ocidental.
    - Suspensão e isenção de impostos conforme legislação.
    - Ajustes automáticos de CFOP e CST.

7. **Cálculos Finais**
    - Calcula total do documento, valor financeiro e valores aproximados de impostos.
    - Atualiza os campos livres do item e retorna o resultado ao documento.

## ⚠️ Regras de Negócio
- A fórmula só pode ser utilizada em documentos de saída.
- Itens devem possuir configuração fiscal válida.
- Regras de ICMS variam conforme UF, tipo de operação e tipo de entidade.
- Aplicação automática de ICMS ST, redução de base e partilha interestadual.
- Tratamento específico para Zona Franca de Manaus e Amazônia Ocidental.
- CSTs inválidos interrompem o processamento com exceção.
- Tabela de preço influencia valor unitário, desconto e comissão.

## 🔧 Métodos Principais
- **obterTipoFormula()**  
  Retorna o tipo de fórmula `SCV_SRF_ITEM_DO_DOCUMENTO`.

- **executar()**  
  Método principal responsável por orquestrar todo o carregamento de dados, validações e cálculos do item.

- **setarValorTabelaPreco()**  
  Define valores unitários, descontos máximos e taxa de comissão com base na tabela de preços.

- **calcularItem()**  
  Realiza todos os cálculos comerciais, fiscais, tributários e financeiros do item.

## 🔧 Dependências

### Bibliotecas
- br.com.multitec.utils.collections.TableMap
- br.com.multitec.utils.ValidacaoException
- br.com.multiorm.criteria

### Framework / Sistema
- FormulaBase
- FormulaTipo
- Session ORM SAM

### Entidades de Negócio
- Eaa01, Eaa0103, Abb01, Abm01, Abe01, Aaj*, Abm*, Aag*

## 📝 Observações Técnicas
- A fórmula possui alto acoplamento com regras fiscais brasileiras.
- Utiliza extensivamente campos livres (JSON) para armazenar valores calculados.
- O processamento é sensível à configuração correta de item, NCM, CST e CFOP.
- Exceções interrompem o fluxo quando há inconsistências fiscais.
- Implementa regras legais vigentes como diferencial de alíquota (DIFAL) e FCI.
