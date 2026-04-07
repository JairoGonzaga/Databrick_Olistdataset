# Databricks Olist Dataset - Pipeline de Dados

Um pipeline de ingestão e transformação de dados do dataset **Olist** em produção no Databricks, utilizando a arquitetura **Medallion** (Bronze → Silver → Gold) com integração de dados de cotação cambial.

---

## 📋 Visão Geral do Projeto

Este projeto implementa um pipeline completo de ETL (Extract, Transform, Load) para o dataset público de e-commerce do **Olist**, integrando:

- **9 arquivos CSV** do dataset Olist
- **Cotação do dólar** via API do Banco Central do Brasil
- **Arquitetura em camadas** (Medallion Architecture) para governança de dados
- **Orchestre automatizado** via Databricks Jobs com cronograma diário

---

## 🏗️ Arquitetura

### Medalha de Dados (Medallion Architecture)

```
┌─────────────────────────────────────────────────────────┐
│  LANDING LAYER                                          │
│  (Arquivos CSV brutos - Volumes Unity Catalog)          │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  BRONZE LAYER                                           │
│  (Dados brutos em Delta Lake + metadados)               │
│  • tb_customers                                         │
│  • tb_orders                                            │
│  • tb_order_items                                       │
│  • tb_order_payments                                    │
│  • tb_order_reviews                                     │
│  • tb_products                                          │
│  • tb_sellers                                           │
│  • tb_geolocalizacao                                    │
│  • tb_product_category_name_translation                │
│  • tb_cotacao_dolar                                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  SILVER LAYER                                           │
│  (Dados transformados e limpos)                         │
│  • Validações e limpeza de dados                        │
│  • Normalização de schemas                              │
│  • Joins e enriquecimento                               │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  GOLD LAYER                                             │
│  (Dados agregados e prontos para BI)                    │
│  • Dashboards                                           │
│  • Relatórios                                           │
│  • Analytics                                            │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 Estrutura de Arquivos

```
Databrick_Olistdataset/
├── README.md                          # Este arquivo
├── pipeline_rocketlab.yml             # Configuração de jobs e cronograma
├── Land_To_Bronze_Notebook.ipynb      # 1º etapa: Ingestão de dados brutos
├── Bronze_To_Silver.ipynb             # 2º etapa: Transformação e limpeza
├── Silver_To_Gold.ipynb               # 3º etapa: Agregações e análises
└── .git/                              # Controle de versão
```

---

## 📊 Descrição dos Notebooks

### 1️⃣ **Land_To_Bronze_Notebook.ipynb** (Ingestão)
Realiza a ingestão inicial dos dados do dataset Olist:
- Leitura de 9 arquivos CSV do Volume Landing
- Consumo da API do Banco Central do Brasil para cotação do dólar
- Criação de tabelas Delta Lake na camada Bronze
- Adição de timestamp de ingestão para rastreabilidade e auditoria

**Catálogo**: `visagio`  
**Schema**: `bronze`  
**Tempo de execução**: ~2-5 minutos

### 2️⃣ **Bronze_To_Silver.ipynb** (Transformação)
Transforma e enriquece os dados da camada Bronze:
- Validações e limpeza de dados
- Normalização de schemas
- Tratamento de valores nulos
- Joins entre tabelas relacionadas
- Cálculos derivados

**Entrada**: Tabelas Bronze  
**Saída**: Tabelas Silver  
**Tempo de execução**: ~3-8 minutos

### 3️⃣ **Silver_To_Gold.ipynb** (Análise)
Cria views agregadas prontas para análise e BI:
- Agregações por períodos
- Métricas de desempenho
- Tabelas fato e dimensão
- Dados prontos para dashboards

**Entrada**: Tabelas Silver  
**Saída**: Tabelas Gold (Analytics)  
**Tempo de execução**: ~2-5 minutos

---

## ⏰ Cronograma de Execução

O pipeline é orquestrado automaticamente via **Databricks Jobs** com base na configuração em `pipeline_rocketlab.yml`:

| Tarefa | Horário | Frequência | Dependência |
|--------|---------|-----------|------------|
| Ingestão (Bronze) | 13:00 | Diária | - |
| Transformação (Silver) | 13:X | Após Bronze | Ingestão |
| Agregação (Gold) | 13:Y | Após Silver | Transformação |

**Fuso horário**: America/Sao_Paulo (Brasília)  
**Configuração**: `pipeline_rocketlab.yml`

---
