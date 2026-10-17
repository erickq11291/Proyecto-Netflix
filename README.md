# 🎬 Netflix: Comportamiento de usuarios y análisis de abandono
Comportamiento de clientes y análisis de abandono

## 📌 Descripción del Proyecto

Este proyecto explora el comportamiento de los suscriptores de Netflix utilizando SQL Server y un dataset enfocado en consumo de contenido y abandono de clientes.

A través de diferentes preguntas de negocio, se analizan patrones relacionados con preferencias de contenido, segmentación de usuarios, hábitos de visualización, suscripciones y churn, con el objetivo de comprender cuáles son los factores que influyen en la permanencia o abandono de la plataforma.

---

## 🎯 Objetivos

Utilizar SQL para analizar el comportamiento de los clientes de Netflix, identificar patrones de consumo, comparar segmentos de usuarios y evaluar indicadores de retención que permitan comprender mejor la dinámica de suscripción y abandono de la plataforma.

---

## 🧠 Habilidades SQL Demostradas

- GROUP BY
- ORDER BY
- Funciones de agregación (`SUM`, `AVG`, `COUNT`)
- CASE WHEN
- Common Table Expressions (CTEs)
- Window Functions
- ROW_NUMBER()
- RANK()
- OVER()

---

## 📈 Preguntas de Negocio Analizadas

### 1. ¿Cómo se distribuyen los clientes entre los planes Basic, Standard y Premium, y qué participación representa cada uno sobre el total de suscriptores?
Comprender la composición de la cartera de clientes y el peso relativo de cada plan dentro de la base total de suscriptores.

````sql
SELECT subscription_type AS Tipo_Suscripcion, COUNT(*) AS Total_Usuarios, CAST(COUNT(*)*100.0/SUM(COUNT(*)) OVER() AS DECIMAL(5,2)) AS Porcentaje_Usuarios
FROM netflix_customers
GROUP BY subscription_type;
````



### 2. ¿Qué regiones concentran la mayor cantidad de suscriptores y cuáles generan más ingresos estimados?
Identificar los mercados más importantes para la plataforma considerando tanto volumen de clientes como potencial de ingresos.

````sql
SELECT region, COUNT(*) AS Total_Usuarios, CAST(AVG(monthly_fee) AS DECIMAL(5,2)) AS Tarifa_Promedio
FROM netflix_customers
GROUP BY region
ORDER BY Tarifa_Promedio DESC;
````



### 3. ¿Qué perfiles de usuarios presentan los niveles de engagement más altos dentro de la plataforma?

Identificar qué combinaciones de plan de suscripción y región están asociadas con los mayores niveles de consumo e interacción dentro de Netflix.
````sql
WITH Engagement AS(
	SELECT subscription_type AS Tipo_Suscripcion, region, CAST(AVG(watch_hours) AS DECIMAL(5,2)) AS Promedio_Horas_Mes, AVG(number_of_profiles) AS Promedio_Perfiles
	FROM netflix_customers
	GROUP BY subscription_type, region
)
SELECT *, RANK() OVER(ORDER BY Promedio_Horas_Mes DESC) AS Ranking_Engagement
FROM Engagement;
````



### 4. ¿Cuál es el género favorito más popular dentro de cada región?

Identificar las preferencias de contenido predominantes en cada mercado y detectar diferencias regionales en los gustos de los usuarios.
````sql
WITH Peliculas_Favoritas AS (
	SELECT region, favorite_genre AS Genero_Favorito, COUNT(*) AS Total_Usuarios
	FROM netflix_customers
	GROUP BY region, favorite_genre
), Ranked_Peliculas AS (
	SELECT region, Genero_Favorito, Total_Usuarios, ROW_NUMBER() OVER(
		PARTITION BY region
		ORDER BY Total_Usuarios DESC) AS Ranking
	FROM Peliculas_Favoritas
)
SELECT region, Genero_Favorito, Total_Usuarios
FROM Ranked_Peliculas
WHERE Ranking=1
ORDER BY Total_Usuarios;
````


### 5. ¿Qué diferencias existen en el consumo de contenido entre usuarios de TV, móvil, computadora y otros dispositivos?

Comparar los hábitos de visualización según el dispositivo utilizado para acceder a la plataforma.
````sql
SELECT device AS Dispositivo, COUNT(*) AS Total_Usuarios,
		CAST(AVG(watch_hours) AS DECIMAL(5,2)) AS Promedio_Horas_Mes,
		AVG(last_login_days) AS Promedio_Dias_Ultimo_Login,
		AVG(number_of_profiles) AS Promedio_Cantidad_Perfiles
		
