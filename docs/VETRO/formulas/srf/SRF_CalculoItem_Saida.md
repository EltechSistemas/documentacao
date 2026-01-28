# SRF_CalculoItem_Saida

## 📖 Descrição
Fórmula responsável pelo cálculo completo dos valores fiscais, comerciais e tributários de um **item de documento de saída (SCV)**. A fórmula realiza a apuração automática de preços, comissões, impostos (ICMS, IPI, PIS, COFINS, ICMS ST), ajustes de CFOP, aplicação de regras fiscais por UF, município, entidade e item, além de contemplar regras especiais como **Zona Franca de Manaus, Área de Livre Comércio, Amazônia Ocidental e Reforma Tributária (CBS/IBS)**.

## 🎯 Finalidade
Garantir o cálculo correto e automatizado dos valores do item em documentos de saída, assegurando conformidade fiscal, consistência comercial e correta formação do total do documento, considerando legislações estaduais, federais e regras específicas do negócio.

## 👥 Público-Alvo
- Departamento Fiscal
- Departamento Contábil
- Departamento Comercial
- Departamento Financeiro
- Equipe de Vendas
- Suporte Operacional
- TI / Sustentação de Sistemas

## 📊 Dados e Fontes

### Tabelas Principais
- Eaa01 – Documento SCV
- Eaa0101 – Endereços do documento
- Eaa0102 – Dados gerais do documento
- Eaa0103 – Itens do documento
- Abb01 – Central do documento
- Abb10 – Operação comercial
- Abd01 – PCD
- Abe01 – Entidade
- Abe0101 – Endereço da entidade
- Abe30 – Condição de pagamento
- Abe40 – Tabela de preços
- Abe4001 – Preço por item
- Abm01 – Cadastro de itens
- Abm0101 – Configuração do item por empresa
- Abm10 – Valores do item
- Abm1001 – Valores do item por UF
- Abm1002 – Valores do item por município
- Abm1003 – Valores do item por entidade
- Abm12 – Configuração fiscal do item
- Abm13 – Configuração comercial do item
- Abm1301 – Fator de conversão de unidades
- Abg01 – NCM
- Aag01 – País
- Aag02 – UF
- Aag0201 – Município
- Aaj07 – Classificação tributária
- Aaj09 – CST CBS/IBS
- Aaj10 – CST ICMS
- Aaj11 – CST IPI
- Aaj12 – CST PIS
- Aaj13 – CST COFINS
- Aaj14 – CSOSN
- Aaj15 – CFOP
- Aac10 – Empresa

### Entidades Envolvidas
- Documento SCV e seus itens
- Entidade (cliente/destinatário)
- Empresa emissora
- Item e suas configurações fiscais/comerciais
- Estados, municípios e país
- Tabelas de preço e condições de pagamento

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|----------|------|-------------|-----------|
| eaa0103 | Objeto | Sim | Item do documento SCV a ser calculado |
| jsonEaa0103 | TableMap | Sim | Campos livres do item utilizados nos cálculos |
| abm01 | Objeto | Sim | Item do cadastro |
| abm0101 | Objeto | Sim | Configuração do item por empresa |
| abm10 | Objeto | Não | Valores do item |
| abm1001 | Objeto | Não | Valores do item por UF |
| abm1002 | Objeto | Não | Valores do item por município |
| abm1003 | Objeto | Não | Valores do item por entidade |
| abm12 | Objeto | Sim | Configuração fiscal do item |
| abm13 | Objeto | Não | Configuração comercial do item |
| abe01 | Objeto | Sim | Entidade do documento |
| aac10 | Objeto | Sim | Empresa ativa |

## 🔄 Fluxo do Processo

### 1. Preparação e Validações Iniciais
- Recupera o item do documento (`Eaa0103`) e o documento principal (`Eaa01`).
- Valida se o documento é de saída.
- Valida dados obrigatórios da entidade e do endereço principal.

### 2. Carregamento de Configurações
- Busca dados da empresa, entidade, UF, município e país.
- Carrega configurações fiscais e comerciais do item.
- Carrega valores do item por UF, município e entidade.

### 3. Formação de Preço e Comissão
- Obtém o preço unitário a partir da tabela de preços.
- Valida vencimento da tabela de preços.
- Aplica taxas de comissão por nível (0 a 4).

### 4. Ajustes Fiscais e Comerciais
- Ajusta CFOP conforme operação, UF, tipo de item e regime da entidade.
- Define CSTs de ICMS, IPI, PIS, COFINS, CSOSN e CBS/IBS.

### 5. Cálculo de Valores
- Calcula total do item.
- Converte quantidades e unidades.
- Calcula peso bruto, peso líquido e volume.
- Aplica descontos condicionais e incondicionais.

### 6. Apuração de Tributos
- ICMS (normal, redução, isenção, ST, desoneração).
- IPI conforme CST e NCM.
- PIS e COFINS conforme CST e regime tributário.
- Aplicação de regras de Zona Franca, ALC e Amazônia Ocidental.

### 7. Totalização
- Calcula total do documento por item.
- Define valor financeiro do item.

## ⚠️ Regras de Negócio
- Apenas documentos de **saída** são permitidos.
- A tabela de preços não pode estar vencida.
- O item deve possuir configuração fiscal válida.
- CSTs devem ser compatíveis com alíquotas e regras fiscais.
- Regras específicas são aplicadas para:
  - Zona Franca de Manaus
  - Área de Livre Comércio
  - Amazônia Ocidental
  - Regimes especiais
  - Entidades não contribuintes
- Reforma Tributária (CBS/IBS) é considerada quando configurada.

## 🔧 Métodos Principais
- **obterTipoFormula()**  
  Retorna o tipo de fórmula `SCV_SRF_ITEM_DO_DOCUMENTO`.

- **executar()**  
  Método principal que realiza todo o processamento do item, incluindo carregamento de dados, cálculos e gravação dos resultados.

- **setarObterPrecoUnitarioTaxasComissaoItem()**  
  Obtém preço unitário e taxas de comissão a partir da tabela de preços e configuração do item.

- **calcularItem()**  
  Executa todos os cálculos fiscais, comerciais e tributários do item.

## 🔧 Dependências

### Bibliotecas
- br.com.multiorm.criteria.criterion.Criterions
- br.com.multitec.utils.ValidacaoException
- br.com.multitec.utils.collections.TableMap
- sam.server.samdev.utils.Parametro
- sam.core.variaveis.MDate

### Entidades
- FormulaBase
- Entidades fiscais, comerciais e tributárias do SAM

## 📝 Observações Técnicas
- A fórmula utiliza extensivamente campos livres (`TableMap`) para flexibilizar regras fiscais.
- O cálculo é altamente dependente da correta configuração do item, entidade e UF.
- Possui grande complexidade fiscal, refletindo legislações estaduais e federais brasileiras.
- O processamento é sensível a CSTs e CFOPs, lançando exceções quando inconsistências são encontradas.
- O código foi projetado para garantir consistência fiscal e evitar cálculos incorretos no momento da emissão do documento.
