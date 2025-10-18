# 🧾 Notebook SUSEP (PySpark + SQL)

## 🧠 Contexto Geral

O notebook tem como objetivo analisar **dados da SUSEP** (Superintendência de Seguros Privados), normalmente usados em estudos de seguros, sinistros e planos de previdência.
Ele combina **PySpark** e **Spark SQL** para:

* Ler e explorar grandes conjuntos de dados;
* Criar *views* temporárias reutilizáveis;
* Executar consultas analíticas;
* Gerar indicadores e proporções de forma eficiente e escalável.

---

## ⚙️ Estrutura do Notebook

O notebook é dividido em três blocos principais:

1. **Configuração e carregamento dos dados**
2. **Criação de *views* temporárias e funções auxiliares**
3. **Consultas SQL para análise dos dados**

---

## 🧩 1. Configuração e Carregamento dos Dados

Aqui o código inicial importa as bibliotecas do PySpark e configura o ambiente Databricks.

### Exemplo

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SUSEP Analysis").getOrCreate()
```

* Cria uma sessão Spark (objeto `spark`) — que é a “porta de entrada” para manipular DataFrames distribuídos.
* Garante compatibilidade com Databricks, mas também permite rodar localmente.

---

## 🧰 2. Criação de Funções Auxiliares

Duas funções centrais aparecem no seu notebook:
`criar_view_temporaria()` e `exibir_resultado()`.

### 🔹 `criar_view_temporaria(df, nome)`

```python
def criar_view_temporaria(df, nome):
    df.createOrReplaceTempView(nome)
```

📘 **O que faz:**
Cria uma *view temporária* acessível via SQL.
Ela é descartada automaticamente ao encerrar a sessão Spark.

💡 **Por que usar isso:**
Permite executar consultas SQL diretamente com `%sql` ou `spark.sql("SELECT ...")` sem precisar referenciar o DataFrame em Python.
Ideal para manter o notebook limpo e modular.

---

### 🔹 `exibir_resultado(query)`

```python
def exibir_resultado(query):
    return spark.sql(query).show()
```

📘 **O que faz:**
Executa uma string SQL e exibe o resultado formatado no Databricks.

💡 **Por que é útil:**
Facilita a leitura e depuração dos resultados sem precisar criar variáveis intermediárias.

---

## 🧮 3. Exploração e Transformação dos Dados

Após carregar os dados brutos (por exemplo, de um CSV ou tabela do Databricks), você usa transformações SQL para:

* **Filtrar colunas relevantes**
* **Converter tipos de dados**
  (ex.: de `string` para `float` ou `int`)
* **Tratar valores inválidos com `TRY_CAST`**

### Exemplo

```sql
SELECT
  TRY_CAST(valor_pago AS DOUBLE) AS valor_pago,
  TRY_CAST(quantidade AS INT) AS quantidade,
  categoria
FROM susep_raw
```

📘 **O que `TRY_CAST` faz:**
Tenta converter o valor para o tipo desejado — se falhar (por exemplo, por causa de caracteres não numéricos), o Spark retorna `NULL` em vez de gerar erro.
Isso evita que a query quebre e é ótimo para dados públicos com inconsistências.

---

## 📊 4. Criação de Views Temporárias

Depois de limpar e converter os dados, você cria *views* intermediárias, como:

* `vendas_por_regiao`
* `sinistros_por_ano`
* `proporcao_produtos`

Cada *view* representa uma etapa do pipeline de análise — como se fossem “tabelas virtuais” encadeadas.

💡 **Benefício:**
Você pode escrever consultas SQL de alto nível sobre resultados já tratados, sem reprocessar tudo do zero.

---

## 📈 5. Consultas SQL Analíticas

Aqui estão os principais tipos de análise realizados:

### 🔹 5.1. Contagem e agregação

```sql
SELECT regiao, COUNT(*) AS total
FROM vendas_por_regiao
GROUP BY regiao
```

→ Mostra o total de registros (ou planos, sinistros etc.) por região.

---

### 🔹 5.2. Cálculos de médias e somas ponderadas

```sql
SELECT
  categoria,
  SUM(valor_pago) / SUM(quantidade) AS valor_medio_ponderado
FROM susep_view
GROUP BY categoria
```

📘 **Por que “ponderado”?**
Em vez de calcular uma média simples, o cálculo leva em conta o peso (quantidade ou valor total).
Isso é essencial para evitar distorções quando há categorias com tamanhos muito diferentes.

---

### 🔹 5.3. Uso de filtros condicionais e joins

```sql
SELECT
  a.categoria,
  b.total_sinistros
FROM categorias a
JOIN sinistros b
ON a.id = b.id_categoria
WHERE b.ano >= 2020
```

→ Combina informações de diferentes tabelas (por exemplo, produtos x sinistros) e aplica filtros temporais.

---

## 🔍 6. Interpretação dos Resultados

Os resultados são normalmente exibidos com:

```python
display(df)
# ou
%sql SELECT * FROM view_analitica
```

Isso permite gerar **tabelas interativas** no Databricks e visualizar:

* Evolução temporal (ano a ano);
* Comparações por região, categoria ou produto;
* Indicadores de sinistralidade, valor médio, proporção de crescimento etc.

---

## 🧾 7. Boas Práticas Aplicadas

| Prática                      | Por que é importante                                                 |
| ---------------------------- | -------------------------------------------------------------------- |
| Uso de `TRY_CAST`            | Evita falhas de conversão e garante integridade numérica             |
| Views temporárias            | Aumentam a modularidade e legibilidade das consultas                 |
| Queries SQL separadas        | Facilitam depuração e interpretação incremental                      |
| Spark SQL + PySpark juntos   | Combina a facilidade do SQL com a performance do Spark distribuído   |
| Funções auxiliares genéricas | Reutilizáveis em outros projetos de dados públicos (IBGE, INEP etc.) |

---

## 🧩 8. Conclusão

O notebook da SUSEP foi estruturado para ser:

* **Escalável:** processa milhões de registros sem travar a execução;
* **Legível:** separa claramente cada etapa de transformação e análise;
* **Reutilizável:** fácil de adaptar para novas bases públicas com estrutura semelhante;
* **Seguro:** uso de `TRY_CAST`, filtragens e conversões bem controladas.