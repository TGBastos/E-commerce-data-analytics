# 🛒 E-commerce Sales Analysis

Análise exploratória e de performance de um e-commerce de varejo com mais de **540 mil transações**. O projeto adota uma abordagem em camadas: **DuckDB** para consultas SQL otimizadas diretamente sobre os arquivos, **Pandas** para montagem dos DataFrames e **Plotly** para visualização interativa dos resultados.

---

## 📌 Contexto do problema

O dataset contém registros brutos de transações de um e-commerce internacional — incluindo vendas, devoluções e movimentações internas. O desafio central foi **separar o sinal do ruído**: filtrar apenas as vendas reais antes de qualquer análise, garantindo que as conclusões reflitam o comportamento real dos clientes.

---

## 🎯 Perguntas respondidas

- Qual o volume real de vendas após limpeza dos dados?
- Quais produtos lideram em quantidade vendida?
- Como as vendas se distribuem por país?
- Qual o valor médio e total por pedido?

---

## 🧱 Estrutura do projeto

```
ecommerceData/
├── complete_ecommerce_data.csv   # Dataset bruto original
├── vendas_simplificada.csv       # Dados tratados (gerado pelo pipeline)
├── Produtos_vendidos.csv         # Agregação por produto
│
├── analise_ecommerce_data.ipynb  # Exploração inicial do dataset bruto
├── analise_vendas.ipynb          # Pipeline de limpeza e transformação
└── analise_playground.ipynb      # Experimentos e análises ad hoc
```

> A separação entre notebooks **não é acidental** — reflete uma decisão arquitetural consciente: manter o pipeline de limpeza isolado das análises de negócio, favorecendo reuso e rastreabilidade.

---

## 🔄 Pipeline de dados

```
Dataset bruto (541k linhas)
        │
        ▼
[ 1. Remoção de transações sem descrição ]
        │  Elimina registros sem identificação de produto
        ▼
[ 2. Filtro de devoluções e movimentações internas ]
        │  Remove Quantity ≤ 0 e CustomerID nulo
        ▼
[ 3. Simplificação por pedido ]
        │  Agrega itens do mesmo InvoiceNo em uma única linha
        ▼
vendas_simplificada.csv  ← base para todas as análises
```

---

## 🛠️ Stack utilizada

| Ferramenta | Papel no projeto | Por que foi escolhida |
|---|---|---|
| **DuckDB** | Consultas SQL sobre os arquivos CSV | Motor OLAP colunar otimizado para analytics — processa grandes volumes diretamente nos arquivos, sem necessidade de banco de dados ou carga prévia em memória |
| **Pandas** | Montagem e manipulação dos DataFrames finais | Integração nativa com DuckDB (`.df()`), ideal para transformações tabulares após as queries |
| **Plotly** | Visualização interativa dos resultados | Gráficos interativos nativos no Jupyter, com exportação fácil para HTML |
| **Jupyter Notebook** | Exploração e documentação das análises | Combina código, resultado e narrativa em um único documento reproduzível |
| **Python 3** | Orquestração do pipeline | — |

---

## 💡 Destaques técnicos

**Por que DuckDB para as consultas?**
DuckDB é um banco de dados OLAP (Online Analytical Processing) colunar, projetado especificamente para cargas de trabalho analíticas. Diferente de soluções tradicionais, ele roda em processo (sem servidor), lê arquivos CSV e Parquet diretamente do disco com execução vetorizada e paralelizada, e se integra nativamente ao Pandas. Para este projeto, isso significa consultar 541k linhas com SQL completo sem nenhuma configuração de infraestrutura.

**Fluxo DuckDB → Pandas → Plotly**
O pipeline segue uma divisão clara de responsabilidades:

```python
import duckdb
import plotly.express as px

# 1. DuckDB faz o trabalho pesado: filtragem, agregação, joins
query = """
    SELECT
        InvoiceNo,
        InvoiceDate,
        CustomerID,
        Country,
        COUNT(Description)               AS TotalItems,
        SUM(Quantity)                    AS TotalQuantity,
        ROUND(SUM(Quantity * UnitPrice)) AS TotalRevenue
    FROM
        read_csv('complete_ecommerce_data.csv', encoding = 'latin-1')
    WHERE
        Description IS NOT NULL
        AND Quantity > 0
        AND CustomerID IS NOT NULL
    GROUP BY
        InvoiceNo, InvoiceDate, CustomerID, Country
    ORDER BY
        InvoiceNo ASC
"""

# 2. Pandas recebe o resultado já tratado como DataFrame
vendas = duckdb.sql(query).df()

# 3. Plotly transforma o DataFrame em visualização interativa
fig = px.bar(vendas.groupby('Country')['TotalRevenue'].sum().reset_index(),
             x='Country', y='TotalRevenue', title='Receita por País')
fig.show()
```

**Reutilização de resultados intermediários** — o DuckDB permite referenciar DataFrames Pandas diretamente como tabelas em queries subsequentes, evitando reprocessamento do arquivo original a cada etapa.

---

## 📊 Dataset

- **Fonte:** [UCI ML Repository — Online Retail Dataset](https://archive.ics.uci.edu/ml/datasets/Online+Retail)
- **Período:** Dezembro/2010 a Dezembro/2011
- **Volume bruto:** ~541.000 linhas
- **Colunas principais:** `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`

---

## ▶️ Como reproduzir

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/ecommerceData.git
cd ecommerceData

# 2. Instale as dependências
pip install duckdb pandas plotly jupyter

# 3. Execute os notebooks na ordem
#    Exploração inicial:
jupyter notebook analise_ecommerce_data.ipynb

#    Pipeline de limpeza (gera vendas_simplificada.csv):
jupyter notebook analise_vendas.ipynb
```

---

## 📁 Próximos passos

- [ ] Análise de sazonalidade (série temporal de receita por mês)
- [ ] Segmentação de clientes por país e valor de pedido
- [ ] Visualizações interativas com Plotly (receita por país, top produtos, evolução mensal)
- [ ] Análise de produtos com maior receita vs. maior volume

---

*Projeto desenvolvido para fins de portfólio e aprendizado em análise de dados.*
