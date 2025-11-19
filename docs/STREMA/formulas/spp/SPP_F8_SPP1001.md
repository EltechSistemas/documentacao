# SPP_F8_SPP1001

## 📖 Descrição
Fórmula F8 para consulta de ordens de produção no sistema SPP (Sistema de Produção e Planejamento). Implementa busca paginada com filtros dinâmicos para visualização de dados de produção.

## 🎯 Finalidade
Fornecer interface de consulta rápida (F8) para ordens de produção, permitindo filtros personalizados e busca textual sobre dados de produção.

## 👥 Público-Alvo
- Planejamento de Produção
- Controle de Produção
- PCP (Planejamento e Controle de Produção)
- Supervisores de Produção

## ⚙️ Parâmetros/Configurações

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| requisicao | RequisicaoDoF8 | Sim | Objeto com parâmetros da consulta F8 | Paginação, filtros e buscas |

## 📊 Estrutura de Dados

### Colunas Retornadas
| Campo | Descrição | Tipo |
|-------|-----------|------|
| aah01codigo | Código do tipo de documento | String |
| abb01num | Número do documento | Integer |
| abb01data | Data do documento | Date |
| bab01status | Status da ordem de produção | String |
| abp20bomCodigo | Código da lista de materiais (BOM) | String |
| abm01codigo | Código do item | String |
| abm01na | Número de alternativo do item | String |
| bab01qt | Quantidade da ordem | BigDecimal |
| abp10codigo | Código do processo | String |
| abp10descr | Descrição do processo | String |

### Tabelas Envolvidas
- `bab01` - Ordens de produção
- `abb01` - Cabeçalho de documentos
- `aah01` - Tipos de documento
- `abp20` - Listas de materiais (BOM)
- `abm01` - Itens (produtos)
- `abp10` - Processos de produção

## 🔄 Fluxo do Processo

1. **Configuração das Colunas**
   - Define colunas que serão exibidas no resultado
   - Mapeia campos do SAM para colunas do F8

2. **Processamento de Filtros**
   - Converte filtros da interface em cláusulas WHERE
   - Adiciona parâmetros para consulta segura

3. **Processamento de Buscas**
   - Converte critérios de busca textual em cláusulas OR
   - Combina múltiplos campos de busca

4. **Execução da Consulta**
   - Monta SQL final com paginação
   - Aplica filtros de segurança (obterWherePadrao)
   - Ordena por data e número decrescente

5. **Retorno dos Dados**
   - Estrutura resposta no formato F8
   - Inclui metadados das colunas e dados paginados

## ⚠️ Regras de Negócio

### Segurança
- Aplica `obterWherePadrao("bab01")` para restringir acesso aos dados
- Utiliza consultas parametrizadas para prevenir SQL injection

### Paginação
- Suporta paginação nativa do F8
- Mantém ordenação consistente entre páginas

### Filtros
- Combina múltiplos filtros com operador AND
- Suporta filtros complexos com múltiplos parâmetros

### Buscas
- Combina múltiplos critérios de busca com operador OR
- Permite busca textual em múltiplos campos

## 🎨 Saídas/Retornos

| Tipo | Descrição | Estrutura |
|------|-----------|-----------|
| RespostaDoF8 | Dados paginados + metadados | Lista de registros + definição de colunas |

## 🔧 Dependências

**Entidades do Sistema:**
- `bab01` - Ordem de produção
- `abb01` - Cabeçalho do documento
- `aah01` - Tipo de documento
- `abp20` - Lista de materiais (BOM)
- `abm01` - Item
- `abp10` - Processo

**Framework:**
- `RequisicaoDoF8` - Parâmetros da consulta
- `RespostaDoF8` - Estrutura de retorno
- `ColunaF8` - Definição de colunas

## 📝 Observações Técnicas

### Performance
- Utiliza paginação nativa do banco de dados
- Aplica índices através de ordenação por data e número
- Consulta otimizada com JOINs necessários apenas

### Manutenibilidade
- Código claro e estruturado
- Fácil adição de novas colunas
- Tratamento genérico de filtros e buscas

### Segurança
- Where padrão aplicado automaticamente
- Parâmetros tratados contra SQL injection
- Acesso restrito por permissões do sistema

**Última Atualização:** 13/11/2025
**Tipo de Fórmula:** F8 (Consulta Rápida)