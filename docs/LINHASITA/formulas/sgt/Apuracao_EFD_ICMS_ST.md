# Apuração EFD ICMS ST - SGT

## 📖 Descrição
Fórmula para cálculo e apuração do ICMS-ST (Substituição Tributária) para a Escrituração Fiscal Digital (EFD), considerando operações de devolução, ressarcimento, retenção, ajustes, deduções e valores extra-apuração. A fórmula gera registros na tabela `Edb01` por estado (UF) para um determinado período (mês/ano).

## 🎯 Finalidade
Calcular automaticamente os valores de ICMS-ST a recolher, saldos credores, deduções e extra-apurações, conforme as regras da EFD, para cada estado envolvido nas operações da empresa.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Faturamento
- Auditoria Fiscal

## 📊 Dados e Fontes

**Tabelas Principais:**
- `Edb01` - Apurações fiscais
- `Eaa01` - Documentos fiscais
- `Eaa0103` - Itens do documento fiscal
- `Eaa01031` - Ajustes por item
- `Eaa01035` - Outros ajustes
- `Aag02` - Estados (UF)
- `Aac10` - Empresas
- `Abb01` - Centrais de negócio
- `Aah01` - Tipos de documento
- `Aaj15` - CFOP
- `Aaj03` - Situação do documento
- `Aaj16` - Códigos de ajuste (Eaa01031)
- `Aaj17` - Códigos de ajuste (Eaa01035)
- `Aaj28` - Tipos de apuração

## ⚙️ Parâmetros da Fórmula

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| edb01 | Edb01 | Sim | Registro de apuração base (contém mês, ano, tipo) |
| icmsST_efd | String | Não | Campo JSON que armazena o valor de ICMS-ST nos itens (Eaa0103) |

## 🔄 Fluxo do Processo

### 1. **Configuração Inicial**
- Validação do registro de apuração (`edb01`)
- Definição do período (data inicial e final do mês)
- Obtenção da UF da empresa
- Determinação dos estados a processar (um específico ou todos)

### 2. **Exclusão de Apurações Anteriores**
- Remoção de registros `Edb01` existentes para o mesmo período e tipo, evitando duplicidade.

### 3. **Cálculo dos Valores por Estado**
Para cada estado (exceto "EX" - Exterior):
- Busca saldo credor anterior (`credAnt`)
- Calcula devoluções (`devolucao`) e ressarcimentos (`ressarcimento`) de ICMS-ST
- Soma outros créditos (`outrosCred`) e ajustes a crédito (`ajustesCred`)
- Calcula retenção por ST (`retencao`)
- Soma outros débitos (`outrosDeb`) e ajustes a débito (`ajustesDeb`)
- Determina saldo devedor antes das deduções (`saldo`)
- Calcula deduções (`deducoes`)
- Calcula total de ICMS-ST a recolher (`saldoDevedor`)
- Calcula saldo credor a transportar (`saldoCredor`)
- Soma valores extra-apuração (`extra`)

### 4. **Persistência dos Resultados**
- Cria ou atualiza registro `Edb01` para o estado, armazenando todos os campos calculados no JSON.
- Persiste apenas se houver valores relevantes ou se for a UF da empresa.

## ⚠️ Regras de Negócio

### Escopo da Apuração
- Processa por estado (UF) individual ou todos de uma vez.
- Ignora operações com UF "EX" (Exterior).
- Considera apenas documentos com `eaa01iEfdIcms = 1` (incluído na EFD ICMS).

### Cálculo de Campos
1. **Saldo credor anterior**: Buscado da apuração do mês anterior para a mesma UF.
2. **Devolução e Ressarcimento**: CFOPs específicos para cada operação.
3. **Outros créditos/débitos**: Ajustes das tabelas `Eaa01031` e `Eaa01035` com códigos específicos.
4. **Retenção por ST**: CFOPs iniciados com "5" ou "6".
5. **Deduções**: Ajustes com código iniciado em "61".
6. **Extra-apuração**: Valores de documentos com situação diferente de "01" ou "07".

