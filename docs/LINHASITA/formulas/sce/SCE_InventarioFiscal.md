# SCE_InventarioFiscal

## 📖 Descrição
Fórmula responsável pelo cálculo e atualização de preços unitários e totais de itens em inventário fiscal, considerando regras de custo, histórico de lançamentos e configuração fiscal do item. Atualiza campos JSON e saldo contábil para correta escrituração e acompanhamento de inventário.

## 🎯 Finalidade
- Calcular preço unitário e total de itens no inventário fiscal.
- Atualizar campos JSON de controle interno e fiscal do item.
- Validar configuração fiscal e grupo de inventário.
- Obter saldo de contas e inventários anteriores.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Desenvolvedores de sistemas de gestão de inventário

## ⚙️ Parâmetros/Configurações

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| bcb11 | Bcb11 | Sim | Item do inventário a ser processado |
| dtIni | LocalDate | Não | Data inicial do período de consulta |
| dtFin | LocalDate | Não | Data final do período de consulta |

## 📊 Estrutura de Processamento

### Inicialização
- Recupera empresa ativa (`Aac10`) e item (`Abm01`).
- Carrega configurações do item (`Abm0101`) e campos JSON (`jsonAbm0101`).
- Define período do inventário baseado na data do lançamento (`bcb10data`).

### Processamento de Itens
- Recupera configuração fiscal (`Abm12`) e valida presença.
- Calcula taxa do grupo de inventário do item.
- Obtém preço unitário via regras de custo ou inventários anteriores.
- Atualiza `bcb11unit` e `bcb11total`.
- Atualiza JSON do item (`vlr_inventario`) e saldo da conta (`inv_est_sdo_cta`).

### Cálculo de Preço Unitário
- Baseado em `campoGI` do item:
    - Preço médio, custo de aquisição, maior/menor custo, último preço, último unitário de saída.
    - Busca histórico em inventários anteriores se necessário.
- Ajusta pelo percentual do grupo de inventário (`taxa`).

### Saldo Contábil
- Obtém saldo do grupo de inventário (`buscarSaldoDoGrupo`).
- Caso saldo zero, busca saldo da conta (`buscarSaldoConta`).

## ⚠️ Regras de Negócio
- Itens sem configuração fiscal interrompem o processamento.
- Tipos fiscais 7, 8, 9 e 99 ignoram validação de grupo de inventário.
- Preços unitários nulos são substituídos por zero.
- Saldo de grupo e contas é agregado via JSON.
- Taxa de inventário aplica-se apenas se `taxa > 0`.

## 🎨 Saídas/Retornos

| Tipo | Descrição | Formato |
|------|-----------|---------|
| Item atualizado | Objeto `Bcb11` com preço unitário, total e JSON ajustado | Objeto Java (JSON interno) |

## 🔧 Dependências

**Bibliotecas:**
- `sam.server.samdev.formula.FormulaBase` - Base da fórmula
- `sam.dicdados.FormulaTipo` - Tipo da fórmula
- `br.com.multitec.utils.collections.TableMap` - Estrutura para campos JSON
- `java.time.LocalDate` e `DateTimeFormatter` - Manipulação de datas

**Entidades do Sistema:**
- `Bcb11`, `Bcb10`, `Abm01`, `Abm0101`, `Abm12`, `Abm11`, `Abm40`
- `Bcc01`, `Bcc02`, `Ebb02`, `Aac10`, `Aag02`

## 📝 Observações Técnicas
- Campos JSON acessados dinamicamente via `TableMap`.
- Consulta de histórico via critérios e SQL direto.
- Cálculo de preço unitário considera diversas regras de custo.
- Saldo de contas e inventário anterior é usado como fallback.
- Atualização do item persiste alterações e garante commit da sessão.