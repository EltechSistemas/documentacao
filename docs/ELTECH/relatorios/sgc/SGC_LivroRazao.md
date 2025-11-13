# SGC_LivroRazao.md

## 📖 Descrição
Sistema de geração do Livro Razão contábil para a Linhasita, responsável pela impressão do registro cronológico de todos os lançamentos contábeis por conta.

## 🎯 Finalidade
Fornecer um relatório completo do Livro Razão para acompanhamento dos lançamentos contábeis, saldos e movimentações por conta no período.

## 👥 Público-Alvo
- Departamento Contábil
- Controladoria
- Auditores internos e externos
- Diretoria Financeira

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SGC_LivroRazao_R2` - Layout retrato
- Template Jasper `SGC_LivroRazao_R3` - Layout paisagem  
- Template Jasper `SGC_LivroRazao_R1` - Termos de abertura/encerramento

**Localização:** `eltech/relatorios/sgc/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `ABC10` - Plano de contas
- `EBB05` - Lançamentos contábeis
- `EBB02` - Saldos contábeis
- `EBA40` - Livros contábeis
- `AAC10` - Empresas/Filiais

**Entidades Envolvidas:**
- `Abc10` - Contas contábeis
- `Ebb05` - Lançamentos
- `Ebb02` - Saldos mensais
- `Eba40` - Controle de livros
- `Aac10` - Empresas

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| imprimir | Integer | Sim | Tipo de impressão | 0-Livro Razão, 1-Termo Abertura, 2-Termo Encerramento |
| dataInicial | LocalDate | Sim | Data inicial do período | dd/MM/yyyy |
| dataFinal | LocalDate | Sim | Data final do período | dd/MM/yyyy |
| contaContabilInicial | String | Não | Conta contábil inicial | Código da conta |
| contaContabilFinal | String | Não | Conta contábil final | Código da conta |
| rascunho | Boolean | Sim | Modo rascunho | true/false |
| ctaComMovimento | Boolean | Sim | Apenas contas com movimento | true/false |

## 📋 Tipos de Impressão

### Livro Razão (0)
- Lançamentos detalhados por conta
- Saldos anteriores e atuais
- Movimentações a débito e crédito
- Histórico dos lançamentos

### Termo de Abertura (1)
- Documento formal de abertura do livro
- Identificação da empresa
- Período contábil
- Assinaturas responsáveis

### Termo de Encerramento (2)
- Documento formal de encerramento
- Confirmação do período
- Encerramento para novo livro
- Assinaturas responsáveis

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica período informado
   - Valida contas contábeis selecionadas
   - Confirma configurações do plano de contas

2. **Busca de Dados Contábeis**
   - Obtém saldos anteriores do período
   - Busca lançamentos a débito
   - Busca lançamentos a crédito
   - Calcula saldos atualizados

3. **Processamento do Razão**
   - Agrupa lançamentos por conta
   - Calcula saldos em tempo real
   - Ordena por código de conta e data

4. **Controle de Livros**
   - Verifica existência de livro em aberto
   - Calcula numeração de páginas
   - Gerencia termos de abertura/encerramento

5. **Geração do Relatório**
   - Aplica formatação específica
   - Insere quebras de página por conta
   - Gera PDF final

## ⚠️ Regras de Negócio

### Validações
- Estrutura do plano de contas deve estar configurada
- Grau da empresa deve ser definido
- Códigos das empresas devem estar atualizados
- Não permite livro em aberto com número diferente

### Cálculos Contábeis
- **Saldo Anterior:** Saldo do mês anterior + lançamentos do início do mês
- **Débitos:** Soma de todos os lançamentos a débito
- **Créditos:** Soma de todos os lançamentos a crédito  
- **Saldo Atual:** Saldo anterior + débitos - créditos

### Controle de Livros
- Livro deve estar em aberto para novas impressões
- Termo de encerramento gera novo número de livro
- Numeração de páginas é sequencial e controlada

## 🎨 Opções de Formatação

| Configuração | Descrição | Valores |
|-------------|-----------|---------|
| Orientação | Layout da página | 0-Retrato, 1-Paisagem |
| Salto | Quebra por conta | 0-Linhas, 1-Página |
| Rascunho | Modo não oficial | true/false |

## 🔧 Dependências

**Parâmetros do Sistema:**
- `GRAUEMPRESA` - Grau da empresa no plano
- `ESTRCODCONTA` - Estrutura do código de conta
- `EMPRATUALIZADAS` - Códigos das empresas atualizadas
- `EB_ATUALIZARCTAS` - Indicador de atualização

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multiorm` - Acesso a dados
- `java.time` - Manipulação de datas

## 📝 Observações Técnicas

- Suporte a múltiplas empresas no mesmo plano
- Cálculo preciso de saldos considerando períodos parciais
- Controle rigoroso de numeração de livros e páginas
- Opção de emitir apenas contas com movimento
- Formatação profissional para documentos contábeis
- Integração com sistema de livros fiscais
- Validações de consistência dos dados contábeis