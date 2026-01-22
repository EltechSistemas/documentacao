# SRF_GerarXMLNfe - Documentação Técnica

## 📖 Descrição  
Fórmula responsável pela geração do arquivo XML da Nota Fiscal Eletrônica (NFe) conforme layout oficial da SEFAZ versão 4.00, incluindo suporte à Reforma Tributária, declarações de importação/exportação, combustíveis, rastreabilidade e formas de pagamento.

## 🎯 Finalidade  
- Converter documentos fiscais do sistema SAM4 em XML padrão SEFAZ
- Aplicar validações de negócio e fiscais
- Gerar chave de acesso da NFe
- Montar estrutura XML completa com todos os grupos
- Tratar tributações (ICMS, IPI, PIS, COFINS, ISSQN, Reforma Tributária)

## 👥 Público-Alvo  
- Departamento Fiscal
- Faturamento
- Desenvolvimento (integração com sistemas SEFAZ)
- Suporte Técnico

## 📊 Dependências Técnicas  

**Bibliotecas:**  
- `multiorm` - Criteria e consultas ao banco  
- `multitec.utils` - Utilitários de data, string e validação  
- `sam.dicdados` - Tipos de fórmula  
- `sam.server.samdev.utils.NFeUtils` - Utilitários específicos para NFe  
- `java.time` - Manipulação de datas  

**Entidades Principais:**  
- `Eaa01` - Documento fiscal  
- `Eaa0102` - Dados gerais do documento  
- `Eaa0103` - Itens do documento  
- `Aac10` - Empresa emitente  
- `Abe01` - Entidades (clientes/fornecedores)  
- `Abb01` - Central de documentos  

## ⚙️ Parâmetros de Entrada  

| Parâmetro | Tipo | Obrigatório | Descrição |  
|-----------|------|-------------|-----------|  
| eaa01 | Eaa01 | Sim | Documento fiscal a ser processado |  
| formaEmis | Integer | Sim | Forma de emissão (1=Normal, 2=Contingência) |  
| contDt | LocalDate | Não | Data da contingência |  
| contHr | LocalTime | Não | Hora da contingência |  
| contJust | String | Não | Justificativa da contingência |  

## 🔄 Fluxo do Processo  

### 1. **Configuração Inicial**  
- Validação dos parâmetros obrigatórios  
- Carregamento da empresa ativa  
- Seleção do alinhamento fiscal (0009 para Simples Nacional, 0010 para outros)  
- Composição dos objetos relacionados (filhos do documento)  

### 2. **Validação de Dados**  
- Validação dos dados do emitente (empresa)  
- Validação dos dados do documento fiscal  
- Validação de itens e tributação  
- Validação de documentos referenciados  
- Verificação de veículos e transportes  

### 3. **Geração da Chave de Acesso**  
- Cálculo do código numérico (cNF)  
- Geração da chave de acesso conforme regras da SEFAZ  
- Definição do ambiente (produção/homologação)  

### 4. **Construção do XML**  
- **Identificação (ide)**: Dados básicos da NFe  
- **Emitente (emit)**: Dados da empresa emitente  
- **Destinatário (dest)**: Dados do cliente  
- **Itens (det)**: Produtos/serviços com tributos  
- **Totais (total)**: Valores totais da nota  
- **Transporte (transp)**: Dados do frete  
- **Cobrança (cobr)**: Faturas e duplicatas  
- **Pagamento (pag)**: Formas de pagamento  
- **Informações Adicionais (infAdic)**: Observações  

### 5. **Tratamento Específico**  
- **Reforma Tributária**: Tratamento de IBS/CBS  
- **Combustíveis**: Tags específicas (ANP, CIDE)  
- **Importação/Exportação**: DI, adições, drawback  
- **Rastreabilidade**: Lotes, datas de fabricação/validade  
- **ISSQN**: Tratamento para serviços  

## ⚠️ Regras de Negócio  

### Validações Obrigatórias  
- Empresa deve ter município configurado com código IBGE  
- Documento deve ter tipo definido (modelo 55 ou 65)  
- Itens devem ter CFOP, unidade de medida e descrição  
- Para Simples Nacional: CSOSN é obrigatório  
- Para outras classificações: CST ICMS é obrigatório  

### Tratamento Tributário  
- **Simples Nacional**: CSOSN 101, 102, 201, 202, 500, 900  
- **Regime Normal**: CST ICMS 00, 10, 20, 30, 40, 41, 50, 51, 60, 70, 90  
- **PIS/COFINS**: CST 01-09, 49-75, 98-99  
- **IPI**: CST 00, 01-05, 49-55  
- **ISSQN**: Aplicado para itens do tipo serviço  

### Formas de Pagamento  
- À vista: parcela única com vencimento igual à emissão  
- À prazo: múltiplas parcelas com datas futuras  
- Meios de pagamento: dinheiro, cheque, cartão, etc.  

## 🔧 Métodos Principais  

### `executar()`  
Método principal que orquestra todo o processo de geração do XML.  

### `validarDadosDaNFe()`  
Realiza validações completas dos dados antes da geração do XML.  

### `emitente()`  
Monta o bloco do emitente com dados da empresa.  

### `destinatario()`  
Monta o bloco do destinatário com dados do cliente.  

### `item()`  
Processa todos os itens do documento, incluindo tributos.  

### `comporFilhosDocumento()`  
Carrega objetos relacionados (central, dados gerais, endereços).  

## 📊 Estrutura de Saída  

