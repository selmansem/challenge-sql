# Challenge
- RDBMS: MySQL
- Ruta: libreta.sql
- Autor: Souhaib E.M.
- Fecha: 02/07/2022
- GitHub: [GitHub Account](https://github.com/selmansem)

## Reto 1: Identificando competidores más relevantes
<p>¿Qué competidores deberíamos mirar más de cerca? Buscamos identificar a aquellos competidores que venden los
mismos productos que nosotros.

Identificar top 4 competidores, según porcentaje de catálogo coincidente con de Amazon.</p>

*Ejemplo:*

| Ranking |    Competidor    | Nro ASINs | % ASINs sobre total en Amazon |
|:-------:|:----------------:|:---------:|:-----------------------------:|
|    1    |    Bestbuy.com   |    812    |              84%              |
|    2    | bhphotovideo.com |    419    |              51%              |
|    3    |    Walmart.com   |    234    |              24%              |
|    4    |   Beach Camera   |     70    |               7%              |

```SQL
SELECT
    Competidor, -- Nombre del competidor
    COUNT(DISTINCT ASIN) AS Nro_ASINs, -- Número de ASINs
    ROUND(COUNT(DISTINCT ASIN) / 500, 2) AS PRTG_ASINs_sobre_total_en_Amazon -- Porcentaje de ASINs sobre el total en Amazon
FROM (
    SELECT DISTINCT c.prices_merchant AS Competidor, c.asin AS ASIN FROM competitors_v1 c
    INNER JOIN product_data_v1 ON product_data_v1.asin = c.asin
) SUB
GROUP BY SUB.Competidor
ORDER BY Nro_ASINs DESC
LIMIT 4;
```

## Reto 2: Identificando categorías más relevantes

<p>¿Qué categorías de productos son las más importantes? Buscamos identificar a aquellas categorías de productos con
mayor número de reviews.

Identificar top 5 categorías, según porcentaje de reviews.</p>

*Ejemplo:*

| **Ranking** | **Product Category** | **Count of ASINs** | **Share in total ASINs** | **SUM of reviews** | **Share in total reviews** |
|:-----------:|:--------------------:|:------------------:|:------------------------:|:------------------:|:--------------------------:|
|    **1**    |        **PC**        |         269        |            28%           |        93803       |             45%            |
|    **2**    |    **electronic**    |         374        |            39%           |        66034       |             32%            |
|    **3**    |     **wireless**     |         122        |            13%           |        21971       |             10%            |
|    **4**    |        camera        |         72         |            7%            |        13930       |             7%             |
|    **5**    |  home_entretainment  |         81         |            8%            |        11086       |             5%             |
|    **6**    |   music_instruments  |         12         |            1%            |        1713        |             1%             |

```SQL
SELECT
    RANK() OVER (ORDER BY Share_in_total_reviews DESC) AS Ranking, -- Enumerador de filas
    p.product_category AS Product_Category, -- Categoría de producto
    COUNT(p.asin) AS Count_of_ASINs, -- Número de ASINs
    CONCAT(ROUND(COUNT(p.asin) * 100 / (SELECT COUNT(c.asin) FROM product_data_v1 c)), "%") AS Share_in_total_ASINs, -- Porcentaje de ASINs en total
    SUM(p.number_of_reviews) AS SUM_of_reviews, -- Suma de reviews
    CONCAT(ROUND(COUNT(p.number_of_reviews) * 100 / (SELECT COUNT(a.number_of_reviews) FROM product_data_v1 a)), "%") AS Share_in_total_reviews -- Porcentaje de reviews en total
FROM product_data_v1 p
GROUP BY Product_Category
ORDER BY Share_in_total_reviews DESC
LIMIT 5;
```

## Reto 3: Adquisición de nuevos clientes

<p>La tabla de ecommerce contiene datos de órdenes a lo largo de dos años (2010 y 2011). Extraer porcentaje de clientes nuevos mensuales a lo largo de 2011.
Nota: entendemos un cliente nuevo aquel que ha sido captado en 2011, es decir, que no ha comprado durante 2010.</p>

*Ejemplo:*

| **Row** | **year** | **month** | **total_shoppers** | **new_shoppers** | **existing_shoppers** | **new_shopper_rate** |
|:-------:|:--------:|:---------:|:------------------:|:----------------:|:---------------------:|:--------------------:|
| 1       |   2011   |         1 |                784 |              421 |                   363 |                 0.54 |
| 2       |   2011   |         2 |                799 |              481 |                   318 |                  0.6 |
| 3       |   2011   |         3 |               1021 |              653 |                   368 |                 0.64 |
| 4       |   2011   |         4 |                900 |              558 |                   342 |                 0.62 |
| 5       |   2011   |         5 |               1080 |              703 |                   377 |                 0.65 |
| 6       |   2011   |         6 |               1052 |              691 |                   361 |                 0.66 |
| 7       |   2011   |         7 |                994 |              657 |                   337 |                 0.66 |
| 8       |   2011   |         8 |                981 |              644 |                   337 |                 0.66 |
| 9       |   2011   |         9 |               1303 |              928 |                   375 |                 0.71 |
| 10      |   2011   |        10 |               1426 |             1071 |                   355 |                 0.75 |
| 11      |   2011   |        11 |               1712 |             1237 |                   475 |                 0.72 |
| 12      |   2011   |        12 |                687 |              426 |                   261 |                 0.62 |

```SQL
SELECT
	j.Row_ AS Row, -- Enumerador de filas
	EXTRACT(YEAR FROM e.InvoiceDate) AS year, -- Año
	EXTRACT(MONTH FROM e.InvoiceDate) AS month, -- Mes
	h.total_shoppers AS total_shoppers, -- Total compradores por mes en 2011
	i.new_shoppers AS new_shoppers, -- Nuevos compradores en 2011 por mes
	(total_shoppers-new_shoppers) AS existing_shoppers, -- Antiguos compradores
	ROUND((new_shoppers / total_shoppers), 2) AS new_shopper_rate -- Ratio nuevos usuarios
FROM
	ecommerce_formatdate e
	INNER JOIN
		(SELECT EXTRACT(MONTH FROM g.InvoiceDate) AS mes, count(distinct g.CustomerID) AS total_shoppers FROM ecommerce_formatdate g GROUP BY mes) AS h
		ON EXTRACT(MONTH FROM e.InvoiceDate) = h.mes
	INNER JOIN
		(SELECT EXTRACT(MONTH FROM f.InvoiceDate) AS mes, count(distinct f.CustomerID) AS new_shoppers FROM ecommerce_formatdate f WHERE EXTRACT(YEAR FROM f.InvoiceDate) = 2011 GROUP BY mes) AS i
		ON EXTRACT(MONTH FROM e.InvoiceDate) = i.mes
	INNER JOIN
		(SELECT RANK() OVER (ORDER BY month) AS Row_, month FROM (SELECT DISTINCT EXTRACT(MONTH FROM InvoiceDate) AS month FROM ecommerce_formatdate) AS t) AS j
		ON EXTRACT(MONTH FROM e.InvoiceDate) = j.month
WHERE
    EXTRACT(YEAR FROM e.InvoiceDate) = 2011
GROUP BY
	h.mes;
```

<p align="center"><a href="#challenge">⇈ Volver arriba ⇈</a></p>
