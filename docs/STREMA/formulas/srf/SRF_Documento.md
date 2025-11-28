# SRF_Documento

## 📖 Descrição
Fórmula responsável pelo processamento e cálculo de informações fiscais de documentos, incluindo preenchimento de observações fiscais, contribuintes, comissões e campos adicionais. Integra dados de entidades, itens, operações comerciais e regras específicas de regime tributário, visando a correta escrituração de documentos para fins fiscais e contábeis.

## 🎯 Finalidade
- Configurar observações fiscais e internas do documento.
- Calcular taxas de comissão de representantes e da entidade.
- Processar campos JSON de visualização e controle fiscal.
- Validar dados obrigatórios e regras de negócio específicas, como regime especial de tributação e CSTs.

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Desenvolvedores de sistemas de EFD e gestão fiscal

## ⚙️ Parâmetros/Configurações

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| eaa01 | Eaa01 | Sim | Documento fiscal principal a ser processado |
| procInvoc | String | Não | Indicador do processo de invocação |

## 📊 Estrutura de Processamento

### Inicialização
- Recupera documento (`eaa01`) e dados relacionados (entidade, endereço principal, operação comercial, parâmetros de cálculo).
- Inicializa campos JSON (`jsonAbe01`, `jsonEaa01`) e objetos auxiliares.

### Processamento de Itens
- Itera sobre os itens do documento (`eaa0103s`) e:
    - Recupera dados do item (`Abm01`, `Abm0101`, `Abm10`, `Abm1001`).
    - Preenche campos JSON por estado e empresa.
    - Atualiza observações fiscais quando aplicável (ex.: CST 20).

### Regras Fiscais
- Valida regime especial (`simples nacional`) e ajusta observações fiscais.
- Calcula campos adicionais de fidelidade ou crédito, dependendo do PCD.
- Determina se a operação é dentro ou fora do estado (`ufEnt`).

### Taxas de Comissão
- Obtém taxas da entidade, tabela de preço e representantes.
- Ajusta valores nulos para zero e grava no documento.

### Composição de Observações
- `comporObservacoesDocumento()`: monta observações internas, fiscais e de contribuintes.
- Substitui placeholders (`&1`, `&2`, `&3`, `&4`) pelos valores reais de impostos ou códigos.
- `comporObsContribuinteComChaveNFeDocumentosReferenciados()`: inclui informações de documentos referenciados com número, data e chave de acesso.

### Validações e Controles
- Verifica existência de endereço principal da entidade.
- Assegura que campos obrigatórios estejam preenchidos.
- Atualiza campos JSON e observações conforme regras fiscais.

## ⚠️ Regras de Negócio

### Observações Fiscais
- Observações podem ser preenchidas por entidade, PCD ou tabela de preço.
- Campos como `eaa01obsFisco` e `eaa01obsContrib` são concatenados e formatados.
- CSTs específicos podem gerar observações adicionais.

### Comissões
- Taxas podem vir de:
    - Entidade (`Abe01`, `Abe02`)
    - Tabela de preço (`Abe40`)
    - Representantes do documento
- Nulos ou zero são substituídos pelos valores disponíveis.

### Cálculos Adicionais
- Fidelidade: 1% do total do documento para PCD 10001.
- Crédito: igual ao total do documento para PCD 11100.
- Valores arredondados a duas casas decimais.

### Regime Especial
- Para Simples Nacional, adiciona observações fiscais com alíquota e valor do ICMS.

### Documentos Referenciados
- Inclui informações de documentos referenciados (número, data, chave NFe) nas observações do contribuinte.

## 🎨 Saídas/Retornos

| Tipo | Descrição | Formato |
|------|-----------|---------|
| Documento atualizado | Objeto `Eaa01` com campos preenchidos e observações ajustadas | Objeto Java (JSON interno) |

## 🔧 Dependências

**Bibliotecas:**
- `sam.server.samdev.formula.FormulaBase` - Base da fórmula
- `sam.dicdados.FormulaTipo` - Tipo de fórmula
- `br.com.multitec.utils.ValidacaoException` - Tratamento de exceções
- `br.com.multitec.utils.collections.TableMap` - Estrutura para campos JSON
- `java.time.format.DateTimeFormatter` - Formatação de datas

**Entidades do Sistema:**
- `Eaa01`, `Eaa0101`, `Eaa0103`, `Eaa01033` - Documentos e itens
- `Abb01`, `Abb10` - Operação comercial
- `Abe01`, `Abe40` - Entidade/cliente
- `Abm01`, `Abm0101`, `Abm10`, `Abm1001` - Itens e tabelas de preço
- `Aaj10` - CST ICMS
- `Aac10`, `Aac13` - Empresa e dados fiscais
- `Aag02`, `Aag0201` - UF e município

## 📝 Observações Técnicas

- Processamento baseado em streams para manipulação de listas.
- Campos JSON acessados dinamicamente via `TableMap`.
- Substituição de placeholders em observações é condicional conforme tipo de PCD.
- Performance otimizada com consultas específicas para entidade, representantes e itens.
- Tratamento de nulos e inicialização de campos críticos.