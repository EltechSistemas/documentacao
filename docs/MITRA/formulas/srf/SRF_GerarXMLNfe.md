# Documentação de Código Fonte - SRF - Geração de XML de Nota Fiscal Eletrônica (NFe)

---

## 📖 Descrição

Fórmula responsável pela geração do XML da Nota Fiscal Eletrônica (NFe) conforme layout 4.00, aplicando regras de validação, composição de tags, tratamento de impostos (ICMS, IPI, PIS, COFINS, ISS), informações de transporte, pagamento, parcelamento, importação, exportação, rastreabilidade e demais elementos obrigatórios conforme legislação vigente.

---

## 🎯 Finalidade

Gerar o arquivo XML da NFe de forma estruturada e válida para transmissão à SEFAZ, incluindo todos os dados do emitente, destinatário, itens, impostos, totais e informações complementares, com validações prévias para garantir conformidade fiscal.

---

## 👥 Público-Alvo

- Departamento Fiscal
- Faturamento
- Desenvolvedores
- Suporte Técnico

---

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Aac10` – Empresa (Emitente)
- `Aac1002` – Inscrição Estadual da Empresa por UF
- `Aag02` – Estados (UF)
- `Aag0201` – Municípios
- `Aah01` – Tipo de Documento
- `Aah20` – Veículos
- `Aaj03` – Situação do Documento
- `Aaj04` – Código ANP (Combustíveis)
- `Aaj10` – CST ICMS
- `Aaj12` – CST PIS
- `Aaj13` – CST COFINS
- `Aaj14` – CSOSN (Simples Nacional)
- `Aaj15` – CFOP
- `Abb01` – Central de Documentos
- `Abd01` – Parâmetros de Cálculo de Documentos (PCD)
- `Abe01` – Entidades (Cliente/Fornecedor)
- `Abe0101` – Endereços da Entidade
- `Abg01` – NCM
- `Abg0101` – NVE (Nomenclatura de Valor Aduaneiro)
- `Abm01` – Produtos
- `Abm0101` – Configurações do Produto por Empresa
- `Eaa01` – Documento Fiscal
- `Eaa0101` – Endereços do Documento
- `Eaa0102` – Dados Gerais do Documento
- `Eaa0103` – Itens do Documento
- `Eaa01033` – Itens Devolvidos Referenciados
- `Eaa01034` – Declaração de Importação
- `Eaa010341` – Adições da DI
- `Eaa01038` – Rastreabilidade do Item
- `Eaa0104` – Declaração de Exportação
- `Eaa0113` – Financeiro do Documento (Parcelas)
- `Eaa01131` – Formas de Pagamento das Parcelas

---

## ⚙️ Parâmetros da Fórmula

| Parâmetro  | Tipo        | Obrigatório | Descrição |
|------------|-------------|-------------|-----------|
| eaa01      | Eaa01       | Sim         | Documento fiscal a ser processado |
| formaEmis  | Integer     | Sim         | Forma de emissão (1=Normal, 2=Contingência) |
| contDt     | LocalDate   | Não         | Data de contingência (se aplicável) |
| contHr     | LocalTime   | Não         | Hora de contingência (se aplicável) |
| contJust   | String      | Não         | Justificativa da contingência |

---

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros de entrada
- Carregamento da empresa emitente (Aac10)
- Carregamento do documento (Eaa01) e seus filhos (endereços, itens, financeiro)
- Obtenção do PCD (Abd01) para regras específicas

### 2. **Validações Pré-Geração**
- Validação de dados obrigatórios do emitente (CNPJ, endereço, IE, município)
- Validação do documento (tipo, modelo, situação, dados gerais)
- Validação de endereços (entrega, retirada)
- Validação de itens (CFOP, CST, NCM, unidades)
- Validação de veículos e transportador
- Validação de documentos referenciados

### 3. **Geração da Estrutura XML**
- Criação do elemento raiz `NFe` e `infNFe`
- Composição da tag `ide` (identificação)
- Composição da tag `emit` (emitente)
- Composição da tag `dest` (destinatário)
- Composição da tag `det` (itens) com:
  - `prod` (dados do produto)
  - `imposto` (tributos)
  - `DI` (declaração de importação)
  - `rastro` (rastreabilidade)
  - `comb` (combustíveis)
- Composição da tag `total` (valores totais)
- Composição da tag `transp` (transporte)
- Composição da tag `cobr` (cobrança)
- Composição da tag `pag` (pagamento)
- Composição da tag `infRespTec` (responsável técnico)

### 4. **Cálculos e Formatações**
- Geração da chave de acesso
- Cálculo do código numérico (cNF)
- Formatação de valores monetários e quantidades
- Conversão de datas para padrão UTC
- Formatação de IE, CNPJ, CPF
- Aplicação de máscaras e limites de campos

### 5. **Validações Finais e Saída**
- Validação de consistência dos dados gerados
- Geração do XML como string
- Retorno da chave da NFe e do XML gerado

---

## ⚠️ Regras de Negócio

### **Validação de Dados do Emitente**
- Município e UF obrigatórios
- IE obrigatória para operações internas
- Endereço completo (logradouro, número, bairro, CEP)
- Razão social e nome fantasia preenchidos

### **Validação de Itens**
- CFOP obrigatório por item
- CST ICMS obrigatório (exceto para Simples Nacional com CSOSN)
- CST PIS/COFINS obrigatórios
- Unidades comercial e tributária preenchidas
- NCM válido para produtos
- Código GTIN ou “SEM GTIN”

### **Regras de Impostos**
- **ICMS**: conforme CST (00, 10, 20, 30, 40, 41, 50, 51, 60, 70, 90)
- **ICMS ST**: para operações com substituição tributária
- **ICMS Simples Nacional**: uso de CSOSN (101, 102, 103, 201, 202, 203, 300, 400, 500, 900)
- **IPI**: apenas se houver base de cálculo
- **PIS/COFINS**: conforme CST específico
- **ISS**: para serviços (tipo de produto 2 ou 3)

### **Regras de Transporte**
- Veículo obrigatório para operações internas
- Dados do transportador (CNPJ/CPF, nome, IE)
- Volumes (quantidade, espécie, marca, peso)

### **Regras de Pagamento**
- Formas de pagamento conforme cadastro (Abf40)
- Indicação de pagamento à vista ou a prazo
- Parcelamento registrado no financeiro (Eaa0113)

### **Regras de Contingência**
- Emissão normal (formaEmis = 1)
- Emissão em contingência (formaEmis = 2) com data, hora e justificativa

---

## 🔧 Métodos Principais

| Método | Descrição |
|--------|-----------|
| `executar()` | Método principal que orquestra toda a geração do XML |
| `validarDadosDaNFe()` | Realiza validações abrangentes antes da geração |
| `emitente()` | Compõe a tag do emitente |
| `destinatario()` | Compõe a tag do destinatário |
| `item()` | Processa todos os itens do documento |
| `comporFilhosDocumento()` | Carrega dados relacionados do documento |
| `buscarItensDoDocumento()` | Busca itens do documento ordenados |
| `buscarFinanceiroPorDocumento()` | Busca parcelas do documento |
| `buscarFormasDePagamentoPorDocumento()` | Busca formas de pagamento |
| `verificarFormaPgto()` | Define se pagamento é à vista ou a prazo |

---

## 📊 Estrutura de Saída

### **XML Gerado**
- Estrutura completa da NFe no padrão 4.00
- Tags obrigatórias e condicionais preenchidas
- Codificação UTF-8
- Caracteres especiais convertidos (ex: &, <, >)

### **Parâmetros de Retorno**
| Campo     | Tipo   | Descrição |
|-----------|--------|-----------|
| chaveNfe  | String | Chave de acesso da NFe (44 dígitos) |
| dados     | String | Conteúdo XML da NFe |

---

## 🧩 Observações Técnicas

- **Layout**: 4.00
- **Namespace**: `http://www.portalfiscal.inf.br/nfe`
- **Ambiente**: Produção ou Homologação conforme parâmetro `NFeDataProducao`
- **Forma de Emissão**: Normal (1) ou Contingência (2)
- **Versão do Processo**: `SAM4_[versão]`

---

## ✅ Validações Críticas

1. **Emitente sem município ou UF**
2. **Item sem CFOP ou CST**
3. **Documento sem tipo ou modelo**
4. **Endereço de entrega incompleto (para modelo 55)**
5. **Chave de acesso inválida**
6. **PCD não encontrado**
7. **Data de emissão futura**
8. **Valores totais inconsistentes**

---

## 📁 Arquivo Relacionado

**SRF_GerarXMLNfe.groovy**  
*Última alteração: 09/12/2025 16:26*  
*Autor: NAGYLA*

---

**Nota**: Esta documentação refere-se à versão atual do código e está alinhada com as regras fiscais vigentes em 2025. Em caso de alterações na legislação, a fórmula deve ser revisada e atualizada conforme necessário.