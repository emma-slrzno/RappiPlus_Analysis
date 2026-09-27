# Analisis-Comercial-Inmobiliario
[![Vista previa del Dashboard](Images/Dashboard.png)](https://public.tableau.com/views/Project_17885024557020/Dashboard2?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

👉 [Ver el dashboard interactivo en Tableau Public](https://public.tableau.com/views/Project_17885024557020/Dashboard2?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)
# Proyecto RappiPlus: de datos a decisiones de negocio

## Objetivo

Evaluar el desempeño del servicio **RappiPlus** para apoyar decisiones de negocio basadas en datos, combinando información de pedidos, catálogo de productos, inversión en marketing, comportamiento de usuario (SQL) y un experimento A/B en el checkout. El proyecto está pensado para los equipos de **negocio, marketing y producto**, que necesitan saber si la operación es rentable, dónde se pierden los usuarios y si los cambios de producto realmente generan impacto.

**Preguntas que responde el dashboard:**
- ¿Es rentable el negocio de RappiPlus? (revenue, costos, marketing y profit)
- ¿En qué etapa del funnel de conversión se pierden más usuarios?
- ¿Los usuarios regresan a la plataforma después de registrarse (retención por cohortes)?
- ¿El cambio en la UI del checkout mejora significativamente la tasa de conversión?

## Datos

- **Fuente:** Datasets del negocio (`rappiplus_orders_raw.csv`, `rappiplus_catalog.csv`, `rappiplus_marketing_spend.csv`, `experiment_checkout_ui.csv`) y tablas SQL (`events`, `users`, `user_activity`) de una base de datos PostgreSQL
- **Periodo:** Datos de pedidos y marketing de 2025 (enero–junio); actividad de usuario por cohortes mensuales de 2025
- **Tamaño:**
  - `orders` — 25,100 pedidos (25,000 tras limpieza de duplicados y nulos)
  - `catalog` — catálogo de productos con costo unitario, categoría y proveedor
  - `marketing` — gasto diario por país y canal
  - `experiment_checkout_ui` — 10,000 usuarios (4,965 control / 5,035 tratamiento)
  - `events`, `users`, `user_activity` — comportamiento y retención vía consultas SQL
- **Variables principales:** `id_pedido`, `id_usuario`, `fecha_hora_pedido`, `pais`, `dispositivo`, `nombre_producto`, `categoria_producto`, `cantidad`, `precio_unitario`, `monto_descuento`, `monto_total`, `costo_unitario`, `canal`, `gasto`, `nombre_evento`, `variante`, `convirtio`

## Herramientas

- Tableau Public
- Python (pandas, numpy, seaborn, matplotlib, scipy, statsmodels)
- SQL (PostgreSQL, vía SQLAlchemy) para funnel y retención por cohortes

## Contenido del dashboard

- **Rentabilidad del negocio:** revenue, COGS, gasto de marketing y profit neto del periodo
- **Comportamiento de ventas:** ticket promedio por orden, unidades por orden y producto más vendido
- **Funnel de conversión:** usuarios únicos por etapa (`add_to_cart` → `begin_checkout` → `purchase`) y tasa de conversión entre pasos
- **Retención por cohortes:** porcentaje de usuarios activos por semana desde su registro, segmentado por cohorte mensual
- **Resultado del test A/B:** comparación de tasa de conversión entre la UI actual y la nueva UI del checkout
- **Filtros interactivos:** país, canal de marketing, categoría de producto
- **Indicadores clave (KPI):** Revenue Total, COGS Total, Ganancia Bruta, Gasto en Marketing, Profit, Margen Neto (%)

## Principales conclusiones

- El negocio es rentable: **Revenue Total de $51,965,834.26**, **COGS de $43,124,018.41**, **Ganancia Bruta de $8,841,815.85**, **Gasto en Marketing de $2,871,843.53**, resultando en un **Profit de $5,969,972.32** y un **Margen Neto de 11.49%**.
- El **ticket promedio por orden es de $2,082.80**, con una mediana de 2 unidades por orden (promedio de 7.12 unidades, sesgado por órdenes grandes).
- **Laptop-Gaming-16GB** es el producto más vendido tanto en unidades (144,198) como en ingresos ($43,429,372.98); **Blender-XL-Red** es el producto con más órdenes distintas (4,176).
- La inversión en marketing está distribuida de forma casi equitativa entre canales: **social (34.1%)**, **organic (33.9%)** y **paid_search (32.0%)**.
- En el funnel de conversión, la mayor caída ocurre entre `begin_checkout` y `purchase`, con una tasa de conversión final acumulada de **81.74%** desde `add_to_cart`.
- El test A/B del checkout (Prueba Z de dos proporciones) no mostró diferencia significativa: tasa de conversión de **15.69% (control)** vs. **16.29% (tratamiento)**, con **Z = -0.8133** y **p-value = 0.416** (> 0.05). No se rechaza la hipótesis nula: no hay evidencia suficiente de que la nueva UI mejore la conversión.

## Aprendizajes

- Validación y limpieza de datos de negocio: detección de duplicados, inconsistencias entre montos calculados vs. registrados, y tratamiento de valores nulos en variables categóricas y numéricas.
- Cálculo de KPIs financieros (revenue, COGS, ganancia bruta, profit, margen neto) integrando múltiples fuentes de datos (pedidos, catálogo, marketing).
- Construcción de un **funnel de conversión** y análisis de **retención por cohortes** usando SQL avanzado (CTEs, funciones de ventana como `LAG` y `FIRST_VALUE`).
- Diseño y evaluación de un **test A/B** con prueba estadística de proporciones (Z-test), interpretando correctamente el resultado en términos de significancia estadística y no solo de diferencia numérica.
- Storytelling con datos: traducir hallazgos técnicos en un dashboard de Tableau orientado a la toma de decisiones de negocio.


## Contacto

- LinkedIn: (https://www.linkedin.com/in/emma-solorzano-hernandez-jauregui-200301345/)
- Perfil de Tableau Public: https://public.tableau.com/views/Project_17885024557020/Dashboard2?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link
