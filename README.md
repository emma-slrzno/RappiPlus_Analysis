# RappiPlus: de datos a decisiones de negocio

Proyecto final de análisis de datos que evalúa el desempeño del servicio **RappiPlus** de punta a punta: calidad de datos, rentabilidad, embudo de conversión, retención de usuarios y validación estadística de un experimento A/B, con los resultados comunicados en un dashboard de BI.

🔗 **Dashboard interactivo (Tableau Public):** [Ver dashboard](https://public.tableau.com/views/Project_17885024557020/Dashboard2?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

---

## 1. Problema o contexto de negocio

RappiPlus opera en tres países (**México, Colombia y Argentina**) y vende productos de las categorías Electrónica, Hogar y Moda. Para decidir dónde invertir, qué corregir y qué cambios lanzar, la dirección necesita responder preguntas como:

- ¿Podemos confiar en los datos que estamos usando para reportar?
- ¿El negocio es realmente rentable después de descontar costos de producto y marketing?
- ¿En qué punto del proceso de compra se nos van los usuarios?
- ¿Los usuarios regresan después de registrarse?
- ¿El nuevo diseño del checkout mejora la conversión, o la diferencia observada es ruido?

## 2. Objetivo del análisis

Convertir datos operativos y de comportamiento en **decisiones accionables**, siguiendo una lógica progresiva:

| # | Pregunta | Enfoque |
|---|----------|---------|
| 1 | ¿Podemos confiar en los datos? | Validación y limpieza en Python |
| 2 | ¿El negocio es rentable? | Revenue, COGS, marketing y profit |
| 3 | ¿Dónde se pierden los usuarios? | Funnel de conversión en SQL |
| 4 | ¿Los usuarios regresan? | Retención por cohortes en SQL |
| 5 | ¿Los cambios generan impacto? | Prueba Z de dos proporciones (A/B test) |
| 6 | ¿Cómo comunicamos los resultados? | Dashboard en Tableau |

## 3. Dataset utilizado

| Fuente | Contenido | Origen |
|--------|-----------|--------|
| `rappiplus_orders_raw.csv` | Pedidos: usuario, fecha, país, dispositivo, fuente de referencia, producto, cantidad, precio, descuento y monto total (25,100 filas en crudo) | CSV |
| `rappiplus_catalog.csv` | Costo unitario, categoría y proveedor por producto | CSV |
| `rappiplus_marketing_spend.csv` | Gasto diario en marketing por país y canal (1,620 filas) | CSV |
| `events` | Eventos de navegación por usuario (embudo de conversión) | PostgreSQL |
| `users` / `user_activity` | Registro de usuarios y actividad posterior (retención) | PostgreSQL |
| `experiment_checkout_ui.csv` | Experimento A/B del checkout: control vs. tratamiento (10,000 usuarios) | CSV |

**Periodo cubierto:** enero a junio de 2025.

## 4. Herramientas y tecnologías

- **Python:** `pandas`, `numpy`, `matplotlib`, `seaborn`
- **Estadística:** `statsmodels` (`proportions_ztest`), `scipy`
- **SQL (PostgreSQL):** CTEs, funciones de ventana (`LAG`, `FIRST_VALUE`), `DATE_TRUNC`, `EXTRACT`
- **Conexión a base de datos:** `SQLAlchemy`
- **Entorno:** Jupyter Notebook
- **BI:** Tableau Public

## 5. Proceso realizado

### Paso 1 · Calidad de datos
- Conversión de fechas a `datetime`.
- Eliminación de **100 pedidos duplicados** por `id_pedido` (25,100 → 25,000).
- Eliminación de filas sin `cantidad` o `precio_unitario` (quedan **24,950** pedidos).
- Validación de consistencia de montos: se contrastó `precio × cantidad − descuento` contra `monto_total`. Coinciden 23,805 de 25,000 pedidos; se recalculó el revenue con la fórmula consistente y la diferencia total fue mínima (~$2.3K sobre ~$52M).
- Revisión de nulos: `pais` (1.2%), `dispositivo`, `fuente_referencia`, `nombre_producto` y `categoria_producto` (≤0.12%).
- Exportación de los datasets limpios (`orders_clean.csv`, `catalog_clean.csv`, `marketing_clean.csv`) para el dashboard.

### Paso 2 · Rentabilidad (KPIs)
- Cruce de pedidos con el catálogo para calcular **COGS** por pedido.
- Cálculo de revenue, ganancia bruta, gasto de marketing, profit y margen neto.
- Ticket promedio, cantidad promedio por orden, producto más vendido y gasto por canal.

### Paso 3 · Funnel de conversión (SQL)
- Usuarios únicos por evento sobre la tabla `events`.
- Tasas de conversión paso a paso y global, y porcentaje de abandono (*drop-off*) con funciones de ventana.

### Paso 4 · Retención por cohortes (SQL)
- Cohorte definida por **mes de registro** (`users`).
- Cruce con `user_activity` para medir cuántos usuarios de cada cohorte siguen activos en cada periodo.

### Paso 5 · Experimento A/B
- Métrica principal: `convirtio` (1 = completó la compra).
- **H₀:** no hay diferencia en la tasa de conversión entre la UI actual y la nueva.
- **H₁:** sí existe una diferencia significativa.
- Prueba Z de dos muestras para proporciones, con α = 0.05.

### Paso 6 · Dashboard
- Visualización de los KPIs y hallazgos en Tableau Public.

## 6. Principales hallazgos

### 💰 Rentabilidad

| Indicador | Valor |
|-----------|------:|
| Revenue total | $51,965,834 |
| COGS total | $43,124,018 |
| Ganancia bruta | $8,841,816 (17.0% del revenue) |
| Gasto en marketing | $2,871,844 (5.5% del revenue) |
| **Profit** | **$5,969,972** |
| **Margen neto** | **11.49%** |

- **El negocio es rentable**, pero con márgenes ajustados: el costo de producto absorbe ~83% de los ingresos.
- **Alta concentración en un solo producto:** *Laptop-Gaming-16GB* es el más vendido por unidades (144,198) y por ingresos (~$43.4M, cerca del **84% del revenue total**). Por número de órdenes, en cambio, lidera *Blender-XL-Red* (4,176).
- **Ticket promedio:** $2,082.80 por orden.
- **Cantidad por orden:** promedio de 7.12 unidades frente a una mediana de 2, lo que indica **pedidos de volumen muy alto** que inflan el promedio.
- **Marketing repartido casi parejo** entre canales: social (34.1%), orgánico (33.9%) y búsqueda pagada (32.0%).

### 🛒 Funnel de conversión

| Etapa | Usuarios únicos | Paso a paso | Global |
|-------|----------------:|------------:|-------:|
| `add_to_cart` | 7,634 | 100% | 100% |
| `begin_checkout` | 7,208 | 94.42% | 94.42% |
| `purchase` | 6,240 | 86.57% | 81.74% |

- La **mayor fuga ocurre entre iniciar el checkout y completar la compra (13.43% de abandono)**, es decir, justo en la etapa que el experimento intenta mejorar.
- Desde el carrito hasta la compra se conserva el **81.74%** de los usuarios.

### 🔁 Retención por cohortes

- Se analizaron las cohortes de enero a mayo de 2025 (entre 1,104 y 1,299 usuarios cada una).
- En todas las cohortes, la actividad del mes 1 supera a la del mes de registro (entre **114.9% y 130.8%**), lo que sugiere que los usuarios **sí regresan** e incluso se activan más después de su primer mes.

### 🧪 Experimento A/B del checkout

| Grupo | Usuarios | Conversiones | Tasa de conversión |
|-------|---------:|-------------:|-------------------:|
| Control | 4,965 | 779 | 15.69% |
| Tratamiento | 5,035 | 820 | 16.29% |

- Diferencia observada: **+0.6 puntos porcentuales**.
- Estadístico Z = −0.8133, **p-value = 0.416** (> 0.05).
- **No se rechaza H₀:** no hay evidencia estadística de que la nueva UI mejore la conversión; la diferencia es compatible con el azar.

## 7. Recomendaciones e impacto para el negocio

1. **Reducir la dependencia de un solo producto.** Con ~84% del revenue en las laptops gaming, cualquier problema de proveedor, precio o demanda golpea directamente los resultados. Conviene diversificar el catálogo y revisar los márgenes de las demás categorías.
2. **Proteger y mejorar el margen.** Con 17% de margen bruto, renegociar costos con proveedores o ajustar precios/descuentos tiene un impacto directo en el profit: cada punto de margen bruto equivale a ~$520K.
3. **Priorizar la etapa checkout → compra.** Es donde más usuarios se pierden (13.4%). Aun recuperando una fracción, el efecto sobre el revenue sería significativo.
4. **No lanzar la nueva UI del checkout solo por la mejora observada.** El resultado no es significativo. Se recomienda extender el experimento (más muestra o más tiempo) o probar cambios más ambiciosos antes de invertir en un rediseño completo.
5. **Medir el retorno por canal de marketing.** El gasto está repartido casi en partes iguales sin criterio de desempeño visible. Cruzar el gasto con conversiones e ingresos por canal permitiría reasignar presupuesto hacia los canales más eficientes.
6. **Capitalizar la retención.** Como los usuarios regresan, conviene reforzar campañas de reactivación y recompra durante el primer mes posterior al registro.

## 8. Limitaciones y puntos a revisar

Para interpretar los resultados con criterio, conviene tener en cuenta lo siguiente:

- **Funnel incompleto:** la consulta solo ordena los eventos `page_view`, `view_item`, `add_to_cart`, `begin_checkout` y `purchase`, pero en los datos aparecen otros nombres (`first_visit`, `select_item`, `add_payment_info`). Por eso el embudo arranca en `add_to_cart` y no incluye las etapas previas ni el pago. Ajustar los nombres de evento permitiría un embudo completo.
- **Retención sobre 100%:** que la actividad del mes 1 supere la del mes 0 indica que la métrica depende de cómo se registra la actividad (el mes 0 solo captura los días posteriores al registro). Sirve como señal de regreso, pero no es una retención clásica; se recomienda complementarla con retención semanal.
- **Pedidos de volumen extremo:** la diferencia entre media y mediana de unidades por orden sugiere valores atípicos que conviene auditar, porque afectan el revenue, el ticket promedio y el ranking de productos.
- **Gasto de marketing sin canal:** 101 registros de marketing no tienen `canal`, por lo que el desglose por canal (~$2.69M) no cubre el total invertido (~$2.87M).
- **Diferencias de monto:** ~4.8% de los pedidos no cuadra con la fórmula `precio × cantidad − descuento`; el impacto agregado es pequeño, pero vale la pena identificar su origen.
- **Costo sin producto:** los pedidos sin `nombre_producto` no cruzan con el catálogo, por lo que su COGS no se contabiliza.

## 9. Cómo reproducir el análisis

```bash
pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary statsmodels scipy jupyter
jupyter notebook S12_Proyecto_Final.ipynb
```

> ⚠️ **Seguridad:** el notebook incluye credenciales de la base de datos en texto plano. Antes de publicar el repositorio, elimínalas y cárgalas desde variables de entorno (por ejemplo, con un archivo `.env` fuera del control de versiones).

## 10. Estructura del proyecto

```
├── S12_Proyecto_Final.ipynb   # Análisis completo
├── orders_clean.csv           # Pedidos limpios
├── catalog_clean.csv          # Catálogo limpio
├── marketing_clean.csv        # Gasto de marketing limpio
└── README.md
```
