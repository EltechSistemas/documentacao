# SGC_BalancoPatrimonialDRE.md

## 📖 Descrição
Sistema de geração de relatórios contábeis para a Linhasita, incluindo Balanço Patrimonial, Demonstração do Resultado do Exercício (DRE) e Notas Explicativas.

## 🎯 Finalidade
Fornecer relatórios contábeis completos e padronizados para atendimento às obrigações legais e análise gerencial da situação patrimonial e de resultados da empresa.

## 👥 Público-Alvo
- Departamento Contábil
- Controladoria
- Diretoria Financeira
- Auditores externos
- Conselho de Administração

## ⚙️ Configuração
**Recursos Necessários:**
- Template Jasper `SGC_BalancoPatrimonialDRE` - Layout dos relatórios
- Parâmetros contábeis configurados

**Localização:** `linhasita/relatorios/sgc/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `ABC10` - Plano de contas
- `EBB02` - Saldos contábeis
- `EBA10` - Agrupamentos de contas
- `EBA30` - Notas explicativas
- `EBA40` - Livros contábeis
- `AAC10` - Empresas/Filiais

**Entidades Envolvidas:**
- `Abc10` - Contas contábeis
- `Ebb02` - Saldos mensais
- `Eba10` - Agrupamentos
- `Eba30` - Notas
- `Aac10` - Empresas

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-------------------|
| emitir | Integer | Sim | Tipo de relatório | 0-BP, 1-DRE, 2-DRE Mensal, 3-Notas |
| data1 | String | Sim | Data de referência 1 | MM/aaaa |
| data2 | String | Não | Data de referência 2 | MM/aaaa |
| data3 | String | Não | Data de referência 3 | MM/aaaa |
| anexarBalanceteAoDiario | Boolean | Não | Vincular ao livro diário | true/false |

## 📋 Tipos de Relatório

### Balanço Patrimonial (0)
- Ativo e Passivo circulante/não circulante
- Patrimônio líquido
- Comparativo entre períodos (até 3 datas)

### DRE - Demonstração do Resultado (1)
- Receitas operacionais
- Custos e despesas
- Resultado líquido do exercício
- Agrupamentos específicos por conta

### DRE Mensal (2)
- Versão mensal da DRE
- Foco em resultados do mês

### Notas Explicativas (3)
- Informações complementares
- Esclarecimentos sobre demonstrações

## 🔄 Fluxo do Processo

1. **Validação de Parâmetros**
   - Verifica datas de referência
   - Valida consistência entre períodos
   - Confirma configurações contábeis

2. **Configuração do Ambiente**
   - Carrega dados da empresa
   - Obtém estrutura do plano de contas
   - Configura assinaturas e termos

3. **Processamento de Dados**
   - Busca saldos contábeis por período
   - Aplica regras de classificação
   - Calcula totais e subtotais

4. **Formatação do Relatório**
   - Composição hierárquica das contas
   - Aplicação de sinais (débito/crédito)
   - Formatação de valores e textos

5. **Geração de Saída**
   - PDF com relatório formatado
   - Numeração de páginas
   - Assinaturas e termos

## ⚠️ Regras de Negócio

### Validações
- Estrutura de plano de contas com 6 graus
- Grau da empresa deve ser o 5º
- Datas de referência não podem ser iguais
- Saldos devem estar atualizados

### Classificação de Contas
- **Ativo:** Contas classe 1
- **Passivo:** Contas classe 2  
- **Resultado:** Contas classe 3
- **Sinais invertidos** para passivo e resultado

### Agrupamentos Específicos
- Contas 31, 32, 33, 36 possuem agrupamentos customizados
- Mapeamento para códigos específicos (990101xx)

### Termos e Assinaturas
- Termos em extenso para valores
- Assinaturas do responsável e contador
- Qualificações profissionais

## 🎨 Saídas Disponíveis

| Formato | Descrição | Método |
|---------|-----------|---------|
| PDF | Relatório contábil completo | `gerarPDF()` |

## 🔧 Dependências

**Bibliotecas:**
- `jasperreports` - Geração de relatórios
- `multitec.utils` - Utilitários e cálculos
- `java.time` - Manipulação de datas

**Parâmetros do Sistema:**
- `GRAUEMPRESA` - Grau da empresa no plano
- `ESTRCODCONTA` - Estrutura do código de conta
- `EB_ATUALIZARCTAS` - Indicador de atualização

## 📝 Observações Técnicas

- Suporte a múltiplos períodos comparativos
- Hierarquia automática baseada em graus
- Tratamento específico para DRE mensal
- Integração com livro diário
- Formatação profissional para relatórios contábeis
- Validações robustas de consistência
- Suporte a notas explicativas periodizadas