# SCV_CRM

## 📖 Descrição
Classe `SCV_CRM` do sistema Linhasita responsável por gerar relatórios de **CRM (Customer Relationship Management)**, com foco no ciclo de compras dos clientes. Calcula status de compra, ciclo médio, próxima compra prevista e ações recomendadas para cada cliente.

## 🎯 Finalidade
Fornecer informações estratégicas de relacionamento com clientes, permitindo:
- Identificar clientes que estão atrasados ou próximos do ciclo de compra;
- Estimar datas de próxima compra;
- Gerar status e ações recomendadas com base no histórico de vendas;
- Permitir filtragem por **entidade**, **representante** e **status**.

## 👥 Público-Alvo
- Equipe de Vendas
- Gerência Comercial
- CRM e Marketing

## ⚙️ Configuração
**Classe Principal:** `SCV_CRM`  
**Pacote:** `linhasita.relatorios.scv`

**Dependências:**
- `multiorm` - Persistência e consultas SQL
- `multitec.utils` - Utilitários de datas e coleções
- `sam.server.samdev.relatorio` - Framework de relatórios

## 📊 Dados e Fontes
**Tabelas Principais:**
- `ABE01` - Entidades/Clientes
- `ABE02` - Informações adicionais de clientes (ciclo de compra, hábito)
- `AAC10` - Empresas

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| entidade | List<Long> | Não | Lista de clientes a filtrar |
| representante | List<Long> | Não | Lista de representantes a filtrar |
| sRep | Boolean | Não | Indicador de considerar representantes sem vínculo (`true`) ou não (`false`) |
| status | List<String> | Não | Filtra os clientes por status (ATRASADO, QUASE ATRASADO, NO PRAZO) |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF | Relatório formatado com todas as informações do CRM | Arquivo |
| dados | Lista de clientes processados com ciclo, status e ação | List<TableMap> |

**Campos de cada registro do relatório:**
- `ent_na` → Nome da entidade
- `ent_cod` → Código da entidade
- `ultima_compra` → Data da última compra
- `dias_ciclo` → Dias do ciclo de compra médio
- `ciclo_compra` → Descrição do ciclo (SEMANAL, MENSAL, etc.)
- `proxima_compra` → Data estimada da próxima compra
- `dias_sem_comprar` → Dias desde a última compra
- `status` → Situação do cliente (ATRASADO, QUASE ATRASADO, NO PRAZO)
- `acao` → Ação recomendada (ENTRAR EM CONTATO, PREPARAR O CONTATO, ACOMPANHAR)
- `rep_na` → Nome do representante
- `rep_cod` → Código do representante

## 🔄 Fluxo do Processo

1. **Criação de valores iniciais**
   - `criarValoresIniciais()` retorna um mapa vazio, apenas placeholders para filtros.

2. **Execução do relatório**
   - Obtém filtros de **entidade**, **representante**, **status** e indicador `sRep`.
   - Chama `obterDadosRelatorio()` para buscar os dados do CRM no banco.
   - Itera sobre os registros, calculando:
      - Ciclo médio de compra (`dias_ciclo` e `ciclo_compra`)
      - Próxima compra prevista
      - Dias desde a última compra
      - Status e ação recomendada
   - Aplica filtro de status se fornecido.
   - Gera PDF com `gerarPDF(dados)`.

3. **Consulta ao banco**
   - `obterDadosRelatorio()` monta SQL dinâmico:
      - JOIN entre `ABE01` (clientes) e `ABE02` (informações adicionais)
      - JOIN com representante (`ABE01 rep`) se houver
      - Filtra por entidade, representante e condições de CRM
      - Ordena por representante e nome do cliente
   - Retorna lista de `TableMap` com os dados processados.

4. **Cálculo de ciclo e status**
   - `buscarDias(dias, retQtd)` → retorna **quantidade de dias** ou **descrição do ciclo** baseado em mapeamento:
     ```
     0 → SEMANAL → 7 dias
     1 → QUINZENAL → 15 dias
     2 → MENSAL → 30 dias
     ...
     ```
   - `obterStatus(diasCiclo, ultimaVenda, retStatus)` → calcula:
      - Data da próxima compra (`ultimaVenda + diasCiclo`)
      - Um terço do ciclo para definição de status "QUASE ATRASADO"
      - Retorna **status** ou **ação recomendada** dependendo do parâmetro `retStatus`

## ⚠️ Regras de Negócio

- Clientes com última compra dentro do ciclo previsto estão **NO PRAZO**.
- Ciclo médio definido pelo campo `dias_ciclo` na tabela `ABE02`.
- Próxima compra prevista = última compra + ciclo médio.
- Status calculado em três níveis:
   - ATRASADO → passou do ciclo previsto
   - QUASE ATRASADO → dentro do último terço do ciclo
   - NO PRAZO → antes do último terço do ciclo
- Ação recomendada baseada no status:
   - ATRASADO → "ENTRAR EM CONTATO"
   - QUASE ATRASADO → "PREPARAR O CONTATO"
   - NO PRAZO → "ACOMPANHAR"
- Possibilidade de filtrar clientes sem representante vinculado usando `sRep`.

## 🔄 Métodos Principais

### `executar()`
- Orquestra a execução do relatório:
   - Obtém dados do banco via `obterDadosRelatorio()`
   - Processa ciclo, status e ações
   - Aplica filtro de status
   - Gera PDF final

### `obterDadosRelatorio(entidades, representantes, sRep)`
- Monta SQL dinâmico com LEFT JOIN entre clientes e representantes
- Filtra por entidades, representantes, ciclo e hábitos de compra
- Retorna lista de `TableMap` com dados de cada cliente

### `buscarDias(dias, retQtd)`
- Retorna **quantidade de dias** ou **descrição do ciclo** baseado em mapeamento interno

### `obterStatus(diasCiclo, ultimaVenda, retStatus)`
- Calcula status e ação recomendada
- Usa datas atuais e ciclo para determinar ATRASADO, QUASE ATRASADO ou NO PRAZO

### `criarValoresIniciais()`
- Inicializa mapa de filtros padrão (vazio neste caso)

## 💡 Observações Técnicas
- Suporte a múltiplos filtros dinâmicos: entidade, representante, status
- SQL dinâmico otimizado com LEFT JOIN e filtros condicionais
- Cálculo de datas usando `LocalDate` e `ChronoUnit.DAYS`
- Controle do status e ações baseado em regras de CRM
- Possibilidade de exportar diretamente para PDF
- Compatível com clientes e representantes vinculados ou não

