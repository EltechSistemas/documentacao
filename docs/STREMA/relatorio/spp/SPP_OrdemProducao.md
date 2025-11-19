# SPP_OrdemProducao - Relatório de Ordem de Produção

## 📖 Descrição
Relatório de ordens de produção com filtros por status, número, tipo, entidade, itens, lotes e data de emissão, gerando PDF com informações completas do processo produtivo.

## 🎯 Finalidade
Fornecer uma visão consolidada das ordens de produção, permitindo acompanhamento do status, componentes utilizados, quantidades planejadas e realizadas, além de informações específicas do processo.

## 👥 Público-Alvo
- Departamento de Produção
- PCP (Planejamento e Controle de Produção)
- Gestores Industriais
- Supervisores de Produção

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| status | Integer | Sim | Status da ordem de produção | 0=Todos |
| numIni | Integer | Não | Número inicial | 0-999999 |
| numFim | Integer | Não | Número final | 0-999999 |
| tipo | List<Long> | Não | Tipos de documento | IDs tipos documento |
| entidade | List<Long> | Não | Entidades específicas | IDs entidades |
| itens | List<Long> | Não | Itens específicos | IDs itens |
| lotes | List<String> | Não | Lotes específicos | Códigos de lote |
| dataEmissao | LocalDate[] | Não | Período de emissão | Data inicial e final |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abb01num | Número do documento | Integer |
| abb01data | Data do documento | Date |
| bab01dtE | Data da ordem | Date |
| rep | Representante | String |
| entCodigo | Código da entidade | String |
| entNome | Nome da entidade | String |
| bab01qt | Quantidade | BigDecimal |
| prodDesc | Descrição do produto | String |
| prodCodigo | Código do produto | String |
| abp20bomNumRev | Número revisão BOM | String |
| abp20bomDtRev | Data revisão BOM | Date |
| compCodigo | Código componente | String |
| compDesc | Descrição componente | String |
| aam06codigo | Unidade medida | String |
| bab0101qtP | Qtde planejada | BigDecimal |
| bab0101qtA | Qtde realizada | BigDecimal |
| bab01obs | Observações | String |
| bab01detProd | Detalhes produção | String |
| carga | Informações de carga | String |
| transporte | Informações transporte | String |
| etiqueta | Informações etiqueta | String |
| outros_itens | Outros itens | String |
| prioridade | Prioridade | String |

## 🔄 Fluxo do Processo

### 1. **Validação de Parâmetros**
- Define valores padrão (status = 0)
- Processa intervalos numéricos
- Converte lista de lotes (separados por ";")

### 2. **Configuração do Relatório**
- Localiza arquivo de logo
- Obtém dados da empresa ativa
- Constrói endereço completo da empresa

### 3. **Construção da Consulta**
- Monta SQL dinâmico com filtros aplicados
- Aplica cláusulas WHERE conforme parâmetros
- Define ordenação por número e descrição do componente

### 4. **Busca de Dados**
- Consulta ordens de produção no banco
- Inclui informações de produto, componentes e entidades
- Extrai campos JSON específicos

### 5. **Geração de Saída**
- Passa parâmetros para template JasperReports
- Gera PDF com dados consultados
- Retorna arquivo para download

## ⚠️ Regras de Negócio

### Filtros de Busca
- **Status**: Filtro obrigatório, padrão = 0 (todos)
- **Números**: Intervalo inclusivo (inicial ≤ número ≤ final)
- **Lotes**: Separados por ponto e vírgula, espaços removidos
- **Datas**: Período inclusivo (data inicial ≤ data ≤ data final)

### Estrutura de Dados
- Join com múltiplas tabelas (bab01, abb01, abe01, abp20, abm01)
- Componentes listados individualmente por ordem
- Campos JSON extraídos para informações específicas

### Ordenação
- Primária: Número do documento (abb01num)
- Secundária: Descrição do componente (comp.abm01descr)

## 🎨 Saídas Disponíveis

| Formato | Descrição | Template | Método |
|---------|-----------|----------|---------|
| PDF | Layout padrão | `SPP_OrdemProducao` | `gerarPDF()` |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios PDF
- `br.com.multitec.utils` - Utilitários e coleções

**Recursos:**
- Arquivo de logo: `/resources/strema/relatorios/spp/Logo.png`
- Template JasperReports

## 📝 Observações Técnicas

### Consulta SQL
- SQL dinâmico com múltiplos filtros opcionais
- Uso de JSON extraction para campos específicos
- Join complexo com aliases para entidades e itens

### Parâmetros do Relatório
- **LOGO**: Caminho completo do arquivo de logo
- **ENDERECO**: Endereço formatado da empresa

### Tratamento de Dados
- Limpeza de espaços em lista de lotes
- Formatação de endereço multi-linha
- Valores padrão para parâmetros numéricos

### Performance
- Filtros aplicados no banco de dados
- Paginação tratada pelo JasperReports
- Consulta otimizada com índices apropriados