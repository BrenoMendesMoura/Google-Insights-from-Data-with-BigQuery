# Google-Insights-from-Data-with-BigQuery

## 1 - Crie uma consulta para responder à pergunta: "Qual foi o total de casos confirmados em April 25, 2020 ?".
```sql
`SELECT SUM(cumulative_confirmed) AS total_cases_worldwide 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE date='2020-04-25'`
```
