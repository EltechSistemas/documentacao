# SCF_Documentos.md

## 📖 Descrição
Sistema de relatórios para documentos financeiros do SCF (Sistema de Controle Financeiro) da Linhasita, permitindo a geração de relatórios analíticos e sintéticos de documentos a receber, recebidos, a pagar e pagos.

## 🎯 Finalidade
Fornecer relatórios financeiros completos e flexíveis para análise de documentos, com diferentes opções de ordenação, filtros e formatos de saída.

## 👥 Público-Alvo
- Departamento Financeiro
- Controladoria
- Crédito e Cobrança
- Diretoria

## ⚙️ Configuração
**Recursos Necessários:**
- Classe `SCF_Documentos` - Relatório de documentos financeiros

**Localização:** `linhasita/relatorios/scf/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA01` - Documentos financeiros
- `ABB01` - Documentos fiscais
- `ABE01` - Entidades/Clientes
- `ABF15` - Portadores
- `ABF16` - Operações
- `ABF20` - PLF (Plano de Livro Fiscal)
- `AAC10` - Empresas

**Entidades Envolvidas:**
- `Daa01` - Documentos financeiros
- `Abb01` - Documentos fiscais
- `Abe01` - Entidades
- `Abf15` - Portadores
- `Abf16` - Operações
- `Abf20` - PLF
- `Aac10` - Empresas

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| ordem | Integer | Sim | Tipo de ordenação (0-Número, 1-Vencimento, 2-Entidade, 3-Pagamento, 4-Portador, 5-Representante) |
| classe | Integer | Sim | Tipo de documento (0-À Receber, 1-Recebidos, 2-À Pagar, 3-Pagos, 4-À Receber/Recebidos, 5-À Pagar/Pagos) |
| numeroInicial | Integer | Sim | Número inicial do documento |
| numeroFinal | Integer | Sim | Número final do documento |
| empresas | List<Long> | Não | Lista de empresas para filtro |
| entidade | List<Long> | Não | Lista de entidades para filtro |
| dataVenc | LocalDate[] | Não | Período de vencimento |
| dataEmissao | LocalDate[] | Não | Período de emissão |
| tipoData | Integer | Não | Tipo de data (0-Pagamento, 1-Recebimento) |
| sintetico | Boolean | Não | Relatório sintético |
| vencimento | Integer | Não | Tipo de vencimento (0-Real, 1-Nominal) |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF/XLSX | Relatório formatado | Arquivo |
| documentos | Lista de documentos processados | List<TableMap> |

## 🔄 Fluxo do Processo

1. **Configuração Inicial**
   - Define valores padrão para filtros
   - Obtém empresas acessíveis ao usuário
   - Configura título do relatório baseado na classe selecionada

2. **Processamento de Filtros**
   - Aplica filtros de data, entidade, documento, etc.
   - Define ordenação conforme parâmetro
   - Configura condições WHERE dinâmicas

3. **Busca de Documentos**
   - Executa consulta SQL com filtros aplicados
   - Processa dados de juros, multa e desconto
   - Calcula dias em atraso

4. **Geração do Relatório**
   - Seleciona template baseado na ordenação
   - Gera PDF ou XLSX conforme seleção
   - Retorna arquivo para download

## ⚠️ Regras de Negócio

### Filtros e Ordenação
- Ordenação por número, vencimento, entidade, pagamento, portador ou representante
- Filtro por período de emissão, vencimento ou pagamento/recebimento
- Suporte a documentos à receber e a pagar

### Cálculos Financeiros
- Cálculo automático de juros e multa baseado em JSON do documento
- Cálculo de dias em atraso
- Tratamento de descontos

### Acesso a Dados
- Restrição por empresas acessíveis ao usuário
- Filtro por PLF (Plano de Livro Fiscal)
- Suporte a documentos transmutados

## 🎨 Tipos de Relatório

| Tipo | Descrição | Template |
|------|-----------|----------|
| Analítico | Detalhado por documento | SCF_Documentos |
| Por Entidade | Agrupado por entidade | SCF_DocumentosEntidade |
| Por Portador | Agrupado por portador | SCF_DocumentosPortador |
| Por Representante | Agrupado por representante | SCF_DocumentosRepresentante |
| Sintético | Resumido | SCF_DocumentosSintetico2 |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Persistência e consultas
- `multitec.utils` - Utilitários e coleções
- `sam.server.samdev.relatorio` - Framework de relatórios

**Consultas:**
- Busca de documentos com múltiplos filtros
- Cálculo de empresas acessíveis
- Agregação de dados por diferentes critérios

## 📝 Observações Técnicas

- Suporte a múltiplos formatos de saída (PDF/XLSX)
- Consulta otimizada com LEFT JOIN e ROW_NUMBER para dados mais recentes
- Tratamento de dados JSON para cálculos financeiros
- Flexibilidade total na ordenação e agrupamento
- Filtros dinâmicos baseados em parâmetros
- Suporte a documentos prévios (previsão) e reais
- Controle de acesso por empresa

## 🔄 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração do relatório.

### `buscaDocumentos()`
Executa a consulta SQL principal com todos os filtros aplicados.

### `criarValoresIniciais()`
Configura valores padrão e acessos iniciais.

### `empresasAcessiveis()`
Retorna lista de empresas que o usuário logado tem acesso.

## 💡 Consulta SQL
A consulta principal utiliza:
- `ROW_NUMBER()` para obter o registro mais recente da `daa0103`
- Múltiplos `LEFT JOIN` para relacionar todas as entidades
- Cláusulas `WHERE` dinâmicas baseadas nos filtros
- `ORDER BY` flexível baseado no parâmetro de ordenação