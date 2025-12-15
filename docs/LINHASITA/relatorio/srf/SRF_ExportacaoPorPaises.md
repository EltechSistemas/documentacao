# SRF_ExportacaoPorPaises

## 📖 Descrição
Relatório de exportação por países, desenvolvido para o módulo Linhasita, que consolida informações de documentos fiscais de exportação, incluindo valores em dólar, pesos líquidos e dados de clientes por país de destino.

## 🎯 Finalidade
Gerar um relatório consolidado das exportações realizadas, agrupando por país de destino, com informações financeiras e operacionais para análise comercial e fiscal.

## 👥 Público-Alvo
- Departamento Comercial/Exportação
- Controladoria
- Departamento Fiscal
- Gestão Estratégica

## 📊 Dados e Fontes

**Tabelas Principais:**
- `eaa01` – Documentos fiscais
- `abb01` – Documentos de origem (central)
- `abe01` – Entidades (clientes)
- `eaa0101` – Itens do documento fiscal
- `aag01` – Países
- `aag10` – Moedas

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valor Padrão |
|-----------|------|-------------|-----------|--------------|
| emissao | LocalDate[] | Não | Período de emissão dos documentos | Mês atual |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Define nome da tarefa: "SRF - Exportação Por Países - Linhasita"
- Configura filtros padrão (mês atual)

### 2. **Coleta de Parâmetros**
- Obtém intervalo de datas do filtro "emissao"
- Busca empresa ativa do sistema (Aac10)

### 3. **Preparação de Parâmetros para Relatório**
- Razão social da empresa
- Período de emissão formatado

### 4. **Busca de Dados**
- Executa consulta SQL com filtros aplicados
- Retorna lista de TableMap com os dados consolidados

### 5. **Geração do Relatório**
- Gera PDF com os dados obtidos

## ⚠️ Regras de Negócio

### Filtros Aplicados
- Apenas documentos não cancelados (`eaa01cancData is null`)
- Apenas documentos de saída (`eaa01esMov = 1`)
- Apenas documentos fiscais (`eaa01clasDoc = 1`)
- Exclui destino "Brasil" (`aag01nome <> 'Brasil'`)
- Considera apenas itens principais (`eaa0101principal = 1`)

### Cálculos
- **Peso líquido:** Extraído do campo JSON `vlr_pl`
- **Total em dólar:** Extraído do campo JSON `total_dolar`
- **Câmbio:** Calculado como `eaa01totDoc / total_dolar`

### Ordenação
- Ordenado por nome do país e número do documento

## 🔧 Métodos Principais

### `getNomeTarefa()`
Retorna o nome descritivo da tarefa/relatório.

### `criarValoresIniciais()`
Configura os valores padrão dos filtros (mês atual).

### `executar()`
Método principal que orquestra a execução do relatório:
1. Obtém filtros
2. Busca empresa
3. Prepara parâmetros
4. Busca dados
5. Gera PDF

### `buscarDados(LocalDate[] emissao)`
Executa a consulta SQL principal com os filtros aplicados.

## 📊 Estrutura de Saída

**Campos Retornados (SQL):**
- `aag01nome` – Nome do país
- `abb01num` – Número do documento
- `abb01data` – Data do documento
- `abe01nome` – Nome do cliente
- `pesoLiq` – Peso líquido (numeric)
- `eaa01totDoc` – Valor total do documento
- `eaa01obsGerais` – Observações gerais
- `aag10id` – ID da moeda
- `totalDolar` – Valor total em dólar (numeric)
- `dolar` – Taxa de câmbio calculada

**Formato de Saída:**
- PDF gerado pelo método `gerarPDF()`

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` – Utilitários de data e coleções
- `sam.model.entities.aa` – Entidade Aac10 (empresa)
- `sam.server.samdev.relatorio` – Classes base para relatórios
- `java.time` – Manipulação de datas

## 📝 Observações Técnicas

### Consulta SQL
- Utiliza campos JSON (`eaa01json ->> 'campo'`)
- Conversão explícita para numérico (`::numeric`)
- Tratamento de valores nulos com `coalesce`
- Junções múltiplas com tabelas relacionadas

### Filtros Dinâmicos
- `obterWherePadrao("eaa01", "where")` – Aplica filtros padrão do sistema
- Condicionais para datas de emissão (início e fim)

### Formatação de Datas
- Utiliza `DateTimeFormatter.ofPattern("dd/MM/yyyy")`
- Período formatado como "dd/MM/yyyy à dd/MM/yyyy"

### Dados Monetários
- Assume que `total_dolar` está armazenado no JSON do documento
- Calcula câmbio como divisão entre valor total e total em dólar

---

**Última Alteração:** 11/12/2025 às 16:30  
**Autor:** Bruno  
**Tipo:** Relatório de Exportação  
**Versão:** 1.0  
**Módulo:** Linhasita  
**Código Fonte:** [SRF_ExportacaoPorPaises.groovy]