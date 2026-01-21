# SRF_Abm0101

## 📖 Descrição
Interceptor ORM responsável por atualizar automaticamente campos livres em formato JSON do cadastro `Abm0101`, calculando o valor unitário com acréscimo percentual durante o processo de persistência, de acordo com a empresa ativa no sistema SRF.

## 🎯 Finalidade
Garantir que o campo livre `vlr_unit` seja corretamente calculado e atualizado antes da gravação do registro, aplicando um reajuste de 3% com base no valor da última compra ou no preço médio unitário informado.

## 👥 Público-Alvo
- Comercial
- Controladoria
- TI / Desenvolvimento
- Administração de Preços

## ⚙️ Configuração
**Recursos Necessários:**
- Interceptor ORM `SRF_Abm0101`

**Momento de Execução:**
- Evento `prePersist` do ORM

## 📊 Dados e Fontes
**Entidades Principais:**
- `Abm0101` – Cadastro base interceptado
- `Aac10` – Empresa ativa no contexto da sessão

**Campos Utilizados:**
- `abm0101pmu` – Preço médio unitário
- `abm0101json` – Campos livres em formato JSON

**Campos JSON Manipulados:**
- `ultima_compra`
- `vlr_unit`

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|----------|------|-------------|----------|
| abm0101 | Abm0101 | Sim | Entidade que será persistida |
| session | Session | Sim | Sessão ativa do ORM |

## 📋 Saídas do Processo

| Campo | Descrição | Tipo |
|------|-----------|------|
| abm0101json.vlr_unit | Valor unitário calculado com acréscimo de 3% | BigDecimal |
| abm0101 | Entidade atualizada antes da persistência | Abm0101 |

## 🔄 Fluxo do Processo

1. **Identificação da Empresa Ativa**
    - Obtém a empresa ativa (`Aac10`) através do `InterceptadorUtils`

2. **Validação da Empresa**
    - Executa o processamento apenas se `aac10id == 1` (LCB)

3. **Validação de Campo Carregado**
    - Verifica se o campo `abm0101pmu` está carregado
    - Garante que o valor não seja nulo

4. **Tratamento de JSON**
    - Inicializa `abm0101json` caso não exista
    - Verifica a existência do campo `ultima_compra`

5. **Cálculo do Valor Unitário**
    - Se existir `ultima_compra` válida:
        - Aplica acréscimo de 3% sobre `ultima_compra`
    - Caso contrário:
        - Aplica acréscimo de 3% sobre `abm0101pmu`

6. **Atualização do Registro**
    - Atualiza o campo JSON `vlr_unit`
    - Persiste a entidade normalmente

## ⚠️ Regras de Negócio

### Regras Gerais
- O interceptor executa apenas no evento de **pré-gravação**
- O cálculo ocorre somente para a empresa **LCB (aac10id = 1)**

### Regras de Cálculo
- Percentual fixo de acréscimo: **3%**
- Prioridade de cálculo:
    1. Campo JSON `ultima_compra`
    2. Campo `abm0101pmu`

### Condições de Execução
- O campo `abm0101pmu` deve estar carregado
- O valor não pode ser nulo
- O JSON é criado automaticamente se inexistente

## 🎨 Campos Livres Atualizados

| Campo JSON | Descrição | Origem do Cálculo |
|----------|-----------|------------------|
| vlr_unit | Valor unitário com acréscimo | `ultima_compra` ou `abm0101pmu` |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Interceptação ORM
- `multitec.utils` – Utilitários gerais
- `multitec.utils.collections` – Manipulação de `TableMap`

**Utilitários Internos:**
- `InterceptadorUtils` – Recuperação da empresa ativa

## 📝 Observações Técnicas

- Interceptor não implementa lógica de exclusão (`preDelete`)
- Código tolerante a JSON nulo
- Acréscimo percentual fixo (hardcoded em 3%)
- Atualização ocorre sem bloquear o processo de persistência
- Estrutura preparada para múltiplas empresas, porém restrita à LCB
- Uso de `TableMap` para flexibilidade de campos livres
- Não há validação de arredondamento ou escala numérica
