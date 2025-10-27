# SGC - Lançamento Contábil

## 📖 Descrição
Fórmula para processamento e formatação automática do histórico de lançamentos contábeis no Sistema de Gestão Contábil (SGC), substituindo coringas por informações dinâmicas do documento.

## 🎯 Finalidade
Automatizar a geração do histórico contábil substituindo placeholders (`$1`, `$2`, etc.) por dados reais do documento, central, entidade e eventos relacionados.

## 👥 Público-Alvo
- Departamento Contábil
- Tesouraria
- Controladoria
- Analistas Financeiros

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Ebb05` - Lançamentos contábeis
- `Abb01` - Cabeçalho de documentos (central)
- `Aah01` - Tipos de documento
- `Abe01` - Entidades (clientes/fornecedores)
- `Abh21` - Eventos contábeis
- `Eca01` - Classificação patrimonial
- `Eba02` - Planilha de lançamento programado

**Entidades Envolvidas:**
- `Ebb05` - Lançamento contábil
- `Abb01` - Documento central
- `Aah01` - Tipo de documento
- `Abe01` - Entidade
- `Abh21` - Evento
- `Eca01` - Classificação patrimonial
- `Eba02` - Lançamento programado

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| ebb05 | Ebb05 | Sim | Lançamento contábil a ser processado |
| abh21 | Abh21 | Não | Evento contábil associado |

## 🔄 Fluxo do Processo

### 1. **Inicialização**
- Obtém o lançamento contábil (Ebb05)
- Verifica existência de histórico para processamento
- Inicializa variáveis de entidades relacionadas

### 2. **Busca de Dados da Central**
- Recupera dados do documento central (Abb01)
- Busca tipo de documento (Aah01) relacionado
- Obtém dados da entidade (Abe01) vinculada
- Recupera classificação patrimonial (Eca01) se existir
- Busca dados do lançamento programado (Eba02)

### 3. **Busca de Dados do Evento**
- Recupera informações do evento contábil (Abh21)
- Prioriza evento passado como parâmetro
- Fallback para evento vinculado ao lançamento

### 4. **Processamento de Coringas**
Substitui placeholders no histórico por valores reais:

| Coringa | Descrição | Fonte |
|---------|-----------|-------|
| `$1` | Nome abreviado do documento | Aah01.aah01na |
| `$2` | Número do documento | Abb01.abb01num |
| `$3` | Série do documento | Abb01.abb01serie |
| `$4` | Data do documento (dd/MM/yyyy) | Abb01.abb01data |
| `$5` | Nome da entidade | Abe01.abe01nome |
| `$6` | Parcela do documento | Abb01.abb01parcela |
| `$7` | Nome da classificação patrimonial | Eca01.eca01nome |
| `$8` | Observação do SGC | Eba02.eba02json.obs_sgc |
| `$9` | Nome do evento | Abh21.abh21nome |

### 5. **Atualização do Histórico**
- Aplica todas as substituições
- Atualiza o campo `ebb05historico` com o resultado final

## ⚠️ Regras de Negócio

### Validações
- Processa apenas se `ebb05historico` não for nulo
- Considera central apenas se `ebb05.ebb05central` existir
- Trata valores nulos com strings vazias para evitar erros

### Formatação de Dados
- **Datas**: Formato "dd/MM/yyyy"
- **Números**: Conversão direta para string
- **Campos JSON**: Acesso ao campo `obs_sgc` específico
- **Valores Nulos**: Substituídos por string vazia

### Prioridade de Eventos
1. Evento passado como parâmetro (`abh21`)
2. Evento vinculado ao lançamento (`ebb05.abh21id`)

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de formatação do histórico.

### Buscas no Banco
- Consultas otimizadas com `getAcessoAoBanco().buscarRegistroUnico()`
- Filtros por ID das entidades
- Aplicação de `WHERE` padrão para entidade Abe01

## 📊 Estrutura de Saída

**Ebb05 (Lançamento Contábil):**
- `ebb05historico` - Histórico formatado com coringas substituídos

## 🔧 Dependências

**Bibliotecas:**
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `sam.server.samdev` - Base de fórmula e utilitários
- `java.time.format` - Formatação de datas

**Entidades:**
- `FormulaBase` - Classe base para fórmulas
- `Parametro` - Parâmetros de consulta ao banco

## 📝 Observações Técnicas

- Utiliza `DateTimeFormatter` para formatação consistente de datas
- Processamento seguro de valores nulos em todas as substituições
- Consultas otimizadas com busca única por entidade
- Suporte a campos JSON para observações customizadas
- Integração com sistema de lançamentos programados (Eba02)
- Compatível com classificação patrimonial (Eca01)

---

**Última Alteração:** 23/10/2025 10:48  
**Autor:** Bruno  
**Tipo:** Fórmula de Lançamento Contábil (LCTO_SGC)  
**Versão:** 1.0