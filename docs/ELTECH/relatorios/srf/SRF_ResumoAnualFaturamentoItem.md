# SRF – Resumo Anual Faturamento por Item

## 📖 Descrição
Relatório anual consolidado de faturamento por item, mostrando valores brutos e líquidos por mês, com tratamento de devoluções, agrupamento de itens por código e filtros corporativos.

## 🎯 Finalidade
Fornecer visão anual do faturamento bruto e líquido por item, mês a mês, permitindo à gestão analisar evolução, sazonalidade e impacto de devoluções.

## 👥 Público-Alvo
- Controladoria
- Departamento Fiscal
- Gestão Comercial
- Diretoria / Gerência Administrativa

## ⚙️ Parâmetros do Relatório

| Parâmetro | Tipo | Obrigatório | Descrição | Valores Possíveis |
|-----------|------|-------------|-----------|-----------------|
| faturamento | Integer | Não | Tipo de operação | 0 / 1 |
| valorRelatorio | Integer | Sim | Soma quantidade ou valor | 0=Quantidade, 1=Total documento |
| anoRef | String | Sim | Ano de referência | AAAA |
| dtIni | String | Automático | Mês/ano inicial | MM/yyyy |
| dtFim | String | Automático | Mês/ano final | MM/yyyy |
| impressao | Integer | Sim | Formato de saída | 0=PDF, 1=XLSX |
| itens | List<Long> | Não | IDs de itens | Lista de IDs |
| tipoDoc | List<Long> | Não | Tipos de documento | Lista de IDs |
| numero | Integer | Não | Número de documento | Inteiro |
| pcd | List<Long> | Não | Critérios PCD | Lista |
| representantes | List<Long> | Não | Representantes vinculados | Lista |
| empresas | List<Long> | Não | Empresas | Lista |
| liquido | Boolean | Não | Apenas valor líquido | true/false |

## 📋 Campos do Relatório

| Campo | Descrição | Tipo |
|-------|-----------|------|
| abm01codigo | Código do item | String |
| abm01na | Nome do item | String |
| Janeiro…Dezembro | Valor bruto e líquido do mês | BigDecimal |
| total | Soma anual | BigDecimal |

## 🔄 Fluxo do Processo
1. Carrega parâmetros e ano de referência.
2. Processa agrupamentos de itens.
3. Busca faturamento bruto ou líquido conforme parâmetro.
4. Aplica tratamento de devoluções.
5. Consolida valores por mês e item.
6. Gera saída em PDF ou XLSX.

## ⚠️ Regras de Negócio
- Agrupamento por códigos de 2 caracteres.
- CFOPs especiais tratados nas somas.
- Devoluções subtraídas do total bruto.

## 🎨 Saídas Disponíveis
| Formato | Descrição |
|---------|-----------|
| PDF | Relatório para impressão |
| XLSX | Planilha analítica |

## 🔧 Dependências
- `multitec.utils` — Datas e utilitários
- `TableMap` — Estrutura de dados
- `jasperreports` — Engine de relatórios
