# EFD_Leiaute15_2021 - Geração de EFD Contribuições (PIS/COFINS)

## 📖 Descrição
Fórmula para geração do arquivo digital da EFD Contribuições (PIS/COFINS) conforme Leiaute 15 versão 2021, contemplando múltiplos blocos e registros exigidos pela legislação fiscal.

## 🎯 Finalidade
Gerar automaticamente o arquivo da Escrituração Fiscal Digital de Contribuições (EFD-Contribuições), processando documentos fiscais, operações diversas, apurações e registros complementares para entrega à Receita Federal.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Departamento Pessoal (para folha de pagamento)

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Aac10` - Empresas
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens dos documentos fiscais
- `Edb10` - Apuração da receita bruta
- `Edb11` - Operações geradoras de crédito
- `Edb12` - Retenções na fonte
- `Edb13` - Deduções diversas
- `Edb14` - Créditos por incorporação/fusão/cisão
- `Abe01` - Entidades (clientes/fornecedores)
- `Abm01` - Itens (produtos/serviços)
- `Abb40` - Processos referencia dos

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtInicial | LocalDate | Sim | Data inicial do período de apuração |
| dtFinal | LocalDate | Sim | Data final do período de apuração |
| situacao | Integer | Sim | Situação da EFD (0=Original, 1=Substituta, 9=Retificadora) |
| numRecibo | String | Não | Número do recibo (para retificações) |
| aliqPisF150 | BigDecimal | Não | Alíquota de PIS para estoque de abertura |
| aliqCofinsF150 | BigDecimal | Não | Alíquota de COFINS para estoque de abertura |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação dos parâmetros obrigatórios
- Busca da empresa matriz e filiais
- Verificação da apuração da receita bruta do período
- Inicialização de contadores e estruturas de dados

### 2. **Geração do Bloco 0 - Abertura e Identificação**
- Registro 0000: Abertura do arquivo e identificação da entidade
- Registro 0001: Abertura do Bloco 0
- Registro 0100: Dados do contabilista
- Registro 0110: Regimes de apuração
- Registro 0111: Receita bruta para rateio de créditos

### 3. **Geração do Bloco A - Documentos Fiscais de Serviços**
- Registro A001: Abertura do Bloco A
- Registro A010: Identificação do estabelecimento
- Registro A100: Nota Fiscal de Serviço
- Registros filhos: A110, A111, A120, A170

### 4. **Geração do Bloco C - Documentos Fiscais de Mercadorias (ICMS/IPI)**
- Registro C001: Abertura do Bloco C
- Registro C010: Identificação do estabelecimento
- Registro C100: Nota Fiscal/NF-e/NFC-e
- Registros filhos: C110, C111, C120, C170, C175
- Registro C395: Nota Fiscal de Venda a Consumidor
- Registro C396: Itens do documento
- Registro C500: Nota Fiscal de Energia/Água/Gás
- Registros filhos: C501, C505, C509
- Registro C860: Cupom Fiscal Eletrônico SAT
- Registro C870: Resumo diário do CF-e

### 5. **Geração do Bloco D - Documentos Fiscais de Serviços (ICMS)**
- Registro D001: Abertura do Bloco D
- Registro D010: Identificação do estabelecimento
- Registro D100: Nota Fiscal/Conhecimento de Transporte
- Registros filhos: D101, D105, D111
- Registro D500: Nota Fiscal de Serviço de Comunicação
- Registros filhos: D501, D505, D509

### 6. **Geração do Bloco F - Demais Documentos e Operações**
- Registro F001: Abertura do Bloco F
- Registro F010: Identificação do estabelecimento
- Registro F100: Operações geradoras de contribuição e créditos
- Registro F120: Bens que geram créditos por depreciação
- Registro F130: Bens que geram créditos por valor de aquisição
- Registro F150: Crédito presumido sobre estoque de abertura
- Registro F550: Consolidação das operações (lucro presumido)
- Registro F600: Contribuição retida na fonte
- Registro F700: Deduções diversas
- Registro F800: Créditos por incorporação/fusão/cisão

### 7. **Geração do Bloco P - Contribuição Previdenciária sobre Receita Bruta**
- Registro P001: Abertura do Bloco P
- Registro P010: Identificação do estabelecimento
- Registro P100: Contribuição previdenciária sobre receita bruta
- Registro P200: Consolidação
- Registro P210: Ajustes

### 8. **Geração do Bloco 1 - Operações Extemporâneas**
- Registro 1001: Abertura do Bloco 1
- Registro 1010: Processo referenciado - Ação judicial
- Registro 1020: Processo referenciado - Processo administrativo
- Registro 1900: Consolidação de documentos PJ lucro presumido

### 9. **Finalização**
- Registros 0140-0600: Tabelas auxiliares (participantes, itens, etc.)
- Registro 0990: Encerramento do Bloco 0
- Registro 9990/9999: Encerramento do arquivo

## ⚠️ Regras de Negócio

### Validações
- Verificação de apuração da receita bruta obrigatória
- Validação de situação dos documentos fiscais
- Controle de empresas já processadas por grupo centralizador
- Verificação de dados obrigatórios para cada registro

### Tratamento de Documentos
- Documentos com situação especial (02, 03, 04, 05) geram registros parciais
- Suporte a múltiplos modelos de documentos fiscais
- Tratamento diferenciado para entrada/saída
- Agrupamento por estabelecimento/empresa

### Cálculos Financeiros
- Cálculo de bases de PIS/COFINS
- Aplicação de alíquotas específicas
- Rateio de créditos comuns
- Consolidação de valores por agrupamento

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra toda a geração da EFD Contribuições.

### `gerarAberturaBloco0()`
Gera os registros iniciais do Bloco 0.

### `gerarBlocoA()`, `gerarBlocoC()`, `gerarBlocoD()`, `gerarBlocoF()`, `gerarBlocoP()`, `gerarBloco1()`
Métodos específicos para geração de cada bloco da EFD.

### `gerarC100()`, `gerarC300()`, `gerarC500()`, `gerarC800()`
Métodos auxiliares para tipos específicos de documentos do Bloco C.

### `gerarFechamentoBloco0()`
Gera as tabelas auxiliares e finaliza o Bloco 0.

### `gerarBloco9()`
Gera o Bloco 9 de controle e encerramento.

### `formatarValor()`, `formatarSerie()`, `retirarMascara()`
Métodos utilitários para formatação de dados.

### `selecionarCSTPis()`, `selecionarCSTCofins()`
Métodos para definição correta do CST conforme movimento e base de cálculo.

## 📊 Estrutura de Saída

**Arquivo Texto:**
- Dois arquivos concatenados (txt1 e txt2)
- Formato delimitado por pipe ("|")
- Codificação específica para EFD
- Registros organizados por blocos

**Conteúdo:**
- Bloco 0: Identificação e tabelas auxiliares
- Bloco A: Documentos de serviços
- Bloco C: Documentos de mercadorias
- Bloco D: Documentos de serviços (ICMS)
- Bloco F: Operações diversas
- Bloco M: Apuração (reservado)
- Bloco P: Contribuição previdenciária
- Bloco 1: Operações extemporâneas
- Bloco 9: Controle e encerramento

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários diversos
- `java.time` - Manipulação de datas
- `java.math` - Cálculos com BigDecimal

**Módulos:**
- Módulo Fiscal
- Módulo Contábil
- Módulo de Estoques
- Módulo de Serviços

## 📝 Observações Técnicas

### Performance
- Processamento paginado para grandes volumes
- Uso de caching para dados frequentes
- Agrupamento em memória para consolidação

### Tratamento de Erros
- Validações preventivas
- Mensagens de erro específicas
- Continuidade após erros não-críticos

### Configurabilidade
- Suporte a múltiplas empresas/filiais
- Configuração por grupo centralizador
- Parâmetros flexíveis por período

### Campos JSON
- Extensão de campos via JSON em várias tabelas
- Mapeamento dinâmico de campos da EFD
- Suporte a versões diferentes do leiaute

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de EFD Contribuições  
**Versão:** 1.0  
**Leiaute:** 015 (versão 2021)  
**Módulo:** Fiscal/Contábil