**Retornos:**  
- `chaveNfe` - Chave de acesso gerada (44 caracteres)  
- `dados` - XML completo da NFe  
- `tagsAssinar` - Tags que necessitam assinatura digital  

## 🔧 Métodos Auxiliares  

### Consultas ao Banco  
- `buscarInscricaoEstadualPorEstado()` - IE da empresa por UF  
- `buscarDocumentosReferenciados()` - Notas referenciadas  
- `buscarItensDoDocumento()` - Itens da nota  
- `buscarFinanceiroPorDocumento()` - Parcelas financeiras  
- `buscarFormasDePagamentoPorDocumento()` - Formas de pagamento  

### Utilitários NFe  
- `verificarFormaPgto()` - Determina se é à vista ou à prazo  
- `getDecimal()` - Formatação decimal para XML  
- `gerarIBSUF()`, `gerarIBSMun()`, `gerarCBS()` - Reforma Tributária  

## 📝 Observações Técnicas  

### Versão do Layout  
- Layout 4.00 da NFe  
- Suporte a NFCe (modelo 65)  
- Formato UTC para datas/horas  

### Tratamento de Caracteres  
- Substituição de caracteres especiais (&, <, >, ", ')  
- Limitação de tamanho de campos conforme manual técnico  
- Formatação de números (2 ou 4 decimais)  

### Ambiente  
- Produção: tpAmb = 1  
- Homologação: tpAmb = 2  
- Destinatário genérico em homologação  

### Reforma Tributária  
- Suporte a IBS (Imposto sobre Bens e Serviços)  
- Suporte a CBS (Contribuição sobre Bens e Serviços)  
- Créditos presumidos  
- Estorno de créditos  

## ⚠️ Validações Críticas  

### Emitente  
- CNPJ/CPF deve estar preenchido  
- Endereço completo (logradouro, número, bairro)  
- Município com código IBGE  
- Inscrição estadual válida para a UF  

### Destinatário  
- Nome/razão social obrigatório  
- Endereço completo para entregas  
- IE obrigatória exceto para isentos e exterior  

### Itens  
- Código do produto (cProd)  
- Descrição (xProd)  
- NCM válido (exceto serviços)  
- CFOP compatível com operação  
- Unidades de medida comercial e tributária  

### Documentos Referenciados  
- Chave de acesso para NFe/CTe  
- Dados completos para NF modelo 01/02  
- Dados de produtor rural para NF modelo 04  

## 🔗 Integrações  

### Com SEFAZ  
- Geração de chave de acesso conforme padrões oficiais  
- Formatação de dados conforme manual técnico  
- Suporte a contingência (formaEmis = 2)  

### Com Sistema SAM4  
- Leitura de documentos fiscais (Eaa01)  
- Consulta a entidades (Abe01)  
- Obtenção de dados tributários (Abm0101)  
- Recuperação de parcelas financeiras (Eaa0113)  

## 🚨 Tratamento de Erros  

### ValidaçãoException  
- Lançada quando parâmetros obrigatórios estão ausentes  
- Interrompe o processamento imediatamente  

### MultiValidationException  
- Acumula múltiplas mensagens de validação  
- Permite correção em lote dos problemas identificados  

### Exemplos de Erros Comuns  
- "Necessário informar o município no cadastro da empresa ativa."  
- "Documento não encontrado na Central de Documentos."  
- "Não foi informado o tipo do documento."  
- "Necessário informar o CFOP do item [código] no documento."  

## 📁 Estrutura de Arquivos  

### Arquivos Relacionados  
- `SRF_GerarXMLNfe.groovy` - Classe principal  
- `NFeUtils.java` - Utilitários específicos para NFe  
- `FormulaBase.java` - Classe base para fórmulas SAM4  

### Configurações Necessárias  
- Alinhamento 0009 (Simples Nacional) configurado  
- Alinhamento 0010 (Outros regimes) configurado  
- Parâmetro "NFeDataProducao" definido no módulo EA  

## 🔄 Ciclo de Vida  

### 1. **Inicialização**  
- Criação da instância da fórmula  
- Carregamento dos parâmetros de entrada  
- Configuração do acesso ao banco de dados  

### 2. **Processamento**  
- Validações iniciais  
- Geração do XML  
- Aplicação de regras tributárias  
- Formatação final  

### 3. **Finalização**  
- Retorno dos resultados (chaveNfe, dados)  
- Limpeza de recursos  
- Log de execução  

## 📈 Métricas de Performance  

### Tempo de Execução  
- Dependente do número de itens  
- Influenciado pela complexidade tributária  
- Afetado por consultas ao banco de dados  

### Consumo de Memória  
- XML final mantido em memória  
- Objetos de negócio carregados sob demanda  
- Cache de consultas frequentes  

## 🔐 Segurança  

### Dados Sensíveis  
- CNPJ/CPF mascarados em logs  
- Chaves de acesso geradas de forma segura  
- XML sanitizado para injeção de caracteres  

### Validações de Integridade  
- Verificação de consistência dos dados  
- Validação de regras fiscais  
- Checagem de autorizações de acesso  

## 🆕 Atualizações  

### Última Alteração  
- **Data**: 18/12/2025 08:20  
- **Autor**: Bruno  
- **Alterações**: Suporte à Reforma Tributária  

### Histórico de Versões  
- **v1.0** (27/10/2025): Versão inicial  
- **v1.1** (18/12/2025): Adicionado suporte à Reforma Tributária  

---  

**Nota**: Esta documentação deve ser atualizada sempre que houver modificações significativas no código fonte.  