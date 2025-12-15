# SPP_EtiquetaProducaoCaixa.md
# SPP - Etiqueta de Produção Caixa

## 📖 Descrição
Relatório responsável pela geração de etiquetas de produção para caixas, incluindo QR Code, dados do produto, lote, série e informações de validade. O relatório também gerencia o status de impressão das etiquetas.

## 🎯 Finalidade
- Gerar etiquetas de produção para caixas
- Incluir QR Code com número da etiqueta
- Imprimir informações de lote, série e validade
- Gerenciar status de impressão das etiquetas
- Filtrar etiquetas por diversos critérios

## 👥 Público-Alvo
- Produção
- Controle de qualidade
- Expedição
- Almoxarifado

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Abm70` - Etiquetas
- `Abm01` - Itens/Produtos
- `Aam06` - Unidades de medida
- `Abb01` - Centrais de documento
- `Aah01` - Tipos de documento
- `Aac10` - Empresa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| numIniEtiqueta | Integer | Não | Número inicial da etiqueta |
| numFimEtiqueta | Integer | Não | Número final da etiqueta |
| data | LocalDate | Sim | Data das etiquetas |
| imprimir | Integer | Não | Status de impressão (0=não impresso, 1=impresso) |
| itens | List<Long> | Não | IDs dos itens para filtrar |
| tipoDoc | List<Long> | Não | IDs dos tipos de documento |
| numIniDoc | Integer | Não | Número inicial do documento |
| numFimDoc | Integer | Não | Número final do documento |
| abm70ids | List<Long> | Não | IDs específicos das etiquetas |
| lote | String | Não | Lotes para filtrar (separados por ;) |
| serie | String | Não | Séries para filtrar (separados por ;) |

## 🔄 Fluxo do Processo

### 1. **Recebimento e Validação dos Parâmetros**
- Leitura dos filtros fornecidos
- Validação e normalização dos dados
- Definição de valores padrão

### 2. **Busca de Dados**
- Consulta ao banco com filtros aplicados
- Carregamento de informações de etiquetas, produtos e documentos
- Ordenação por número da etiqueta

### 3. **Geração de QR Codes**
- Criação de QR Code para cada etiqueta
- Utilização do número da etiqueta como conteúdo
- Conversão para imagem compatível com JasperReports

### 4. **Preparação dos Dados para Impressão**
- Estruturação dos dados em TableMap
- Adição de imagens QR Code aos dados
- Preparação do DataSource para o relatório

### 5. **Atualização de Status**
- Marcação das etiquetas como impressas (status = 1)
- Persistência no banco de dados

### 6. **Geração do PDF**
- Renderização do relatório Jasper
- Retorno do PDF para download/visualização

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de geração do relatório.

**Fluxo:**
1. Coleta e valida parâmetros
2. Busca caminho do logo da empresa
3. Carrega dados das etiquetas (`buscarEtiquetas`)
4. Gera QR Code para cada etiqueta (`gerarQrCode`)
5. Atualiza status de impressão (`gravarStatusImpressaoEtiquetas`)
6. Gera PDF com os dados processados

### `buscarEtiquetas()`
Executa consulta SQL para buscar etiquetas com base nos filtros aplicados.

**Parâmetros:**
- Todos os parâmetros de filtro do relatório

**Retorno:** `List<TableMap>` com dados das etiquetas

### `gerarQrCode(String qrCode)`
Gera imagem QR Code a partir de uma string.

**Parâmetros:**
- `qrCode`: String a ser codificada no QR Code

**Retorno:** `Image` - Imagem do QR Code

### `gravarStatusImpressaoEtiquetas(List<Long> abm70ids)`
Atualiza o status das etiquetas para "impresso".

**Parâmetros:**
- `abm70ids`: Lista de IDs das etiquetas a serem atualizadas

## 📊 Estrutura de Dados

### **Campos Retornados na Consulta:**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| seq | Integer | Sequência por lote |
| abm70id | Long | ID da etiqueta |
| abm70num | Integer | Número da etiqueta |
| abm70dv | String | Dígito verificador |
| abb01num | Integer | Número do documento |
| abm70qt | BigDecimal | Quantidade |
| abm70lote | String | Número do lote |
| abm70validade | Date | Data de validade |
| abm70data | Date | Data da etiqueta |
| abm70fabric | Date | Data de fabricação |
| abm70serie | String | Número da série |
| abm01codigo | String | Código do item |
| abm01descr | String | Descrição do item |
| abm01na | String | Nome alternativo |
| abm01pesobruto | BigDecimal | Peso bruto |
| abm01pesoliq | BigDecimal | Peso líquido |
| abm01gtin | String | GTIN/EAN |
| aam06codigo | String | Código da unidade de medida |
| tara | BigDecimal | Tara (do campo livre JSON) |

### **Parâmetros do Relatório Jasper:**
| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| LOGO | String | Caminho do arquivo de logo |
| CNPJ | String | CNPJ da empresa |
| imgqrcode | Image | Imagem do QR Code (por registro) |

## ⚠️ Regras de Negócio

### **Filtros Aplicados por Padrão**
- Exclusão de etiquetas com quantidade = -1
- Filtro por empresa ativa (`obterWherePadrao`)
- Ordenação por número da etiqueta

### **Formatação de Parâmetros**
- Lotes: separados por `;`, espaços removidos
- Séries: separadas por `;`, espaços removidos
- Valores nulos ou vazios são tratados como não aplicáveis

### **Geração de QR Code**
- Tamanho fixo: 400x400 pixels
- Conteúdo: número da etiqueta como string
- Formato: QR Code padrão

### **Status de Impressão**
- **0**: Não impresso
- **1**: Impresso
- Atualização ocorre apenas se houver etiquetas processadas

### **Valores Padrão**
- Data padrão: data atual
- Filtros numéricos: 0 quando não informados
- Status de impressão: todos quando não filtrado

## 🔧 Dependências

**Bibliotecas:**
- `multitec.utils` - Utils e TableMap
- `google.zxing` - Geração de QR Code (core e j2se)
- `java.awt` - Manipulação de imagens
- `sam.model` - Entidades do sistema
- `sam.server.samdev.relatorio` - Base para relatórios
- `sam.server.samdev.utils` - Parametro

**Arquivos:**
- `Logo.png` - Logo da empresa (no mesmo pacote da classe)
- `SPP_EtiquetaProducaoCaixa.jrxml` - Template JasperReports

## 📝 Observações Técnicas

### **Caminho do Logo**
```java
String logo = pasta.getCanonicalPath().replace("\\" , "/") + 
              "/samdev/resources/" + nomeClasse + "/Logo.png";