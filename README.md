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

### 1. ¿Cuáles son los 10 productos más vendidos?
Identificar los productos que generan el mayor volumen de ventas y contribuyen significativamente a los ingresos de la empresa.

````sql
SELECT TOP 10 Product_Name, SUM(Sales) AS Ventas_Totales
FROM Orders
GROUP BY Product_Name
ORDER BY Ventas_Totales DESC;

````



### 2. ¿Qué clientes generan más ingresos para la empresa?

Determinar qué clientes tienen el mayor impacto en las ventas totales.
````sql
SELECT TOP 5 Customer_Name, SUM(Sales) AS Monto_Gastado
FROM Orders
GROUP BY Customer_Name
ORDER BY Monto_Gastado DESC;
````



### 3. ¿Qué porcentaje de las ventas totales aporta cada región?

Analizar la distribución geográfica de los ingresos e identificar las regiones más importantes para el negocio.
````sql
SELECT Region, SUM(Sales) AS Total_Ventas, ROUND(SUM(Sales)*100/SUM(SUM(Sales)) OVER(),2) AS Porcentaje_Ventas
FROM Orders
GROUP BY Region
ORDER BY Porcentaje_Ventas;
````



### 4. ¿Cómo han evolucionado las ventas a lo largo de los años?

Evaluar el crecimiento o disminución de las ventas mediante una comparación año contra año.
````sql
WITH Ventas_Anuales AS(
	SELECT YEAR(Order_Date) AS Año, SUM(Sales) AS Ventas_Totales
	FROM Orders
	GROUP BY YEAR(Order_Date)
)
SELECT Año, Ventas_Totales,
		LAG(Ventas_Totales) OVER(ORDER BY Año) AS Año_Anterior,
		Ventas_Totales - LAG(Ventas_Totales) OVER(ORDER BY Año) AS Diferencia_Ventas,
		ROUND(
			(Ventas_Totales-LAG(Ventas_Totales) OVER(ORDER BY Año))/LAG(Ventas_Totales) OVER(ORDER BY Año)*100,2
		) AS Porcentaje_Crecimiento
FROM Ventas_Anuales;
````



### 5. ¿Qué categorías de productos generan más beneficio?

Identificar qué categorías aportan la mayor rentabilidad al negocio.
````sql
SELECT Category, SUM(Profit) AS Beneficio_Total
FROM Orders
GROUP BY Category
ORDER BY Beneficio_Total;
````



### 6. ¿Qué productos tienen el mayor margen de beneficio?

Determinar qué productos son más eficientes generando ganancias en relación con sus ventas.
````sql
SELECT  TOP 10 Product_Name, SUM(Sales) AS Ventas_Totales, SUM(Profit) AS Ganancia, ROUND(SUM(Profit)*100/SUM(Sales),2) AS Ganancia_Total
FROM Orders
GROUP BY Product_Name
ORDER BY Ganancia_Total DESC;
````


### 7. ¿Cómo afectan los descuentos a la rentabilidad?

Analizar el impacto de los diferentes niveles de descuento sobre los beneficios obtenidos.
````sql
SELECT CASE
			WHEN Discount = 0 THEN '0%'
			WHEN Discount <= 0.1 THEN '0% - 10%'
			WHEN Discount <= 0.2 THEN '11% - 20%'
			WHEN Discount <= 0.3 THEN '21% - 30%'
			ELSE 'Más del 30%'
		END AS Porcentaje_Descuento, COUNT(*) AS Cantidad_Pedidos, ROUND(AVG(Profit),2) AS Promedio_Ganancia, ROUND(SUM(Profit),2) AS Total_Ganancia
FROM Orders
GROUP BY CASE
			WHEN Discount = 0 THEN '0%'
			WHEN Discount <= 0.1 THEN '0% - 10%'
			WHEN Discount <= 0.2 THEN '11% - 20%'
			WHEN Discount <= 0.3 THEN '21% - 30%'
			ELSE 'Más del 30%'
		END
ORDER BY Porcentaje_Descuento;
````


### 8. ¿Qué clientes realizan más pedidos?

Identificar los clientes más frecuentes y comprender mejor el comportamiento de compra.
````sql
SELECT TOP 10 Customer_Name, COUNT(DISTINCT Order_ID) AS Cantidad_Pedidos
FROM Orders
GROUP BY Customer_Name
ORDER BY Cantidad_Pedidos DESC;
````


### 9. ¿Cuál fue el producto más vendido de cada año?

Detectar qué productos lideraron las ventas en cada periodo anual.
````sql
WITH Ventas_Anuales AS(
	SELECT YEAR(Order_Date) AS Año, Product_Name, SUM(Sales) AS Ventas_Totales
	FROM Orders
	GROUP BY YEAR(Order_Date), Product_Name
), Ranked_Productos AS (
	SELECT Año, Product_Name, Ventas_Totales, ROW_NUMBER() OVER(
		PARTITION BY Año
		ORDER BY Ventas_Totales DESC) AS Ranking 
 	FROM Ventas_Anuales
)
SELECT Año, Product_Name, Ventas_Totales
FROM Ranked_Productos
WHERE Ranking =1
ORDER BY Año;
````


### 10. ¿Qué productos mantuvieron un crecimiento sostenido durante tres años consecutivos?

Identificar productos con tendencias de crecimiento consistentes a largo plazo.
````sql
WITH Ventas_Anuales AS (
	SELECT Product_Name, SUM(Sales) AS Total_Ventas, YEAR(Order_Date) AS Año
	FROM Orders
	GROUP BY Product_Name,YEAR(Order_Date)
),
Historial_Ventas AS (
					SELECT Product_Name, Año, Total_Ventas,
							LAG(Total_Ventas, 1) OVER(PARTITION BY Product_Name ORDER BY Año) AS Hace_Un_Año,
							LAG(Total_Ventas, 2) OVER(PARTITION BY Product_Name ORDER BY Año) AS Hace_Dos_Años
					FROM Ventas_Anuales
)
SELECT Product_Name, Año, Total_Ventas, Hace_Un_Año, Hace_Dos_Años
FROM Historial_Ventas
WHERE Total_Ventas>Hace_Un_Año AND Hace_Un_Año>Hace_Dos_Años;
````


---
