# SRF - Gerar XML NFe (Reforma Tributária)

## 📖 Descrição
Fórmula responsável pela geração do arquivo XML da Nota Fiscal Eletrônica (NFe) seguindo o layout oficial da SEFAZ, com suporte à Reforma Tributária (CBS/IBS). A fórmula monta toda a estrutura XML conforme as especificações técnicas da NFe versão 4.00, incluindo tratamento para operações especiais como Zona Franca, ISSQN, ICMS ST e os novos tributos da Reforma Tributária.

## 🎯 Finalidade
Gerar o arquivo XML da NFe completo e válido para transmissão à SEFAZ, contemplando todas as regras fiscais brasileiras, incluindo as novas disposições da Reforma Tributária.

## 👥 Público-Alvo
- Departamento Fiscal
- Faturamento
- Integração com sistemas de emissão de NF-e
- Desenvolvedores de integração fiscal

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Eaa01` - Documentos fiscais
- `Eaa0102` - Dados gerais do documento
- `Eaa0103` - Itens do documento fiscal
- `Eaa01034` - Declaração de importação
- `Eaa010341` - Adições da DI
- `Eaa0104` - Declarações de exportação
- `Eaa01038` - Rastreabilidade
- `Eaa0113` - Financeiro do documento
- `Eaa01131` - Formas de pagamento
- `Aac10` - Empresa (emitente)
- `Abb01` - Central de documentos
- `Abe01` - Entidades (clientes/fornecedores)
- `Abe0101` - Endereços das entidades
- `Abm01` - Itens (produtos/serviços)
- `Abm0101` - Configuração do item por empresa
- `Aaj07` - Classificação tributária CBS/IBS
- `Aaj09` - CST CBS/IBS
- `Aaj25` - Crédito presumido

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal a ser convertido em XML |
| formaEmis | Integer | Sim | Forma de emissão (1-Normal, 2-Contingência, etc.) |
| contDt | LocalDate | Não | Data da contingência (se aplicável) |
| contHr | LocalTime | Não | Hora da contingência (se aplicável) |
| contJust | String | Não | Justificativa da contingência (se aplicável) |
| empresa | Aac10 | Sim | Empresa emitente |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Carregamento da empresa emitente
- Seleção do alinhamento (layout) conforme classificação tributária
- Composição dos dados do documento e seus relacionamentos

### 2. **Validações de Dados**
- Verificação de dados obrigatórios do emitente
- Validação de dados do destinatário
- Verificação de documentos referenciados
- Validação de itens e impostos
- Checagem de informações de transporte

### 3. **Geração da Estrutura XML**
- **Cabeçalho (ide)**: Identificação da NFe
- **Emitente (emit)**: Dados da empresa emissora
- **Destinatário (dest)**: Dados do cliente
- **Itens (det)**: Produtos e serviços com impostos
- **Totais (total)**: Valores consolidados
- **Transporte (transp)**: Informações de frete
- **Pagamento (pag)**: Formas de pagamento
- **Informações adicionais (infAdic)**

### 4. **Tratamento de Impostos**
- **ICMS/ICMS-ST**: Todos os CSTs e CSOSN
- **IPI**: Tributado e não tributado
- **PIS/COFINS**: Normal e ST
- **ISSQN**: Para serviços
- **Reforma Tributária**: CBS/IBS com todas as variações

### 5. **Elementos Especiais**
- Declaração de Importação (DI)
- Exportação
- Rastreabilidade
- Devolução
- Contingência

## ⚠️ Regras de Negócio

### Validações Críticas
- Empresa deve ter município com código IBGE válido
- Documento deve ter tipo definido e modelo (55-NFe, 65-NFCe)
- Destinatário deve ter dados completos para operações internas
- Itens devem ter CFOP, NCM e impostos configurados

### Regras Fiscais
- **Classificação tributária**: Define o regime (Simples Nacional, Lucro Real, etc.)
- **CST/CSOSN**: Aplicados conforme origem e destino
- **CFOP**: Ajustado automaticamente conforme tipo de operação
- **ICMS ST**: Calculado quando aplicável
- **Diferencial de alíquota**: Para operações interestaduais

### Reforma Tributária
- **CBS (Contribuição sobre Bens e Serviços)**: Novo imposto federal
- **IBS (Imposto sobre Bens e Serviços)**: Novo imposto estadual/municipal
- **Créditos presumidos**: Tratamento específico
- **Regimes especiais**: Zona Franca, Amazônia Ocidental
- **Classificações tributárias**: CST específicos para CBS/IBS

### Campos Livres (JSON)
Utilizados extensivamente para armazenar valores calculados:
- Valores de impostos por item (ICMS, IPI, PIS, COFINS)
- Base de cálculos e alíquotas
- Informações da Reforma Tributária
- Totais consolidados

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração do XML.

### `emitente()`
Monta a tag `<emit>` com dados da empresa emissora.

### `destinatario()`
Monta a tag `<dest>` com dados do cliente.

### `item()**
Processa cada item do documento, gerando as tags `<det>` com todos os impostos.

### `validarDadosDaNFe()`
Realiza validações abrangentes antes da geração do XML.

### `gerarIBSUF()`, `gerarIBSMun()`, `gerarCBS()`
Métodos específicos para a Reforma Tributária.

### `comporFilhosDocumento()**
Carrega todos os relacionamentos do documento.

## 📊 Estrutura de Saída

**XML Gerado:**
- Estrutura completa da NFe conforme layout 4.00
- Tags para todos os impostos (ICMS, IPI, PIS, COFINS, ISSQN, CBS/IBS)
- Chave de acesso calculada
- Informações de transporte e pagamento
- Observações fiscais e complementares

**Parâmetros de Saída:**
- `chaveNfe` - Chave de acesso da NFe (44 posições)
- `dados` - XML completo da nota
- `tagsAssinar` - Tags que necessitam assinatura digital (opcional)

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data, validação e strings
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas
- `java.text` - Formatação

**Utilitários:**
- `NFeUtils` - Funções específicas para NFe
- `ElementXml` - Manipulação de XML

**Módulo:** SRF (Sistema de Regras Fiscais)

## 📝 Observações Técnicas

### Tratamento de XML
- Uso de `ElementXml` para construção estruturada
- Escape de caracteres especiais (&, <, >, ", ')
- Formatação de números com precisão adequada
- Validação de tamanhos de campos

### Performance
- Carregamento lazy de entidades relacionadas
- Consultas otimizadas com joins
- Cache de dados frequentes (empresa, configurações)

### Segurança
- Validações preventivas antes da geração
- Tratamento de dados sensíveis (CPF/CNPJ)
- Formatação correta de campos numéricos

### Manutenibilidade
- Código modularizado por funcionalidade
- Métodos específicos para cada grupo de tags
- Comentários explicativos para regras complexas

### Compatibilidade
- Layout NFe 4.00
- Suporte a múltiplos regimes tributários
- Compatível com Reforma Tributária

---

**Última Alteração:** 12/01/2025 às 16:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Geração de XML  
**Versão:** 1.0