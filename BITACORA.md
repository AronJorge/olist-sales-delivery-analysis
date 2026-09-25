# Plan y bitácora del proyecto

## Objetivo

Completar la entrega obligatoria del pipeline ELT de Olist: extracción, carga en SQLite, siete consultas SQL, dos transformaciones con Pandas, gráficos y notebook ejecutado con las respuestas de análisis. Apache Airflow queda excluido por decisión del usuario.

## Reglas de trabajo

- Mantener la estructura, las firmas de funciones y los contratos existentes de resultados.
- Mantener las versiones de dependencias de la tarea y usar Python 3.11.
- No dejar comentarios en el código entregado ni usar iconos. Registrar explicaciones y decisiones en este documento y en las celdas Markdown correspondientes del notebook.
- Al completar las funciones y consultas, retirar sus comentarios de plantilla y marcadores TODO. No modificar los tests originales para acomodar resultados.
- Actualizar esta bitácora con cada etapa: cambios, validaciones, resultados y pendientes. Distinguir siempre lo realizado de lo previsto.
- Usar los JSON de referencia únicamente para validar resultados, nunca como datos del pipeline.
- Conservar los cambios del usuario; revisar el estado de los archivos antes de editarlos.

## Plan de implementación

### 1. Entorno y datos

Verificar las dependencias del entorno `.venv-py311` e instalar las que falten respetando `requirements.txt`. Usar los nueve CSV en `dataset/`. Comprobar la respuesta real de Nager.Date para Brasil en 2017: los tests esperan 14 registros y siete columnas después de limpiar la respuesta. Informar cualquier diferencia externa sin fabricar datos ni alterar expectativas.

### 2. Extracción y carga

Completar la petición de festivos con Requests, un tiempo de espera explícito y validación del estado HTTP. Convertir errores de petición en SystemExit, eliminar las columnas types y counties y convertir date a datetime64[ns]. Conservar la lectura de CSV y el diccionario de tablas existente.

Cargar los DataFrames con to_sql, sin índice y reemplazando tablas existentes, para permitir ejecuciones repetidas sin duplicación.

### 3. Consultas SQL

| Consulta | Implementación prevista |
| --- | --- |
| Estados de pedidos | Contar todos los pedidos por estado y ordenar por estado. |
| Ingresos por mes y año | Filtrar pedidos entregados con fecha real presente, usar el pago mínimo por pedido según la instrucción particular de la tarea y devolver doce meses para 2016–2018, con cero donde no haya ingresos. |
| Ingresos por estado | Sumar los pagos de pedidos entregados con fecha real presente y seleccionar los diez estados con mayores ingresos. |
| Categorías con mayores y menores ingresos | Traducir categorías al inglés, excluir categorías nulas y pedidos sin entrega válida, contar pedidos únicos y atribuir el pago completo del pedido a cada categoría que contiene, evitando duplicaciones por artículos repetidos. |
| Diferencia de entrega por estado | Calcular fecha estimada menos fecha real sin hora, promediar por estado y convertir a entero según la referencia. |
| Tiempo real y estimado | Calcular intervalos desde la compra para pedidos únicos entregados con fecha real presente, agrupando por mes y año de compra; conservar nulos donde no existan observaciones. |

Respetar nombres y orden de columnas, ordenar series por mes y rankings por ingresos. Resolver empates de manera determinista y comprobar la ordenación contra las referencias.

### 4. Transformaciones con Pandas

Unir pedidos, artículos y productos, filtrar pedidos entregados y sumar freight_value y product_weight_g por order_id. Devolver una fila por pedido, ordenada por identificador, con las columnas esperadas.

Para pedidos diarios, convertir fechas de compra, filtrar 2017, contar por día y marcar fechas festivas. Devolver order_count, date y holiday en orden cronológico, sin agregar días sin pedidos ausentes de la referencia.

### 5. Visualizaciones y notebook

