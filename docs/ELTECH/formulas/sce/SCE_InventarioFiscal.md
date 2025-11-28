# SCE - Inventário Fiscal

## 📖 Descrição
Classe que calcula valores de inventário fiscal de itens, determinando preço unitário, total e saldo contábil. Busca histórico de inventários anteriores, aplica taxas de grupo de inventário e compõe campos JSON para integração com o módulo de contabilidade.

## 🎯 Finalidade
Garantir cálculo correto dos valores de inventário fiscal, incluindo:
- Preço unitário e total do item
- Saldo contábil do grupo
- Aplicação de regras fiscais específicas por tipo de item
- Integração com inventários anteriores e contas contábeis

## 👥 Público-Alvo
- Controladoria
- Departamento Fiscal
- Auditoria Interna
- Gestão de Estoque
- Equipes de TI responsáveis por regras de negócio contábil

## ⚙️ Parâmetros e Campos Principais

| Campo / Parâmetro | Tipo | Descrição |
|------------------|------|-----------|
| bcb11 | Bcb11 | Item do inventário sendo processado |
| dtIni | LocalDate | Data inicial do período de cálculo |
| dtFin | LocalDate | Data final do período de cálculo |
| aac10 | Aac10 | Empresa ativa |
| abm01 | Abm01 | Item cadastrado no sistema |
| abm0101 | Abm0101 | Configuração do item por empresa |
| abm12 | Abm12 | Configuração fiscal do item |
| preco_livre | BigDecimal | Preço livre definido na configuração do item |
| bcb11.bcb11unit | BigDecimal | Preço unitário calculado |
| bcb11.bcb11total | BigDecimal | Preço total calculado |
| bcb11.bcb11json | TableMap | Campos adicionais em JSON para integração contábil |

## 🔄 Fluxo do Processo
1. Carrega item (`Bcb11`) e datas de referência (`dtIni`, `dtFin`).
2. Busca empresa ativa (`Aac10`) e dados do item (`Abm01`, `Abm0101`, `Abm12`).
3. Valida existência de configuração fiscal; interrompe execução se inexistente.
4. Obtém taxa do grupo de inventário do item (`obterTaxaGrupoInventarioItem`).
5. Calcula preço unitário:
	- Aplica taxa se houver
	- Busca histórico de inventários anteriores se unitário = 0
6. Calcula total (`bcb11total`) e arredonda para 2 casas decimais.
7. Atualiza JSON do item (`bcb11json`) com saldos contábeis.
8. Busca saldo do grupo ou saldo da conta, se necessário (`buscarSaldoDoGrupo`, `buscarSaldoConta`).

## ⚠️ Regras de Negócio
- Tipo fiscal do item determina regras de cálculo (exceto 7, 8, 9, 99).
- Itens próprios e de terceiros têm validações de grupo de inventário.
- Preço unitário pode ser: preço médio, último unitário de saída ou maior preço unitário.
- Histórico de inventário anterior é usado quando não há valor calculado.
- Campos JSON atualizam saldos contábeis do item para o inventário fiscal.

## 🎨 Saídas Disponíveis
| Saída | Descrição |
|-------|-----------|
| Campos atualizados em `Bcb11` | Inclui `unit`, `total` e `json` para integração contábil |

## 🔧 Dependências
- Entidades: `Aac10`, `Abm01`, `Abm0101`, `Abm11`, `Abm12`, `Abm40`, `Bcb10`, `Bcb11`, `Bcc01`, `Bcc02`, `Ebb02`
- Framework: `FormulaBase` (classe base)
- Utilitários: `TableMap`, `Criterions`, `Fields`, `Joins`, `Parametro`, `SCE5520Service`, `SCEService`
- Manipulação de datas: `LocalDate`, `DateTimeFormatter`
- Sessão de banco de dados via `getSession()` e `getAcessoAoBanco()`
- Métodos de arredondamento e validação fiscal integrados ao sistema

## 📌 Métodos Principais
| Método | Descrição |
|--------|-----------|
| `executar()` | Método principal que calcula unitário, total e atualiza JSON do inventário |
| `buscarUnitarioLancamento(Long, LocalDate)` | Retorna preço unitário baseado em histórico e configuração do item |
| `buscarSaldoConta(LocalDate, Abm40)` | Retorna saldo contábil de uma conta na data do inventário |
| `buscarSaldoDoGrupo(Long)` | Retorna saldo do grupo de inventário em JSON |
| `obterTaxaGrupoInventarioItem(Long, Integer)` | Retorna taxa de ajuste do grupo de inventário do item |
| `obterCampoGrupoInventarioItem(Long)` | Retorna tipo de cálculo do grupo de inventário |
| `ultimnoUnitatioSaida(Long)` | Retorna último preço unitário de saída do item |
| `maiorPrecoUnitario(Long)` | Retorna maior preço unitário de saída do item |
| `precoMedioUnit(Long)` | Retorna preço médio unitário baseado no histórico |
| `buscarItemInventarioAnterior(Long)` | Busca preço unitário de inventários anteriores |
| `obterTipoFormula()` | Retorna tipo da fórmula: `FormulaTipo.INVENTARIO` |
