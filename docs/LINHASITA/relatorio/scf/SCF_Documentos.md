# SCF_Documentos

## 📖 Descrição
Classe `SCF_Documentos` do sistema Linhasita responsável pela geração de relatórios financeiros de documentos à receber, recebidos, à pagar e pagos. Suporta filtros detalhados, cálculos financeiros e geração em PDF/XLSX.

## 🎯 Finalidade
Fornecer relatórios financeiros completos, permitindo:
- Filtragem por período, entidade, operação, portador, PLF e tipo de documento;
- Ordenação por número, vencimento, entidade, pagamento, portador ou representante;
- Cálculo de juros, multa e desconto automático;
- Relatórios analíticos, sintéticos ou agrupados.

## 👥 Público-Alvo
- Departamento Financeiro
- Controladoria
- Crédito e Cobrança
- Diretoria

## ⚙️ Configuração
**Classe Principal:** `SCF_Documentos`  
**Pacote:** `linhasita.relatorios.scf`

**Dependências:**
- `multiorm` - Persistência e consultas
- `multitec.utils` - Utilitários e coleções
- `sam.server.samdev.relatorio` - Framework de relatórios

## 📊 Dados e Fontes
**Tabelas Principais:**
- `DAA01` - Documentos financeiros
- `DAA0103` - Histórico de documentos financeiros
- `ABB01` - Documentos fiscais
- `ABE01` - Entidades/Clientes
- `ABF15` - Portadores
- `ABF16` - Operações
- `ABF20` - PLF (Plano de Livro Fiscal)
- `AAC10` - Empresas

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
| dataLcto | LocalDate[] | Não | Período de lançamento |
| tipoData | Integer | Não | Tipo de data (0-Pagamento, 1-Recebimento) |
| portador | List<Long> | Não | Filtro por portador |
| operacao | List<Long> | Não | Filtro por operação |
| plf | List<Long> | Não | Filtro por PLF |
| rep | List<Long> | Não | Filtro por representante |
| documento | List<Long> | Não | Filtro por tipo de documento |
| sintetico | Boolean | Não | Relatório sintético |
| vencimento | Integer | Não | Tipo de vencimento (0-Real, 1-Nominal) |
| isTotalDia | Boolean | Não | Total diário no relatório |
| considerarTransmutado | Boolean | Não | Considerar ou ignorar documentos transmutados |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF/XLSX | Relatório formatado | Arquivo |
| documentos | Lista de documentos processados | List<TableMap> |

## 🔄 Fluxo do Processo

1. **Criação de Valores Iniciais**
   - Define valores padrão de filtros (ordem, classe, números, datas e PLF)
   - Obtém empresas acessíveis ao usuário
   - Configura parâmetros iniciais, como considerar documentos transmutados

2. **Configuração de Título e Parâmetros**
   - Define título do relatório conforme `classe` e `ordem`
   - Adiciona parâmetros de empresa, período, tipo de data e total diário

3. **Busca de Documentos**
   - Executa método `buscaDocumentos()` com filtros aplicados
   - Realiza LEFT JOIN com `daa0103` usando `ROW_NUMBER()` para pegar o registro mais recente
   - Filtra por PLF, entidade, portador, operação, tipo de documento e transmutado

4. **Processamento Financeiro**
   - Calcula dias de atraso
   - Calcula juros, multa e desconto a partir de JSON do documento
   - Marca documentos como "Previsão" ou "Real"
   - Remove duplicidades

5. **Geração do Relatório**
   - Seleciona template com base em ordem e tipo sintético
   - Gera PDF ou XLSX conforme parâmetro `impressao`

## ⚠️ Regras de Negócio

- Ordenação por número, vencimento, entidade, pagamento, portador ou representante
- Filtro por período de emissão, vencimento, pagamento/recebimento
- Cálculo automático de juros, multa e desconto
- Consideração de documentos transmutados
- Controle de acesso por empresa e usuário
- Suporte a documentos à receber e à pagar, prévios e reais

## 🎨 Tipos de Relatório

| Tipo | Descrição | Template |
|------|-----------|----------|
| Analítico | Detalhado por documento | SCF_Documentos |
| Por Entidade | Agrupado por entidade | SCF_DocumentosEntidade |
| Por Portador | Agrupado por portador | SCF_DocumentosPortador |
| Por Representante | Agrupado por representante | SCF_DocumentosRepresentante |
| Sintético | Resumido | SCF_DocumentosSintetico2 |

## 🔄 Métodos Principais

### `executar()`
- Orquestra a geração do relatório, aplica filtros, calcula juros/multa/desconto, remove duplicidades e gera PDF/XLSX.

### `buscaDocumentos(...)`
- Executa a consulta SQL principal, com múltiplos LEFT JOINs e filtros dinâmicos.
- Suporta `ROW_NUMBER()` para buscar a versão mais recente do documento.

### `criarValoresIniciais()`
- Retorna os filtros padrão, PLF e empresa do usuário logado.

### `empresasAcessiveis()`
- Retorna lista de empresas que o usuário tem acesso.

### `buscarEmpresa()`
- Retorna a empresa principal do relatório concatenada em string.

## 💡 Observações Técnicas
- Suporte a múltiplos formatos de saída (PDF/XLSX)
- Consulta SQL otimizada com LEFT JOIN e ROW_NUMBER
- Tratamento de dados JSON para cálculos financeiros
- Flexibilidade na ordenação e agrupamento
- Filtros dinâmicos baseados em parâmetros
- Suporte a documentos prévios (previsão) e reais
- Controle de acesso por empresa