Completar la dispersión de peso en gramos frente a valor de flete y la serie diaria de pedidos con líneas verticales para festivos. Seguir las imágenes de referencia y revisar títulos, unidades, leyendas y legibilidad.

Ejecutar el notebook en su orden original con Python 3.11. Completar las respuestas 4.1 y 4.2 en español a partir de resultados reales: comparar pedidos en festivos y días ordinarios sin afirmar causalidad y calcular la correlación de Pearson entre peso y flete como apoyo a la interpretación del gráfico.

Guardar el notebook ejecutado con tablas, gráficos y respuestas. Añadir instrucciones breves de reproducción con uv al README.

### 6. Validación

- Ejecutar los tests originales de extracción y transformación sin alterar sus expectativas.
- Comparar las nueve salidas con las referencias, incluyendo columnas, orden, fechas, nulos y tolerancias numéricas establecidas.
- Añadir pruebas puntuales de fallo HTTP, carga repetida y pedidos con varios artículos, pagos o categorías.
- Ejecutar el notebook desde un kernel limpio y revisar visualmente los gráficos.
- Verificar que una segunda carga conserva los conteos de tablas.
- Aplicar Black con longitud de línea 88 al código de implementación y comprobar que no queden comentarios ni marcadores pendientes en el código entregado.

## Criterios de finalización

La entrega estará completa cuando pasen los tests originales y las comprobaciones adicionales, las nueve salidas coincidan con las referencias, el notebook se ejecute sin errores desde un kernel limpio y estén guardadas sus visualizaciones y respuestas. Cualquier limitación externa pendiente se documentará explícitamente; no se marcarán como realizadas verificaciones que no se hayan ejecutado.

## Registro de actividad

### 2026-09-25 — Diagnóstico del entorno

Se revisaron requirements.txt y las instrucciones. El error compartido durante la instalación correspondía a un intento de compilar pandas 2.0.3 con Python 3.14. Se indicó utilizar Python 3.11. Posteriormente se comprobó que el entorno `.venv-py311` ejecuta Python 3.11.15 y puede importar Pandas y leer los CSV. Todavía no se ha auditado la totalidad de las dependencias instaladas.

### 2026-09-25 — Inspección y planificación

Se leyeron README.md, ASSIGNMENT.md, los módulos Python, las siete consultas, los tests y las celdas del notebook. Se inspeccionaron las dimensiones y muestras de los JSON de referencia. Se identificaron dos respuestas de análisis pendientes en el notebook y la regla especial del pago mínimo para ingresos mensuales. El usuario eligió completar solo la parte obligatoria, sin Airflow. Se presentó el plan de implementación.

### 2026-09-25 — Verificación del dataset descargado por el usuario

Se confirmó que los nueve CSV están en dataset/ y se leyeron con Pandas. Sus dimensiones coinciden con las expectativas de los tests:

| Tabla | Filas | Columnas |
| --- | ---: | ---: |
| olist_customers | 99441 | 5 |
| olist_geolocation | 1000163 | 5 |
| olist_order_items | 112650 | 7 |
| olist_order_payments | 103886 | 5 |
| olist_order_reviews | 99224 | 7 |
| olist_orders | 99441 | 8 |
| olist_products | 32951 | 9 |
| olist_sellers | 3095 | 4 |
| product_category_name_translation | 71 | 2 |

Esta verificación confirma lectura y dimensiones; no constituye una validación completa del contenido ni de las transformaciones.

### 2026-09-25 — Creación de la bitácora

Se creó este documento a solicitud del usuario para conservar el plan y registrar el avance. Se incorporó la restricción de entregar código sin comentarios y no usar iconos. En esta etapa el asistente no ha implementado el pipeline ni ejecutado los tests o el notebook.

## Estado actual

Planificación y verificación de dimensiones del dataset completadas. Pendientes: comprobación de dependencias y API, implementación, pruebas, ejecución del notebook, respuestas de análisis y documentación de reproducción.
