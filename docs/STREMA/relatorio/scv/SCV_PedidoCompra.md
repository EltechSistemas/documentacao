# SCV_PedidoCompra

## 📖 Descrição
Classe `SCV_PedidoCompra` do sistema Strema responsável por gerar relatórios de **Pedidos de Compra**, permitindo análise detalhada dos pedidos emitidos, itens, valores e destinatários, além de informações sobre representantes e dados fiscais da empresa.

## 🎯 Finalidade
Fornecer informações completas de pedidos de compra, permitindo:
- Listar pedidos emitidos dentro de um período;
- Consultar itens, quantidades, valores e impostos (ICM, IPI, ST);
- Obter informações do cliente, endereço, frete e CNPJ/IE;
- Identificar representante responsável pelo pedido;
- Exportar relatório em PDF.

## 👥 Público-Alvo
- Compras e Suprimentos
- Financeiro e Contabilidade
- Gerência de Operações
- Representantes comerciais

## ⚙️ Configuração
**Classe Principal:** `SCV_PedidoCompra`  
**Pacote:** `strema.relatorios.scv`

**Dependências:**
- `multiorm` e `multitec.utils` – Utilitários de datas, mapas e critérios de pesquisa
- `sam.server.samdev.relatorio` – Framework de relatórios
- Entidades SAM: `Aac10`, `Aac1002`

## 📊 Dados e Fontes
**Tabelas principais consultadas:**
- `EAA01` – Documentos de pedidos de compra
- `ABB01` – Cabecalho de pedidos
- `ABE01` – Entidades (clientes e despachos)
- `AAB10` – Representantes
- `EAA0101`, `EAA0102`, `EAA0103` – Detalhes de endereço, cliente e itens
- `ABM01`, `AAM01`, `AAM06`, `ABE30` – Informações de itens, classe e unidades

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| numIni | Integer | Não | Número inicial do pedido |
| numFim | Integer | Não | Número final do pedido |
| pedCompra | Boolean | Não | Se `true` gera pedido de compra (44308805), se `false` gera pedido de exportação (44329831) |
| entidade | List<Long> | Não | Lista de clientes a filtrar |
| classeItens | List<Long> | Não | Filtra itens por classe |
| dataEmissao | LocalDate[] | Não | Intervalo de datas de emissão do pedido |
| dataEntrega | LocalDate[] | Não | Intervalo de datas de entrega do pedido |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|-------|-----------|------|
| PDF | Relatório PDF do pedido de compra com todos os detalhes | Arquivo |
| dados | Lista de pedidos processados | List<TableMap> |

**Campos principais de cada registro:**
- `abb01num` → Número do pedido
- `abb01data` → Data de emissão
- `entidade` → Nome e código do cliente
- `eaa0102ni` → Número de identificação (CNPJ/CPF)
- `eaa0102frete` → Tipo de frete
- `endereco completo` → Endereço completo do cliente
- `abm01codigo` e `abm01descr` → Código e descrição do item
- `eaa0103qtComl` → Quantidade comercial
- `eaa0103unit` → Valor unitário
- `icm_icm`, `st_icm`, `ipi_ipi` → Impostos aplicáveis
- `total_icm_icm`, `total_st_icm`, `total_ipi_ipi`, `total_vlr_frete_dest` → Totais do pedido
- `cp` → Centro de produção
- `despachoAbe01nome` → Nome do despacho (destino)
- `representante` → Nome do representante responsável
- `aab10eMail` → Email do representante

## 🔄 Fluxo do Processo

1. **Criação de valores iniciais**
   - `criarValoresIniciais()` define:
      - Intervalo do mês atual como padrão para `dataEmissao`
      - `pedCompra` como `true` por padrão

2. **Execução do relatório**
   - Coleta filtros do relatório (`numIni`, `numFim`, `pedCompra`, `entidade`, `classeItens`, datas)
   - Determina **tipo de pedido** (`pedCompra` → 44308805, outro → 44329831)
   - Monta caminho dos logos do relatório
   - Obtém informações da **empresa ativa** (`Aac10`) e inscrição estadual (`Aac1002`)
   - Monta **endereço completo** da empresa
   - Adiciona parâmetros para o IReport (`LOGO`, `LOGO_EXPOWER`, `ENDERECO`)
   - Chama `buscarDados()` para montar SQL e buscar registros
   - Gera PDF com `gerarPDF(dados)`

3. **Busca de dados**
   - `buscarDados()` monta SQL dinâmico:
      - Filtra por número de pedido, tipo, entidade, classe de itens, data de emissão e entrega
      - Join com tabelas de clientes, itens, municípios, unidades, despachos e representantes
      - Retorna lista de `TableMap` com todos os campos calculados

## ⚠️ Regras de Negócio

- Pedido de compra padrão (`pedCompra = true`) → tipo 44308805
- Pedido de exportação (`pedCompra = false`) → tipo 44329831
- Filtragem opcional por:
   - Número inicial e final do pedido
   - Cliente ou entidade
   - Classe de itens
   - Intervalos de datas de emissão e entrega
- Considera apenas movimentos ativos (`eaa01esMov = 0`) e documentos principais (`eaa0101principal = 1`)
- Totais de impostos e valores são calculados diretamente do JSON dos itens (`eaa0103json` e `eaa01json`)

## 🔄 Métodos Principais

### `executar()`
- Orquestra a execução:
   - Coleta filtros do usuário
   - Determina tipo de pedido
   - Obtém informações da empresa ativa e inscrição estadual
   - Monta parâmetros do relatório (`LOGO`, `ENDERECO`, etc.)
   - Busca dados detalhados (`buscarDados()`)
   - Gera PDF final

### `buscarDados(numIni, numFim, tipo, entidade, classeItens, dataEmissao, dataEntrega)`
- Monta SQL dinâmico aplicando filtros opcionais
- Faz joins com tabelas de pedidos, clientes, itens, representantes, unidades e impostos
- Converte campos JSON em valores numéricos
- Retorna lista de `TableMap` com dados prontos para relatório

### `criarValoresIniciais()`
- Define valores padrão para filtros do relatório
- Define o intervalo do mês atual como padrão para `dataEmissao`
- Define `pedCompra` como `true`

## 💡 Observações Técnicas
- Suporte a filtros dinâmicos múltiplos
- Conversão de campos JSON para valores numéricos (`icm`, `ipi`, `st`)
- Uso de `LocalDate` para manipulação de datas de emissão e entrega
- Endereço da empresa é concatenado com informações de CEP, telefone e site
- Suporte a geração de PDF via `gerarPDF`