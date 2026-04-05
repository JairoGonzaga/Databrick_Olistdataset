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

## 🚀 Como Usar

### Pré-requisitos
- Acesso ao Databricks Workspace (visagio)
- Permissões no Unity Catalog
- Volume Landing com arquivos CSV do Olist

### Executar Manualmente
1. Abra o Databricks Workspace
2. Navegue até os notebooks no caminho `/Workspace/Users/jaironeto4828@gmail.com/VisagioRocket_Eng.Dados/`
3. Execute sequencialmente:
   - `Land_To_Bronze_Notebook`
   - `Bronze_To_Silver`
   - `Silver_To_Gold`

### Executar via Jobs (Recomendado)
O pipeline já está configurado para execução automática diária:
```bash
# Valide a configuração do job
databricks jobs create --json @pipeline_rocketlab.yml
```

---

## 🔧 Configurações Principais

| Parâmetro | Valor | Descrição |
|-----------|-------|-----------|
| Catálogo | `visagio` | Catálogo Unity Catalog onde os dados são armazenados |
| Schema Bronze | `bronze` | Schema para tabelas brutas |
| Schema Silver | `silver` | Schema para tabelas transformadas |
| Schema Gold | `gold` | Schema para tabelas agregadas |
| Volume Landing | `/Volumes/visagio/landing/atividade_rocketlab/` | Origem dos arquivos CSV |
| Performance | PERFORMANCE_OPTIMIZED | Configuração de cluster para melhor throughput |

---

## 📦 Datasets Inclusos

### Dataset Olist (9 tabelas)
1. **customers** - Informações de clientes
2. **orders** - Dados de pedidos
3. **order_items** - Itens por pedido
4. **order_payments** - Métodos e valores de pagamento
5. **order_reviews** - Avaliações e comentários
6. **products** - Catálogo de produtos
7. **sellers** - Informações de vendedores
8. **geolocalizacao** - Dados geográficos
9. **product_category_name_translation** - Tradução de categorias

### Dados Complementares
10. **cotacao_dolar** - Cotação diária do dólar (API Banco Central)

---

## 📈 Principais Transformações

### Validações
- Remoção de registros duplicados
- Preenchimento de nulos com padrões
- Validação de formato de datas

### Enriquecimentos
- Adição de moeda local (conversão BRL/USD)
- Derivação de períodos (ano, mês, trimestre)
- Cálculos de métricas (ticket médio, conversão)

### Agregações (Gold)
- Vendas por período
- Receita por categoria
- Performance de sellers
- Análise de clientes

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Versão | Uso |
|-----------|--------|-----|
| Databricks | - | Plataforma de execução |
| Apache Spark | 3.x | Engine de processamento |
| Delta Lake | - | Formato de armazenamento |
| Python | 3.x | Linguagem de programação |
| PySpark | - | API Spark para Python |
| Unity Catalog | - | Governança de dados |

---

## 📝 Logs e Monitoramento

Os logs de execução podem ser consultados em:
- **Databricks UI**: Página de Jobs → Pipeline_RocketLab
- **Run History**: Histórico completo de execuções e tempos
- **Cluster Logs**: Detalhes técnicos por tarefa

---

## ⚠️ Troubleshooting

### Erro: "refusing to merge unrelated histories"
Se receber este erro ao fazer git pull:
```bash
git pull --allow-unrelated-histories origin main
```

### Erro: "não há permissão no Volumes"
Verifique permissões no Unity Catalog no Databricks:
- Acesse Admin Console → Data
- Confirme permissões no Catalog `visagio`

### Tarefa falha no cronograma
Verifique:
1. Status do cluster Databricks
2. Permissões de acesso aos Volumes
3. Disponibilidade da API do Banco Central

---

## 👥 Contribuições

Para contribuir com melhorias:
1. Crie uma branch: `git checkout -b feature/nova-funcionalidade`
2. Faça commit das mudanças: `git commit -m "Adiciona nova funcionalidade"`
3. Push para a branch: `git push origin feature/nova-funcionalidade`
4. Abra um Pull Request

---

## 📞 Contato

Para dúvidas ou sugestões sobre o pipeline, entre em contato via:
- GitHub: [JairoGonzaga](https://github.com/JairoGonzaga)
- Email: jaironeto4828@gmail.com

---

**Última atualização**: Abril de 2026  
**Status**: ✅ Em produção