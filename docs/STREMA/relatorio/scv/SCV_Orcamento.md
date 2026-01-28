# SCV_Orcamento.md

## 📖 Descrição
Script de relatório para emissão de orçamentos e propostas comerciais no ERP Strema. Ele realiza a junção de dados complexos entre o cabeçalho do orçamento (`CBE10`), dados da central de documentos (`ABB01`), informações da empresa e detalhes tributários extraídos de campos JSON.

## 🎯 Finalidade
Gerar o documento oficial de proposta comercial para o cliente, permitindo a impressão física ou o envio digital direto pelo ERP, garantindo que o cliente receba os valores, prazos e condições de pagamento de forma padronizada e profissional.

## 👥 Público-Alvo
* Vendedores e Representantes Comerciais.
* Assistentes de Vendas.
* Clientes finais (recebimento da proposta).

## ⚙️ Configuração
* **Recursos Necessários:** * Classe `SCV_Orcamento`
   * Template IReport: `SCV_Orcamento.jasper`
   * Imagens de Logo e marca d'água de "Cancelado" no diretório de resources.
* **Localização:** `strema.relatorios.scv`
* **Tipo de Tarefa:** Relatório (`SCV - Orçamento - Strema`)

## 📊 Dados e Fontes
### Tabelas Principais:
* **CBE10 / CBE1001** - Cabeçalho e itens do orçamento.
* **ABB01** - Central de documentos (gerencia numeração e data).
* **ABE01 / ABE0101** - Cadastro de clientes e endereços principais (entrega/faturamento).
* **AAC10** - Dados da empresa ativa (emitente da proposta).
* **AAB10 / AAB1008** - Dados do usuário logado e sua configuração de e-mail/assinatura.
* **ABE0104** - Contatos da entidade (utilizado para buscar e-mails de destino).

## ⚙️ Parâmetros de Filtro (Tela)
| Parâmetro | Tipo | Descrição |
| :--- | :--- | :--- |
| numeroInicial / numeroFinal | Integer | Intervalo de números de orçamento para geração em lote. |
| entidades | List<Long> | Filtro por clientes específicos. |
| emissao | Date Range | Intervalo de datas de criação do documento. |
| status | Boolean | Filtros booleanos para Criado, Concluído ou Cancelado. |
| impressao | Integer | 0 para PDF (visualização), 1 para XLSX (análise de dados). |
| enviaEmail | Boolean | Se ativado, dispara o PDF automaticamente para os contatos do cliente. |

## 🔄 Fluxo do Processo
1.  **Valores Iniciais:** O sistema pré-seleciona orçamentos com status "Criado" e "Concluído".
2.  **Carga de Parâmetros:** Identifica os caminhos físicos das logos e carrega os dados da empresa (`comporDadosEmpresa`) para popular o cabeçalho do relatório.
3.  **Execução da Query:** Busca os dados via SQL, realizando **casts em campos JSON** para extrair valores dinâmicos como alíquotas de IPI, ICMS e valores de ST calculados previamente.
4.  **Lógica de E-mail:** * Se `enviaEmail` for verdadeiro, gera o PDF em memória.
   * Identifica o e-mail do usuário logado (`AAB1008`) para configurar o servidor de saída (SMTP).
   * Busca contatos do cliente com classificação comercial (código "9001").
   * Dispara o e-mail com a proposta em anexo, normalizando o corpo do texto para compatibilidade ASCII.
5.  **Saída:** Retorna o arquivo (PDF ou Excel) para o navegador do usuário.

## ⚠️ Regras de Negócio
### Extração de Dados JSON
Diferente de relatórios convencionais, este script acessa a "memória de cálculo" guardada nos JSONs do banco:
* `cbe1001json ->> 'ipi_aliq'`: Alíquota de IPI individual por item.
* `cbe10json ->> 'st_icm'`: Valor consolidado de Substituição Tributária.
* `cbe10json ->> 'prazo_entrega'`: Informação logística customizada no documento.

### Automação de E-mail
* **Remetente:** O usuário logado deve obrigatoriamente ter uma conta de e-mail e assinatura configuradas na tabela `AAB1008`.
* **Segurança:** O corpo da mensagem passa por um processo de normalização (`Normalizer.normalize`) para garantir que caracteres acentuados não causem erro no envio.

## 🔧 Dependências Técnicas
* **Bibliotecas:** JasperReports (Engine de impressão), JavaMail (Protocolo SMTP).
* **Serviços:** `CAS1010Service` (Gerenciador de assinaturas digitais de e-mail).