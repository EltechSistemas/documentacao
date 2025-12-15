# SGT_Leiaute18_2024_ICMS_IPI.md

## 📖 Descrição
Fórmula responsável pela geração da Escrituração Fiscal Digital (EFD) – ICMS/IPI conforme Leiaute 18/2024. Consolida operações tributárias, movimentações de estoque, apurações fiscais e todas as informações exigidas pela legislação para entrega ao fisco.

## 🎯 Finalidade
Gerar arquivo digital no formato EFD (Blocos 0 a 9) contemplando:
- Registros de documentos fiscais de entrada e saída
- Apuração de ICMS, IPI, ICMS-ST, Diferencial de Alíquota e CIAP
- Controle de produção e estoque
- Inventário físico
- Cadastro de itens, entidades, unidades de medida e tabelas auxiliares

## 👥 Público-Alvo
- Departamento Fiscal
- Contabilidade
- Auditoria Fiscal
- Controladoria

## ⚙️ Configuração
**Recursos Necessários:**
- Fórmula `SGT_Leiaute18_2024_ICMS_IPI`
- Banco de dados com tabelas AA, AB, EA, BC, EC, ED

**Localização:** `linhasita/formulas/sgt/`

## 📊 Dados e Fontes
**Tabelas Principais:**
- `EAA01` – Documentos fiscais
- `ABB01` – Centrais de documentos
- `ABM01` – Cadastro de itens
- `AAC10` – Empresa
- `EDB01` – Apurações fiscais
- `BCC01` – Movimentações de estoque
- `ECC01` – Fichas CIAP
- `BCB11` – Itens de inventário

**Entidades Envolvidas:**
- `Aac10` – Empresa
- `Eaa01` – Documento fiscal
- `Abb01` – Central de documento
- `Abm01` – Item
- `Edb01` – Apuração fiscal
- `Ecc01` – Ficha CIAP
- `Bcc01` – Movimentação de estoque

## ⚙️ Parâmetros do Processo

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| dtInicial | LocalDate | Sim | Data inicial do período de apuração |
| dtFinal | LocalDate | Sim | Data final do período de apuração |
| arqSubstituto | Integer | Não | Indicador de arquivo substituto |
| dtInventario | LocalDate | Não | Data do inventário físico |

## 🔄 Fluxo do Processo

1. **Inicialização e Validações**
   - Obtém dados da empresa (AAC10) e configurações fiscais
   - Valida existência de município, endereço e informações fiscais
   - Define período de apuração (mês/ano)

2. **Geração dos Blocos da EFD**
   - **Bloco 0**: Abertura, identificação e referências
   - **Bloco B**: Escrituração e apuração do ISS
   - **Bloco C**: Documentos fiscais I – Mercadorias (ICMS/IPI)
   - **Bloco D**: Documentos fiscais II – Serviços (ICMS)
   - **Bloco E**: Apuração do ICMS e do IPI
   - **Bloco G**: CIAP – Controle do Ativo Imobilizado
   - **Bloco H**: Inventário Físico
   - **Bloco K**: Controle da Produção e Estoque
   - **Bloco 1**: Outras Informações

3. **Processamento por Bloco**
   - Carregamento de documentos por modelo (01, 1B, 04, 55, etc.)
   - Cálculos de totais, bases e impostos
   - Consolidação por CFOP/CST
   - Geração de registros analíticos e complementares

4. **Encerramento**
   - Geração dos registros 9900 (controle de linhas)
   - Bloco 9 – Controle e encerramento do arquivo digital
   - Cálculo do total de linhas do arquivo

## ⚠️ Regras de Negócio

### Validações Críticas
- Empresa deve ter município e endereço cadastrados
- Informações fiscais (perfil, tipo de atividade) são obrigatórias
- Documentos devem ter situação fiscal definida
- Apurações de ICMS e IPI devem existir para o período

### Perfis de Apuração
- **Perfil A**: Indústria e equiparado
- **Perfil B**: Comércio
- **Perfil C**: Prestador de serviços
- Geração de registros varia conforme perfil e operação (entrada/saída)

### Tratamentos Específicos
- **NFe Própria**: Registros específicos no Bloco E
- **Substituição Tributária**: Bloco E200–E250
- **Diferencial de Alíquota**: Bloco E300–E316
- **CIAP**: Bloco G com movimentações de bens
- **Inventário**: Bloco H com valoração e classificação

### Controle de Linhas
- Contagem individual por registro
- Totalização por bloco
- Registro 9900 com quantitativo de cada tipo

## 🎨 Saídas Geradas

| Saída | Descrição | Tipo |
|-------|-----------|------|
| dadosArquivo | Conteúdo completo da EFD formatado | TextFile |
| Bloco 0–9 | Todos os registros da escrituração | Texto delimitado por "|" |

## 🔧 Dependências

**Bibliotecas:**
- `multiorm` – Persistência e consultas
- `sam.dicdados` – Definições de tipos de fórmula
- `sam.model.entities` – Entidades do sistema
- `br.com.multitec.utils` – Utilitários diversos

**Entidades Relacionadas:**
- Todas entidades dos pacotes `sam.model.entities.aa`, `ab`, `ea`, `bc`, `ec`, `ed`

## 📝 Observações Técnicas

### Processamento
- **Execução**: Síncrona, com processamento paginado de documentos
- **Memória**: Uso de `TableMap` e coleções para agrupamento
- **Performance**: Consultas otimizadas com paginação e filtros por período

### Estrutura do Arquivo
- **Delimitador**: Pipe ("|")
- **Codificação**: UTF-8
- **Formato**: Texto fixo conforme leiaute oficial

### Campos Livres (JSON) Utilizados
- `eaa01json`: Valores calculados do documento
- `eaa0103json`: Valores calculados do item
- `edb01json`: Valores de apuração
- `ecc0101json`: Valores do CIAP
- `aac10municipio.aag0201json`: Alíquotas internas

### Tratamentos Especiais
- **Entidades com múltiplas IE**: Geração de código único (codigo + IE)
- **Unidades de medida**: Registro 0190 automático
- **Fatores de conversão**: Registro 0220 conforme configuração do item
- **Itens correlatos**: Registro 0221 para mercadorias para revenda

### Versionamento
- **Leiaute**: Versão 18
- **Alinhamentos**: 0050 (EFD), 0030 (ICMS), 0033 (ICMS-ST), 0032 (IPI), 0031 (DIFAL)
- **Compatibilidade**: Modelos fiscais atualizados conforme legislação vigente

### Tratamento de Erros
- `ValidacaoException` para dados inconsistentes
- Verificação de cancelamento do processo
- Logs de status durante a execução

---

**Última Alteração**: 09/12/2025 às 08:20  
**Autor**: Bruno  
**Versão**: Leiaute 18/2024  
**Classe**: `linhasita.formulas.sgt.SGT_Leiaute18_2024_ICMS_IPI`