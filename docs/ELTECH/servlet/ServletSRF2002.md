# ServletSRF2002

## 📖 Descrição
Servlet responsável por processar e persistir as configurações do relatório SRF2002, gravando os parâmetros recebidos da interface em formato JSON no banco de dados.

## 🎯 Finalidade
Receber parâmetros de configuração da tela do relatório SRF2002, validá-los e armazená-los no repositório correspondente (tabela `Aba2001`) para uso posterior na geração de relatórios.

## 👥 Público-Alvo
- Desenvolvedores Backend
- Analistas de Sistemas
- Administradores de Banco de Dados

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Aba20` – Metadados do relatório (código SRF2002)
- `Aba2001` – Configurações do relatório (campos JSON)
- `Abm0101` – Entidade de suporte (não utilizada diretamente)

## ⚙️ Parâmetros do Servlet

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| srf2002_entretro | Integer | Sim | Configuração de entrada retroativa |
| srf2002_entretroultmes | Integer | Sim | Entrada retroativa último mês |
| srf2002_tpsaida | Integer | Sim | Tipo de saída |
| srf2002_consisaidaretro | Integer | Sim | Consistência de saída retroativa |
| srf2002_saidaultmes | Integer | Sim | Saída último mês |
| srf2002_mediaunitultfci | Integer | Sim | Média unitária último FCI |
| srf2002_cizero | Integer | Sim | Configuração CI zero |
| srf2002_attorigmerc | Integer | Sim | Atualização de origem de mercado |

## 🔄 Fluxo do Processo

### 1. **Recebimento dos Dados**
- Leitura do corpo da requisição HTTP
- Conversão do JSON para `TableMap`

### 2. **Busca do Relatório (Aba20)**
- Consulta na tabela `Aba20` pelo código "SRF2002"
- Validação da existência do registro

### 3. **Busca das Configurações (Aba2001)**
- Consulta na tabela `Aba2001` pelo ID do relatório (`aba2001rd`)
- Recuperação do campo JSON existente

### 4. **Atualização dos Parâmetros**
- Atualização dos valores no objeto `TableMap` (aba2001json)
- Inclusão de todos os parâmetros recebidos

### 5. **Persistência**
- Início de transação (se não existir)
- Persistência do objeto `Aba2001`
- Commit da transação

## ⚠️ Regras de Negócio

### Validações
- O relatório deve estar cadastrado em `Aba20` com código "SRF2002"
- Deve existir um registro correspondente em `Aba2001`
- Todos os parâmetros são obrigatórios e do tipo inteiro

### Transações
- A transação é iniciada apenas se não houver uma em aberto
- Commit é realizado após a persistência

## 🔧 Métodos Principais

### `getNome()`
Retorna o nome do servlet: "SRF2002".

### `getMetadata()`
Retorna `null` (não utiliza metadados de dashboard).

### `executar()`
Método principal que orquestra o processamento da requisição:
1. Lê o corpo da requisição
2. Busca o relatório e suas configurações
3. Atualiza os parâmetros
4. Persiste no banco de dados

## 📊 Estrutura de Saída
- **Tipo:** `ResponseEntity<Object>`
- **Conteúdo:** Nenhum corpo de resposta específico (apenas status HTTP implícito)
- **Efeito:** Persistência dos dados no banco de dados

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Criteria e consultas ao banco
- `multitec.utils` – Utilitários (`TableMap`, `JSonMapperCreator`)
- `sam.dto.samdev` – DTOs de dashboard
- `sam.model` – Entidades (`Aba20`, `Aba2001`, `Abm0101`)
- `sam.server.samdev.relatorio` – Classe base `ServletBase`
- `org.springframework.http` – `ResponseEntity`
- `com.fasterxml.jackson` – Serialização JSON

## 📝 Observações Técnicas

### Tratamento de JSON
- Utiliza `JSonMapperCreator` para desserialização
- Campos são armazenados no formato `TableMap` (mapa chave-valor)

### Gerenciamento de Sessão
- Utiliza a sessão do `ServletBase` (`getSession()`)
- Controle manual de transações

### Identificação do Relatório
- Código fixo: "SRF2002"
- Relacionamento via `aba2001rd` (referência a `Aba20.aba20id`)

### Campos Atualizados (aba2001json)
- `srf2002_entretro`
- `srf2002_entretroultmes`
- `srf2002_tpsaida`
- `srf2002_consisaidaretro`
- `srf2002_saidaultmes`
- `srf2002_mediaunitultfci`
- `srf2002_cizero`
- `srf2002_attorigmerc`

---

**Última Alteração:** 20/09/2025 às 15:30  
**Autor:** Nagyla  
**Tipo:** Servlet de Relatório  
**Versão:** 1.0  
**Código Fonte:** [ServletSRF2002.groovy]