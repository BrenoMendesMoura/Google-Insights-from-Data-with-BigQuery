# Google-Insights-from-Data-with-BigQuery
<br>

> ### 1 - Crie uma consulta para responder à pergunta: "Qual foi o total de casos confirmados em April 25, 2020 ?".
```sql
`SELECT SUM(cumulative_confirmed) AS total_cases_worldwide 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE date='2020-04-25'`
```
<br>

> ### 2 - Crie uma consulta para responder à pergunta: "Quantos estados nos EUA tiveram mais de 250 mortes em April 25, 2020 ?"
```sql
`SELECT COUNT(*) AS count_of_states
FROM (
    SELECT 
        subregion1_name AS state, 
        SUM(cumulative_deceased) AS death_count
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
    WHERE country_name="United States of America" AND date='2020-04-25' AND subregion1_name IS NOT NULL
    GROUP BY subregion1_name
)
WHERE death_count > 250`
```
<br>

> ### 3 - Crie uma consulta para responder à pergunta: "Quais estados dos EUA tiveram mais de 2000 casos confirmados em April 25, 2020 ? Liste todos".
```sql
`SELECT 
    subregion1_name AS state,
    SUM(cumulative_confirmed) AS total_confirmed_cases
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name="United States of America" AND date='2020-04-25' AND subregion1_name IS NOT NULL
GROUP BY subregion1_name
HAVING total_confirmed_cases > 2000
ORDER BY total_confirmed_cases DESC`
```
<br>

> ### 4 - Crie uma consulta para responder à pergunta: "Qual foi a taxa de letalidade dos casos na Itália em June de 2020?".
```sql
`SELECT 
    SUM(cumulative_confirmed) AS total_confirmed_cases,
    SUM(cumulative_deceased) AS total_deaths,
    (SUM(cumulative_deceased)/SUM(cumulative_confirmed))*100 AS case_fatality_ratio
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name="Italy" AND date BETWEEN "2020-06-01" AND "2020-06-30"
`
```
<br>

> ### 5 - Crie uma consulta para responder à pergunta: "Em que dia o total de mortes passou de 8000 na Itália?". 

```sql
`SELECT date
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` 
WHERE country_name="Italy" AND cumulative_deceased > 8000
ORDER BY date ASC
LIMIT 1`
```
<br>

> ### 6 - A consulta a seguir foi criada para identificar o número de dias na Índia entre 21, Feb 2020 e 15, March 2020 sem aumento no número de casos confirmados.
```sql
`WITH india_cases_by_date AS (
    SELECT
        date,
        SUM(cumulative_confirmed) AS cases
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE country_name ="India" AND date BETWEEN '2020-02-21' AND '2020-03-15'
    GROUP BY date
    ORDER BY date ASC 
), 
india_previous_day_comparison AS (
    SELECT
        date,
        cases,
        LAG(cases) OVER(ORDER BY date) AS previous_day,
        cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
    FROM india_cases_by_date
)

SELECT COUNT(*) AS days_with_zero_increase
FROM india_previous_day_comparison
WHERE net_new_cases=0
`
```
<br>

> ### 7 - Com base na consulta anterior, crie uma consulta para descobrir as datas em que os casos confirmados aumentaram mais de 20 % em comparação ao dia anterior (indicando a taxa de duplicação de aproximadamente sete dias) nos EUA entre 22 de março e 20 de abril de 2020.


```sql
`WITH us_cases_by_date AS (
    SELECT
        date,
        SUM( cumulative_confirmed ) AS cases
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE country_name="United States of America" AND date between '2020-03-22' and '2020-04-20'
    GROUP BY date
    ORDER BY date ASC
), 
us_previous_day_comparison AS (
    SELECT
        date,
        cases,
        LAG(cases) OVER(ORDER BY date) AS previous_day,
        cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,
       (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
    FROM us_cases_by_date
)
SELECT
    Date,
    cases AS Confirmed_Cases_On_Day,
    previous_day AS Confirmed_Cases_Previous_Day,
    percentage_increase AS Percentage_Increase_In_Cases
FROM us_previous_day_comparison
WHERE percentage_increase > 20`
```

> ### 8 - Crie uma consulta para listar as taxas de recuperação dos países em ordem decrescente ( 20 no máximo) até 10 de maio de 2020.
```sql
`WITH cases_by_country AS (
    SELECT
        country_name AS country,
        SUM(cumulative_confirmed) AS cases,
        SUM(cumulative_recovered) AS recovered_cases
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE date='2020-05-10'
    GROUP BY country_name
), 
recovered_rate AS (
    SELECT
        country, cases, recovered_cases,
        (recovered_cases * 100)/cases AS recovery_rate
    FROM cases_by_country
)

SELECT country, cases AS confirmed_cases, recovered_cases, recovery_rate
FROM recovered_rate
WHERE cases > 50000
ORDER BY recovery_rate desc
LIMIT 20`
```
<br>

> ### 9 - A consulta não está sendo executada corretamente. Corrija o erro.
> Analytic function LEAD cannot be called without an OVER clause at
```sql
`WITH france_cases AS (
    SELECT
        date,
        SUM(cumulative_confirmed) AS total_cases
    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
    WHERE country_name="France" AND date IN ('2020-01-24', '2020-05-10')
    GROUP BY date
    ORDER BY date
), 
summary AS (
    SELECT
        total_cases AS first_day_cases,
        LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
        DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
    FROM france_cases
    LIMIT 1
)

SELECT 
    first_day_cases, 
    last_day_cases, 
    days_diff, 
    POWER(last_day_cases/first_day_cases,1/days_diff)-1 as cdgr
FROM summary`
```
<br>

> ### 10 - Crie um relatório do Datastudio que represente o seguinte para os Estados Unidos:
> - Número de casos confirmados
> - Número de mortes
> - Date range : 2020-03-15 to 2020-04-23![desafio](https://user-images.githubusercontent.com/80074264/202470442-4f07ab51-ce38-49fd-9c2d-d4be8354fe15.png)

```sql
`SELECT
    date, SUM(cumulative_confirmed) AS country_cases,
    SUM(cumulative_deceased) AS country_deaths
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN '2020-03-15' AND '2020-04-23' AND country_name ="Uni![Uploading desafio.png…]()
ted States of America"
GROUP BY date`
```
