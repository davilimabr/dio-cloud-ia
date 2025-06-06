
# Guia de Transformação de Dados de Vendas em Insights

Este documento resume, de forma detalhada, o aprendizado adquirido nas aulas do módulo **Processamento de Dados para Geração de Insights**. Cada seção corresponde a um tópico abordado em aula, complementado com exemplos práticos de código **Python/pandas** e **prompts** que demonstram como explorar de maneira eficiente conjuntos de dados de vendas, como os fornecidos no repositório público [dataset‑gamesshop](https://github.com/digitalinnovationone/dataset-gamesshop) e nos arquivos disponibilizados pelos instrutores.

> **Objetivo:** Consolidar as melhores práticas para transformar dados brutos em informações acionáveis, utilizando um fluxo reproducível de preparação, análise e disponibilização de relatórios _self‑service_.

---

## 1. Introdução

A ciência de dados aplicada a vendas possibilita transformar registros transacionais em **insights** que sustentam decisões estratégicas (previsão de demanda, otimização de estoque, definição de campanhas de marketing e avaliação de performance).  
Nesta etapa:

* Definimos o escopo do problema.
* Alinhamos expectativas com as partes interessadas.
* Identificamos as perguntas de negócio prioritárias.

### Exemplo de prompt introdutório

> “Com base no histórico de vendas do primeiro semestre, quais produtos apresentaram maior crescimento percentual de receita e quais fatores podem ter contribuído para esse desempenho?”

---

## 2. Pré‑requisitos

| Categoria                       | Detalhes                                                                              |
|---------------------------------|---------------------------------------------------------------------------------------|
| Linguagem                       | Python 3.10 +                                                                          |
| Bibliotecas principais          | `pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`, `scikit‑learn` (opcional)        |
| Ferramentas de desenvolvimento  | VS Code / Jupyter Lab                                                                  |
| Ambiente                        | Conda ou `venv` + arquivo `requirements.txt`                                           |
| Controle de versão              | Git                                                                                    |
| Acesso a LLM                    | Conta em plataforma de IA generativa (ex.: ChatGPT ou modelo local)                    |
| Base de dados (opcional)        | SQLite / PostgreSQL para persistir dados processados                                   |

### Prompt para verificação de ambiente

> “Liste as versões instaladas de pandas, numpy e matplotlib no meu ambiente atual e aponte incompatibilidades conhecidas.”

---

## 3. Use Case

O **Caso de Uso** escolhido envolve a consolidação de vendas de consoles portáteis (ex.: “Anbernic” e “Meganium”) provenientes de múltiplos canais (AliExpress, Shopee, Etsy). O objetivo é:

1. **Unificar** as fontes em um único _dataset_.
2. **Normalizar** campos (datas, moedas, SKU).
3. **Calcular** métricas‑chave (receita, ticket médio, margem estimada).
4. **Gerar** dashboards interativos para stakeholders.

### Prompt para entendimento do caso

> “Explique, em linguagem executiva, como a unificação de dados multicanal apoia decisões de precificação dinâmica.”

---

## 4. Estrutura de Pastas

```text
project-root/
├── data/
│   ├── raw/            # arquivos originais (.csv, .xlsx, .json)
│   ├── interim/        # dados após limpeza inicial
│   └── processed/      # datasets prontos para consumo analítico
├── notebooks/          # análises exploratórias (.ipynb)
├── src/
│   ├── extract.py
│   ├── transform.py
│   ├── load.py
│   └── visualize.py
├── reports/
│   ├── figures/
│   └── dashboards/
└── README.md
```

### Prompt para geração automática da estrutura

> “Crie um script shell que gere a árvore de diretórios acima e adicione arquivos `.gitkeep` onde necessário.”

---

## 5. Criando o **Process Data**

Nesta fase desenvolvemos um **pipeline ETL** simples utilizando `pandas`.

```python
import pandas as pd
from pathlib import Path

RAW_PATH = Path("data/raw")
INTERIM_PATH = Path("data/interim")

def normalize_columns(df: pd.DataFrame) -> pd.DataFrame:
    """Padroniza nomes e tipos de colunas."""
    col_map = {
        "Order Date": "order_date",
        "Gross Sales": "gross_sales",
        "SKU": "sku",
        "Marketplace": "marketplace"
    }
    df = df.rename(columns=col_map)
    df["order_date"] = pd.to_datetime(df["order_date"], errors="coerce")
    df["gross_sales"] = (
        df["gross_sales"]
        .replace("[\$ R$]", "", regex=True)
        .astype(float)
    )
    return df

def process_file(path: Path) -> pd.DataFrame:
    """Lê, normaliza e retorna um DataFrame processado."""
    df = pd.read_csv(path)
    df = normalize_columns(df)
    df["source_file"] = path.name
    return df

frames = [process_file(p) for p in RAW_PATH.glob("*.csv")]
df_combined = pd.concat(frames, ignore_index=True)
df_combined.to_parquet(INTERIM_PATH / "sales_clean.parquet", index=False)
```

#### Prompt para validação de qualidade

> “Verifique se existem valores ausentes em `order_date` ou `gross_sales` e sugira tratamentos adequados.”

---

## 6. Consolidando

Consolidação implica agrupar e enriquecer os dados.

```python
import pycountry

def add_country(row):
    """Extrai país a partir do marketplace."""
    mapping = {"Shopee": "BR", "AliExpress": "CN", "Etsy": "US"}
    return pycountry.countries.get(alpha_2=mapping.get(row["marketplace"])).name

df = pd.read_parquet("data/interim/sales_clean.parquet")
df["country"] = df.apply(add_country, axis=1)

kpi = (
    df.groupby(["country", pd.Grouper(key="order_date", freq="M")])
      .agg(
          receita_total=("gross_sales", "sum"),
          pedidos=("sku", "count"),
          ticket_medio=("gross_sales", "mean")
      )
      .reset_index()
)
kpi.to_csv("data/processed/kpi_mensal.csv", index=False)
```

#### Prompt para geração de KPIs

> “Crie uma tabela comparando a receita total e o ticket médio de cada marketplace nos últimos três meses.”

---

## 7. Entendendo os Dados

### Análise Exploratória

* **Distribuição de vendas por marketplace.**  
* **Sazonalidade mensal.**  
* **Produtos campeões de receita.**

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.lineplot(
    data=kpi,
    x="order_date",
    y="receita_total",
    hue="country"
)
plt.title("Receita Mensal por País")
plt.tight_layout()
plt.show()
```

### Exemplos de prompts analíticos

1. “Identifique anomalias de volume de vendas superiores a 3 σ da média mensal.”  
2. “Sugira três hipóteses para queda de ticket médio em abril.”  
3. “Gere uma segmentação ABC dos SKUs com base em receita acumulada.”

---

## 8. Aonde Vamos Utilizar os Dados

| Área              | Aplicação prática                                                  |
|-------------------|--------------------------------------------------------------------|
| **Marketing**     | Otimização de campanhas pagas, *cross‑sell* e *up‑sell*.           |
| **Supply Chain**  | Previsão de demanda e negociação com fornecedores.                 |
| **Finanças**      | Projeção de fluxo de caixa e análise de margem.                    |
| **Produto**       | Priorização de roadmap segundo desempenho de categoria.            |

### Prompt para storytelling

> “Escreva um resumo executivo destacando 3 oportunidades de crescimento a partir dos dados de vendas consolidados.”

---

## 9. Self Service Report (versão paga)

### Ferramenta sugerida: **Power BI Pro**

1. Importar `kpi_mensal.csv` via *Get Data*.  
2. Configurar *Dataflows* para atualização automática (gateway).  
3. Criar visuais: gráficos de linha, mapa geográfico, matriz de ticket médio.  
4. Definir *Row‑Level Security* por marketplace.  
5. Publicar conjunto de dados e compartilhar **App**.

#### Prompt para automação no Power BI

> “Gere um script M no Power Query que converta a coluna `order_date` para padrão ISO‑8601 e preencha lacunas de meses sem venda.”

---

## 10. Self Service Report (versão gratuita)

### Ferramenta sugerida: **Google Looker Studio**

1. Conectar‑se ao Google Sheets com tabela `kpi_mensal`.  
2. Configurar *Blending* para adicionar metas mensais armazenadas em um segundo *sheet*.  
3. Criar controles de filtro por país e trimestre.  
4. Agendar envio automático em PDF para diretoria.

#### Prompt para explicação didática

> “Escreva um passo a passo para que um gestor sem conhecimentos técnicos crie um filtro de data relativa (últimos 90 dias) no Looker Studio.”

---

## 11. Conclusão

O fluxo apresentado — da ingestão de arquivos brutos à publicação de relatórios self‑service — evidencia como **prompts bem‑estruturados** potencializam a eficiência analítica. Ao delegar tarefas repetitivas (documentação de ambiente, validação de qualidade, geração de hipóteses) a uma IA generativa, direcionamos o esforço humano para a **interpretação** dos resultados e a **tomada de decisão**.

### Próximos passos sugeridos

* Implementar testes unitários para funções de transformação (*pytest*).  
* Integrar o pipeline a um orquestrador (*Prefect*, *Airflow*) para atualizações regulares.  
* Explorar modelos de previsão (ARIMA, Prophet) para planejamento de estoque.

---

> *Documento gerado em 06/06/2025.*