FROM netflix_customers
GROUP BY device
ORDER BY Promedio_Horas_Mes;
````



### 6. ¿Qué diferencias existen en el comportamiento de consumo entre grupos de edad y género?

Analizar cómo varían las horas de visualización y las preferencias de contenido entre distintos perfiles demográficos.
````sql
SELECT CASE WHEN age < 20 THEN '18 - 30 años'
			WHEN age < 40 THEN '30 - 45 años'
			WHEN age < 60 THEN '45 - 60 años'
			ELSE 'Más de 60 años' END AS Rango_Edad,
		gender, COUNT(*) AS Total_Usuarios,
		CAST(AVG(watch_hours) AS DECIMAL(5,2)) AS Promedio_Horas_Mes,
		AVG(last_login_days) AS Promedio_Dias_Ultimo_Login,
		AVG(number_of_profiles) AS Promedio_Cantidad_Perfiles
FROM netflix_customers
GROUP BY CASE WHEN age < 20 THEN '18 - 30 años'
			WHEN age < 40 THEN '30 - 45 años'
			WHEN age < 60 THEN '45 - 60 años'
			ELSE 'Más de 60 años' END, gender
ORDER BY Rango_Edad;
````


### 7. ¿Cómo se relaciona el número de perfiles de una cuenta con sus horas de visualización y tarifa mensual?

Explorar si las cuentas con más perfiles presentan patrones diferentes de consumo y gasto.
````sql
SELECT number_of_profiles AS Cantidad_Perfiles, monthly_fee AS Tarifa_Mensual, SUM(watch_hours) AS Horas_Contenido
FROM netflix_customers
GROUP BY number_of_profiles, monthly_fee
ORDER BY number_of_profiles;
````


### 8. ¿Cuál es la tasa general de abandono y cómo varía según el tipo de suscripción?

Medir el nivel de churn de la plataforma y determinar si ciertos planes presentan mayores problemas de retención.
````sql
SELECT subscription_type AS Tipo_Suscripcion, COUNT(*) AS Total_Usuarios,
		SUM(CASE WHEN churned = 1 THEN 1 ELSE 0 END) AS Total_Churn,
		CAST(SUM(CASE WHEN churned = 1 THEN 1 ELSE 0 END)*100.0/COUNT(*) AS DECIMAL(5,2)) AS Tasa_Churn
FROM netflix_customers
GROUP BY subscription_type
ORDER BY Tasa_Churn DESC;
````


### 9. ¿Qué características diferencian a los clientes que abandonan de aquellos que permanecen, considerando inactividad, consumo y tarifa mensual?

Detectar patrones de comportamiento asociados al abandono y comprender qué factores pueden influir en la decisión de cancelar la suscripción.
````sql
SELECT CASE 
			WHEN churned=1 THEN 'Abandona'
			WHEN churned=0 THEN 'Permanece'
			END AS Churn,
	
		COUNT(churned) AS Total_Usuarios,
		CAST(COUNT(*)*100.0/SUM(COUNT(*)) OVER() AS DECIMAL(5,2)) AS Tasa_Abandono,
		AVG(last_login_days) AS Promedio_Dias_Ultimo_Login,
		CAST(AVG(watch_hours) AS DECIMAL(5,2)) AS Promedio_Horas_Consumo,
		CAST(AVG(monthly_fee) AS DECIMAL(5,2)) AS Promedio_Tarifa_Mensual
FROM netflix_customers
GROUP BY churned;
````


### 10. ¿Qué segmentos de clientes presentan mayor riesgo de abandono?

Identificar grupos específicos de usuarios con alta probabilidad de churn para orientar futuras estrategias de retención y fidelización.
````sql
WITH Tasa_Abandono AS(
	SELECT subscription_type AS Tipo_Suscripcion, device AS Dispositivo, region, COUNT(*) AS Total_Usuarios,
			SUM(CASE WHEN churned=1 THEN 1 ELSE 0 END) AS Usuarios_Abandonaron,
			CAST(SUM(CASE WHEN churned=1 THEN 1 ELSE 0 END) *100.0/COUNT(*) AS DECIMAL(5,2)) AS Tasa_Abandono
	FROM netflix_customers
	GROUP BY subscription_type, device, region
)
SELECT *, RANK() OVER(ORDER BY Tasa_Abandono DESC) AS Ranking_Riesgo
FROM Tasa_Abandono;
````


---
