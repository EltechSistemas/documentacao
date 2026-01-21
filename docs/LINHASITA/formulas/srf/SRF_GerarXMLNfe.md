# SRF_GerarXMLNfe

## 📄 Descrição
O **SRF_GerarXMLNfe** é uma classe responsável pela **validação, composição e conferência de documentos fiscais eletrônicos** (como NF-e e NFC-e), garantindo que todas as informações necessárias para a emissão estejam corretas antes do processamento.

Ela realiza verificações de dados do emissor, destinatário, itens, impostos, importações, endereços e parcelas, além de buscar informações relacionadas no banco de dados e consolidar os registros para emissão do XML fiscal.

---

## 🎯 Finalidade
- Validar campos obrigatórios em documentos fiscais
- Garantir consistência de informações de endereço, município, UF e IE
- Validar dados tributários de ICMS, PIS, COFINS, IPI e ISS
- Conferir informações de importação e exportação de mercadorias
- Preparar os dados completos para geração do XML da NF-e/NFC-e
- Suportar diferentes tipos de documentos fiscais e regimes tributários

---

## 👥 Público-Alvo
- Equipes de emissão fiscal
- Desenvolvedores do módulo fiscal
- Controladoria e auditoria fiscal
- Sistemas integradores de ERP

---

## 📊 Dados e Fontes
- **Documentos fiscais:** tabelas `Eaa01`, `Eaa0102`, `Eaa0103`, `Eaa01034`, `Eaa010341`, `Eaa0104`, `Eaa01038`, `Eaa0113`
- **Endereços e entidades:** tabelas `Abe01`, `Abe0101`, `Aag02`, `Aac10`, `Aac1002`
- **Tributação e classificações:** tabelas `Aaj01`, `Aaj03`, `Aaj10`, `Aaj12`, `Aaj13`, `Abm0101`, `Abg0101`, `Aac13`
- **Financeiro e parcelamento:** tabelas `Eaa0113`, `Eaa01131`, `Abf40`
- Dados obtidos via consultas SQL customizadas ou critérios de ORM (`Criterions`, `Joins`, `TableMap`).

---

## 🔢 Parâmetros do Processo
O processo **não recebe parâmetros externos diretos**.  
As validações e cálculos são feitos com base nos registros do documento fiscal:

- Documento fiscal (`Eaa01`)
- Itens (`Eaa0103`) e seus campos livres
- Dados de importação (`Eaa01034`) e adições (`Eaa010341`)
- Endereços de despacho, retirada e entrega
- Parcelamentos e formas de pagamento (`Eaa0113`)
- Regime tributário da empresa e classificação tributária (`Aac13` / `Aaj01`)

---

## 📤 Saídas do Processo
- Lançamento de mensagens de validação (`ValidationMessage`) em caso de inconsistências
- Composição dos filhos do documento (`Eaa0102`, `Eaa0101`, `Abb01`, etc.)
- Consolidação de dados fiscais, tributários e de logística para emissão do XML
- Possível exceção (`validations`) caso existam erros críticos

---

## 🔄 Fluxo do Processo
1. Carrega o documento fiscal (`Eaa01`) e seus registros relacionados
2. Verifica informações do **despacho**
3. Valida **endereços de retirada e entrega**
4. Valida **itens do documento**:
    - Unidade de medida comercial e tributária
    - Descrição, CFOP, CSOSN/CST de ICMS, PIS, COFINS, IPI
    - Serviços e ISS, se aplicável
5. Valida **importação** e adições
6. Valida **parcelamento de cupom fiscal** (modelo 65)
7. Determina a **forma de pagamento** (à vista ou a prazo)
8. Composição dos **filhos do documento** (dados gerais, endereços, central)
9. Busca dados complementares:
    - Inscrição estadual por estado
    - Documentos referenciados
    - Rastreabilidade de itens
    - Formas de pagamento e totais parcelados
10. Lança exceções caso haja inconsistências detectadas

---

## 📜 Regras de Negócio
- IE obrigatória para empresas contribuintes de ICMS
- Município e UF obrigatórios nos endereços
- CFOP obrigatório nos itens
- Validação de CSTs para ICMS, PIS, COFINS e IPI conforme regime tributário
- CSOSN obrigatório para Simples Nacional
- Campos livres obrigatórios quando exigidos
- Validação de importação e adições para DI (data, UF, código do exportador, fabricante estrangeiro)
- Parcelamento de cupom fiscal deve indicar tipo do documento para gerar forma de pagamento

---

## ⚠️ Inconsistências
- Campos obrigatórios ausentes: endereço, município, UF, IE, CFOP, CST/CSOSN, dados de importação
- Valores tributários inválidos ou não preenchidos
- Parcelamento sem tipo de documento definido
- Dependência do banco de dados para configuração de itens e classificação tributária
- Dados ausentes em campos livres podem impedir emissão do XML

---

## 🔗 Dependências
- ORM/Session (`getSession()`)
- Utilitários de data (`DateUtils`) e NFe (`NFeUtils`)
- Estruturas de validação (`ValidationMessage`, `Validations`)
- Tabelas do ERP fiscal/comercial (`Eaa01`, `Abe01`, `Aac10`, etc.)
- Framework base (`FormulaBase`)
- Recursos de busca e consulta: `getAcessoAoBanco()`

---

## 🛠️ Observações Técnicas
- Classe estende `FormulaBase`
- Uso extensivo de `Criterions` e `Joins` para ORM
- Integra validação fiscal, tributária, logística e financeira
- Formatação de valores e datas para XML fiscal
- Preparada para lidar com diferentes modelos de documento (NF-e, NFC-e, cupom fiscal)
- Integração com regras de reforma tributária (CSOSN, IBS, CBS)
