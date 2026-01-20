# SCE – Registro de Inventário – Modelo P7 – El Tech

## 📖 Descrição
Relatório oficial de **Registro de Inventário – Modelo P7**, utilizado para atender exigências fiscais e contábeis, apresentando a posição de inventário por **grupos, subgrupos (graus)** e itens, com totalizações, médias, unidades de medida e termos legais de abertura e encerramento de livro.

## 🎯 Finalidade
Permitir a escrituração formal do inventário, possibilitando:
- Atendimento à legislação fiscal e contábil
- Emissão do Livro de Registro de Inventário
- Controle por grupos (grau 1, grau 2 e grupo completo)
- Apuração de quantidades, valores totais e médias
- Geração de termos de abertura e encerramento do livro

## 👥 Público-Alvo
- Contabilidade
- Fiscal
- Auditoria
- Administração
- Órgãos fiscalizadores

## 📊 Dados e Fontes

### Tabelas Principais
- Bcb10 – Inventário
- Bcb11 – Itens do Inventário
- Abm40 – Grupo de Inventário
- Abm01 – Item / Produto
- Abe01 – Entidade
- Aac10 – Empresa
- Aac1002 – Inscrição Estadual por UF
- Aag0201 – Município
- Aag02 – UF

### Entidades Envolvidas
- Empresa emissora
- Grupos de inventário
- Itens inventariados
- Clientes / Entidades (quando aplicável)

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição |
|---------|------|-------------|-----------|
| inventario | Long | Sim | ID do inventário |
| grupos | Lista (Long) | Não | Filtro de grupos de inventário |
| livroNum | Integer | Sim | Número do livro |
| livroPag | Integer | Sim | Página inicial do livro |
| impressao | Integer | Sim | 0 = Página / 1 = Folha |
| imprimir | Integer | Sim | 0 = Livro / 1 = Termo Abertura / 2 = Termo Encerramento |
| resumo | Integer | Não | 0 = Sem resumo / 1 = Grau 1 / 2 = Grau 2 |
| totUniMed | Boolean | Não | Totalizar por unidade de medida |
| totQtd | Boolean | Não | Totalizar quantidades |
| rascunho | Boolean | Sim | Define se o relatório é rascunho |
| data | LocalDate | Sim | Data do inventário |
| dataTermo | String | Sim | Data por extenso para termo |
| repLeg1 | String | Não | Representante legal |
| contab1 | String | Não | Contador |
| contab2 | String | Não | CRC do contador |

## 📋 Campos do Relatório

### Campos Principais
| Campo | Descrição |
|------|-----------|
| abm40codigo | Código do grupo |
| abm40descr | Descrição do grupo |
| abm01codigo | Código do item |
| abm01descr | Descrição do item |
| abm01tipo | Tipo do item |
| bcb11unid | Unidade de medida |
| bcb11ncm | NCM |
| bcb11qtde | Quantidade |
| bcb11unit | Valor unitário |
| bcb11total | Valor total |
| bcb10data | Data do inventário |

### Campos de Totalização
| Campo | Descrição |
|------|-----------|
| totComp | Total do grupo |
| totGrau1 | Total do Grau 1 |
| totGrau2 | Total do Grau 2 |
| bcb11Media | Média do grupo |
| mediaGrau1 | Média Grau 1 |
| mediaGrau2 | Média Grau 2 |
| qtdUniMed | Quantidade por unidade |
| totUniMed | Total por unidade |

## 🔄 Fluxo do Processo

### 1. Inicialização
- Define valores padrão
- Preenche dados da empresa
- Gera data do termo por extenso
- Define assinaturas e responsáveis

### 2. Validação de Execução
- Verifica tipo de impressão (Livro / Termos)
- Define layout do relatório (R1 ou R2)

### 3. Processamento Principal
- Busca itens do inventário por grupo e data
- Ordena por código de grupo e item
- Cria hierarquia:
    - Grau 1 (2 primeiros dígitos)
    - Grau 2 (4 primeiros dígitos)
    - Grupo completo

### 4. Totalizações
- Soma valores por grupo
- Soma valores por grau
- Calcula médias por quantidade
- Totaliza por unidade de medida (opcional)

### 5. Resumo (Opcional)
- Gera resumo por Grau 1, Grau 2 ou ambos
- Apresenta totais consolidados

### 6. Termos Oficiais
- Termo de Abertura
- Termo de Encerramento
- Atualiza número de páginas do livro no inventário

### 7. Pós-Execução
- Atualiza livro e página no inventário
- Retorna PDF para download

## ⚠️ Regras de Negócio

### Agrupamentos
- Grau 1: primeiros 2 caracteres do código do grupo
- Grau 2: primeiros 4 caracteres do código do grupo
- Grupo: código completo

### Cálculos
- Total = soma de `bcb11total`
- Média = total / quantidade
- Quantidade acumulada por grau e grupo

### Livro Fiscal
- Se não for rascunho, atualiza:
    - Número do livro
    - Número da última página utilizada

### Unidades de Medida
- Totalização opcional
- Pode ser feita por grupo ou por grau
- Restringe repetição de unidades já listadas

## 🎨 Saídas Disponíveis

| Formato | Descrição |
|-------|-----------|
| PDF | Livro de Registro de Inventário – Modelo P7 |

## 🔧 Dependências

### Frameworks e Bibliotecas
- JasperReports
- MultiORM
- sam.server.samdev.relatorio
- sam.model.entities

### Infraestrutura
- Sessão ORM ativa
- Templates Jasper (`SCE_RegistroDeInventarioP7_R1` e `R2`)

## 📝 Observações Técnicas
- Uso intensivo de SQL nativo para performance
- Controle rigoroso de totalizações hierárquicas
- Compatível com exigências fiscais brasileiras
- Relatório sensível a parâmetros de impressão e rascunho
- Atualiza dados persistidos do inventário após impressão oficial