### Validações
- Apenas documentos com situação EFD diferente de "02", "03", "04", "05" são considerados.
- Campos JSON são utilizados para armazenar valores de ICMS-ST (`icmsST_efd`).

## 🔧 Métodos Principais

### `executar()`
Método principal que orquestra todo o processo de apuração.

### `buscarEstados()`
Retorna lista de todos os estados (UF) cadastrados.

### `buscarApuracaoAnterior(Long aag02id, Edb01 edb01)`
Busca a apuração do mês anterior para a UF especificada.

### `buscarDevolucoesRessarcimentosIcmsST(boolean isDevolucao, Long aag02id, String cpoVlr1, LocalDate dtInicial, LocalDate dtFinal)`
Calcula devoluções ou ressarcimentos de ICMS-ST conforme CFOPs específicos.

### `buscarOutrosCredEaa01035IcmsSt(...)`, `buscarOutrosCredIcmsST(...)`, `buscarAjustesCredEaa01031IcmsSt(...)`
Métodos para calcular outros créditos e ajustes a crédito.

### `buscarRetencaoIcmsSt(...)`
Calcula o valor de ICMS-ST retido nas operações com CFOP iniciado em "5" ou "6".

### `buscarOutrosDebEaa01035IcmsSt(...)`, `buscarAjustesDebEaa01031IcmsSt(...)`
Métodos para calcular outros débitos e ajustes a débito.

### `buscarDeducoesEaa01031IcmsSt(...)`, `buscarDeducoesEaa01035IcmsSt(...)`
Calculam as deduções de ICMS-ST.

### `buscarExtraApurC100IcmsSt(...)`, `buscarExtraApurEaa01031IcmsSt(...)`, `buscarExtraApurEaa01035IcmsSt(...)`
Calculam valores extra-apuração.

### `excluirApuracoes(Integer mes, Integer ano, Long aaj28id, Long aag02id)`
Remove apurações anteriores para o mesmo período, tipo e UF (se especificada).

## 📊 Estrutura de Saída

**Registro Edb01 (JSON):**
- `credAnt` - Saldo credor do período anterior
- `devolucao` - ICMS-ST de devoluções
- `ressarcimento` - ICMS-ST de ressarcimentos
- `outrosCred` - Outros créditos e estornos
- `ajustesCred` - Ajustes a crédito
- `retencao` - ICMS retido por ST
- `outrosDeb` - Outros débitos e estornos
- `ajustesDeb` - Ajustes a débito
- `saldo` - Saldo devedor antes das deduções
- `deducoes` - Deduções
- `saldoDevedor` - Total do ICMS-ST a recolher
- `saldoCredor` - Saldo credor a transportar
- `extra` - Valores extra-apuração

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` - Criteria e consultas ao banco
- `multitec.utils` - Utilitários de data e coleções
- `sam.dicdados` - Tipos de fórmula
- `sam.model` - Entidades do sistema
- `java.time` - Manipulação de datas

**Módulo:** SGT (Sistema de Gestão Tributária)

## 📝 Observações Técnicas

### Filtros de Período
- Utiliza `eaa01esData` ou `abb01data` conforme o tipo de movimento (`eaa01esMov`).
- Considera o mês completo (do dia 1 ao último dia).

### Campos Dinâmicos
- O campo `icmsST_efd` é configurável e deve conter o nome do campo JSON que armazena o ICMS-ST nos itens.

### Tratamento de Estados
- Processa apenas estados com operações relevantes ou a UF da empresa.
- Ignora "EX" (Exterior).

### Performance
- Utiliza exclusão prévia para evitar registros duplicados.
- Consultas SQL otimizadas com somatórios diretos no banco.

---

**Última Alteração:** 09/12/2025 às 08:20  
**Autor:** Bruno  
**Tipo:** Fórmula de Apuração (SGT)  
**Versão:** 1.0