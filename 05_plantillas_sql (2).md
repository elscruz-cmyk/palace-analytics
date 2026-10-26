# Plantillas SQL (BigQuery, export de GA4)

Proyecto: `pr-mktg-analyt-prod`. Dataset de General Palace: `analytics_268229233`. Reglas: siempre `_TABLE_SUFFIX`, solo lectura salvo las plantillas 12 y 13 (solo en el dataset de pruebas, con aprobación), dry run antes de consultas grandes. Reemplaza los valores entre `< >`.

Costos de referencia (observados): un mes de General Palace con `event_params` ≈ 7 a 9 GiB; el histórico completo con 4 columnas angostas ≈ 33 GiB. Si una consulta devuelve `job_complete = false`, recupérala con `get_query_results` indicando la ubicación del job.

Desde la línea de comandos (`bq`), con dry run y tope de bytes:
`bq query --use_legacy_sql=false --location=us-central1 --dry_run < consulta.sql`
`bq query --use_legacy_sql=false --location=us-central1 --maximum_bytes_billed=50000000000 < consulta.sql`
Nota: en la sesión de trabajo anterior `bq` 2.1.32 falló con un error interno al hacer dry run; es POR VALIDAR si pasa en tu máquina. Alternativa: el conector MCP de BigQuery (`execute_sql_readonly` con `dryRun: true`).

Las ubicaciones se consultan por separado: `us-central1` (51 datasets) y `US` (10 datasets, todos de booking de Baglioni). No se pueden unir en una consulta.

---

## 1. Total users, sesiones y pings por mes (un sitio)

```sql
SELECT
  FORMAT_DATE('%Y-%m', PARSE_DATE('%Y%m%d', event_date)) AS mes,
  COUNT(DISTINCT user_pseudo_id) AS total_users_bq,
  COUNT(DISTINCT IF(is_active_user, user_pseudo_id, NULL)) AS active_users_bq,
  COUNT(DISTINCT CONCAT(user_pseudo_id,
    CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING))) AS sesiones_con_id,
  COUNTIF(user_pseudo_id IS NULL) AS eventos_sin_id,
  COUNTIF(event_name = 'session_start' AND user_pseudo_id IS NULL) AS pings_session_start_sin_id
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
WHERE _TABLE_SUFFIX BETWEEN '20260101' AND '<ULTIMO_DIA_MES_COMPLETO>'
  AND device.web_info.hostname = '<hostname.exacto.com>'
GROUP BY mes
ORDER BY mes
```

Usa el hostname EXACTO (equivale al regex anclado de Looker). Compara contra Looker con la dimensión nativa `Month`.

---

## 2. Total users de la propiedad y por sitio (mapeo del CASE de Looker)

```sql
WITH base AS (
  SELECT
    user_pseudo_id,
    is_active_user,
    CASE
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^beach\.palaceresorts\.com$') THEN 'Beach Palace'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^cozumel\.palaceresorts\.com$') THEN 'Cozumel Palace'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^playacar\.palaceresorts\.com$') THEN 'Playacar Palace'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^sun\.palaceresorts\.com$') THEN 'Sun Palace'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^nizuc\.moonpalacecancun\.com$') THEN 'Moon Palace Nizuc'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^thegrandpuntacana\.moonpalace\.com$') THEN 'Moon Palace Punta Cana'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^jamaica\.moonpalace\.com$') THEN 'Moon Palace Jamaica'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^thegrand\.moonpalace\.com$') THEN 'Moon Palace The Grand'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^cancun\.leblancsparesorts\.com$') THEN 'Le Blanc Cancun'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^los-cabos\.leblancsparesorts\.com$') THEN 'Le Blanc Los Cabos'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^(www\.)?palaceresorts\.com$') THEN 'Palace Resorts Brand'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^(www\.)?leblancsparesorts\.com$') THEN 'Le Blanc Brand'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^(www\.)?moonpalace\.com$') THEN 'Moon Palace Brand'
      WHEN REGEXP_CONTAINS(device.web_info.hostname, r'^(www\.)?moonpalacecancun\.com$') THEN 'Moon Palace Cancun'
      ELSE 'No properties'
    END AS sitio
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
)
SELECT
  IFNULL(sitio, '== TOTAL PROPIEDAD ==') AS sitio,
  COUNT(DISTINCT user_pseudo_id) AS total_users,
  COUNT(DISTINCT IF(is_active_user, user_pseudo_id, NULL)) AS active_users,
  COUNT(DISTINCT IF(sitio != 'No properties', user_pseudo_id, NULL)) AS users_filtro_solo_propiedades
FROM base
GROUP BY GROUPING SETS ((), (sitio))
ORDER BY total_users DESC
```

---

## 3. Embudo por hostname (usuarios distintos en el periodo)

```sql
SELECT
  device.web_info.hostname AS hostname,
  COUNT(DISTINCT user_pseudo_id) AS users,
  COUNT(DISTINCT IF(event_name = 'view_item_list', user_pseudo_id, NULL)) AS u_view_item_list,
  COUNT(DISTINCT IF(event_name = 'click_to_booking', user_pseudo_id, NULL)) AS u_click_to_booking,
  COUNT(DISTINCT IF(event_name = 'add_to_cart', user_pseudo_id, NULL)) AS u_add_to_cart,
  COUNT(DISTINCT IF(event_name = 'begin_checkout', user_pseudo_id, NULL)) AS u_begin_checkout,
  COUNT(DISTINCT IF(event_name = 'purchase', user_pseudo_id, NULL)) AS u_purchase,
  COUNTIF(event_name = 'purchase') AS purchase_events
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
GROUP BY hostname
ORDER BY users DESC
LIMIT 20
```

No sumar estas cifras entre hostnames. Si agregas `purchase_revenue`, verifica primero la moneda.

---

## 4. Calidad del evento purchase

```sql
SELECT
  COUNT(*) AS purchase_events,
  COUNTIF(user_id IS NOT NULL) AS con_user_id,
  COUNTIF(user_pseudo_id IS NOT NULL) AS con_user_pseudo_id,
  COUNTIF(ecommerce.transaction_id IS NOT NULL AND ecommerce.transaction_id != '(not set)') AS con_transaction_id,
  COUNT(DISTINCT ecommerce.transaction_id) AS transaction_ids_distintos,
  COUNT(DISTINCT user_pseudo_id) AS usuarios_pseudo_distintos,
  COUNTIF(privacy_info.analytics_storage = 'No') AS analytics_storage_no
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
  AND event_name = 'purchase'
```

Parámetros del evento (solo nombres y tipos, sin valores):

```sql
SELECT
  ep.key AS parametro,
  COUNT(*) AS eventos_con_parametro,
  COUNTIF(ep.value.string_value IS NOT NULL) AS tipo_string,
  COUNTIF(ep.value.int_value IS NOT NULL) AS tipo_int,
  COUNTIF(ep.value.double_value IS NOT NULL OR ep.value.float_value IS NOT NULL) AS tipo_decimal
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`, UNNEST(event_params) AS ep
WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
  AND event_name = 'purchase'
GROUP BY parametro
ORDER BY eventos_con_parametro DESC
```

---

## 5. Historial de actividad de un comprador (por fecha y hostname), a partir de una reserva

No muestra ningún ID. Usa el rango más amplio que necesites (el histórico completo ≈ 33 GiB).

```sql
DECLARE tx STRING DEFAULT '<TRANSACTION_ID>';

WITH u AS (
  SELECT DISTINCT user_pseudo_id
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<FECHA_COMPRA>' AND '<FECHA_COMPRA>'
    AND event_name = 'purchase'
    AND ecommerce.transaction_id = tx
)
SELECT
  event_date,
  device.web_info.hostname AS hostname,
  COUNT(*) AS eventos,
  COUNTIF(event_name = 'first_visit') AS first_visit,
  COUNTIF(event_name = 'session_start') AS session_start,
  COUNTIF(event_name = 'purchase') AS purchase
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
WHERE _TABLE_SUFFIX BETWEEN '20230412' AND '<FIN>'
  AND user_pseudo_id IN (SELECT user_pseudo_id FROM u)
GROUP BY event_date, hostname
ORDER BY event_date, hostname
```

---

## 6. Recorrido detallado (agrupado por página) de varias reservas

Etiqueta a los usuarios A, B, C... y no muestra IDs. Hora en la zona de la propiedad (UTC−6; confirmar el nombre de la zona en GA4 Admin). Excluye eventos de ruido y agrupa eventos consecutivos de la misma página y sesión.

```sql
DECLARE tx_list ARRAY<STRING> DEFAULT ['<TX1>', '<TX2>', '<TX3>'];

WITH elegidos AS (
  SELECT user_pseudo_id,
         ROW_NUMBER() OVER (ORDER BY MIN(event_timestamp)) AS n
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<FECHA_COMPRA_MIN>' AND '<FECHA_COMPRA_MAX>'
    AND event_name = 'purchase'
    AND ecommerce.transaction_id IN UNNEST(tx_list)
  GROUP BY user_pseudo_id
),
ev AS (
  SELECT
    CHR(64 + e.n) AS usuario,
    x.event_date,
    x.event_timestamp,
    x.event_name,
    x.device.web_info.hostname AS hostname,
    (SELECT value.int_value FROM UNNEST(x.event_params) WHERE key = 'ga_session_id') AS sid,
    (SELECT value.int_value FROM UNNEST(x.event_params) WHERE key = 'batch_ordering_id') AS bo,
    REGEXP_EXTRACT(
      (SELECT value.string_value FROM UNNEST(x.event_params) WHERE key = 'page_location'),
      r'^https?://[^/]+(/[^?#]*)') AS path
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*` AS x
  JOIN elegidos AS e USING (user_pseudo_id)
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO_VENTANA>' AND '<FIN_VENTANA>'
    AND x.event_name NOT IN ('user_engagement', 'scroll', 'all_page_scroll')
),
marcado AS (
  SELECT *,
    IF(hostname = LAG(hostname) OVER w
       AND IFNULL(path, '') = IFNULL(LAG(path) OVER w, '')
       AND sid = LAG(sid) OVER w, 0, 1) AS nuevo
  FROM ev
  WINDOW w AS (PARTITION BY usuario ORDER BY event_timestamp, bo)
),
grupos AS (
  SELECT *,
    SUM(nuevo) OVER (PARTITION BY usuario ORDER BY event_timestamp, bo ROWS UNBOUNDED PRECEDING) AS g
  FROM marcado
),
ses AS (
  SELECT usuario, sid, ROW_NUMBER() OVER (PARTITION BY usuario ORDER BY sid) AS ses_n
  FROM (SELECT DISTINCT usuario, sid FROM ev)
),
conteo AS (
  SELECT usuario, g, event_name, COUNT(*) AS n_ev, MIN(event_timestamp) AS t0,
         ANY_VALUE(hostname) AS hostname, ANY_VALUE(path) AS path,
         ANY_VALUE(sid) AS sid, ANY_VALUE(event_date) AS event_date
  FROM grupos
  GROUP BY usuario, g, event_name
)
SELECT
  c.usuario,
  MIN(c.event_date) AS fecha,
  FORMAT_TIMESTAMP('%H:%M:%S', TIMESTAMP_MICROS(MIN(c.t0)), '-06:00') AS hora_local,
  s.ses_n AS sesion,
  ANY_VALUE(c.hostname) AS hostname,
  ANY_VALUE(c.path) AS ruta,
  STRING_AGG(CONCAT(c.event_name, IF(c.n_ev > 1, CONCAT('x', CAST(c.n_ev AS STRING)), '')), ', '
             ORDER BY c.t0, c.event_name) AS eventos
FROM conteo AS c
JOIN ses AS s ON c.usuario = s.usuario AND c.sid IS NOT DISTINCT FROM s.sid
GROUP BY c.usuario, c.g, s.ses_n
ORDER BY c.usuario, MIN(c.t0)
```

Ventana: empieza en la primera visita que encontró la consulta 5 y termina unos días después de la última actividad. Busca por reserva (`transaction_id`), no por `user_pseudo_id`, por privacidad.

---

## 7. Inventario de tablas por dataset y zona horaria

```sql
SELECT
  CASE
    WHEN table_name LIKE 'events_intraday_%' THEN 'events_intraday'
    WHEN table_name LIKE 'events_%' THEN 'events'
    WHEN table_name LIKE 'pseudonymous_users_%' THEN 'pseudonymous_users'
    WHEN table_name LIKE 'users_%' THEN 'users'
    ELSE 'other'
  END AS tipo,
  COUNT(*) AS tablas, MIN(table_name) AS primera, MAX(table_name) AS ultima
FROM `pr-mktg-analyt-prod.analytics_268229233.INFORMATION_SCHEMA.TABLES`
GROUP BY tipo
ORDER BY tipo
```

Zona horaria de la propiedad (min/max UTC de un día de tabla; si empieza a las 06:00 UTC, es UTC−6):

```sql
SELECT
  FORMAT_TIMESTAMP('%Y-%m-%d %H:%M', TIMESTAMP_MICROS(MIN(event_timestamp))) AS min_utc,
  FORMAT_TIMESTAMP('%Y-%m-%d %H:%M', TIMESTAMP_MICROS(MAX(event_timestamp))) AS max_utc,
  MIN(event_date) AS event_date
FROM `pr-mktg-analyt-prod.analytics_268229233.events_20260915`
```

---

# Plantillas nuevas para la unificación

## 8. Inventario de TODOS los datasets GA4 de una ubicación (metadatos, sin costo de datos)

Una consulta por ubicación (no se pueden unir). Cambia `region-us-central1` por `region-us` para los 10 datasets de `US`. Excluye los `_temp`.

```sql
WITH a AS (
  SELECT table_schema AS ds,
    COUNTIF(REGEXP_CONTAINS(table_name, r'^events_\d{8}$')) AS ev,
    COUNTIF(STARTS_WITH(table_name, 'pseudonymous_users_')) AS ps,
    COUNTIF(STARTS_WITH(table_name, 'users_')) AS us,
    PARSE_DATE('%Y%m%d', MIN(IF(REGEXP_CONTAINS(table_name, r'^events_\d{8}$'), SUBSTR(table_name, 8), NULL))) AS d0,
    PARSE_DATE('%Y%m%d', MAX(IF(REGEXP_CONTAINS(table_name, r'^events_\d{8}$'), SUBSTR(table_name, 8), NULL))) AS d1,
    ROUND(SUM(total_logical_bytes) / POW(1024, 3), 1) AS gib
  FROM `pr-mktg-analyt-prod`.`region-us-central1`.INFORMATION_SCHEMA.TABLE_STORAGE
  WHERE STARTS_WITH(table_schema, 'analytics_') AND NOT ENDS_WITH(table_schema, '_temp')
  GROUP BY ds
)
SELECT ds, ev AS tablas_events, d0 AS primera, d1 AS ultima,
  DATE_DIFF(d1, d0, DAY) + 1 - ev AS dias_faltantes,
  ps AS pseudonymous, us AS users_tablas, gib
FROM a
ORDER BY ds
```

## 9. Hostnames que recibe un dataset (muestra de un día)

Usa una tabla que exista (verifica con `list_table_ids`: algunos datasets tienen huecos o están detenidos; la última tabla siempre existe). Para revisar varios datasets, genera el `UNION ALL` por código.

```sql
SELECT
  device.web_info.hostname AS hostname,
  platform,
  COUNT(*) AS eventos
FROM `pr-mktg-analyt-prod.analytics_<PROPERTY_ID>.events_<AAAAMMDD>`
GROUP BY hostname, platform
ORDER BY eventos DESC
```

## 10. Tarea 1: diferencia por mes y hostname entre booking de General Palace (GP) y General Booking (GB)

Ambas están en `us-central1`. Brecha = 1 − GB / GP (declara cuál es la referencia y no la cambies). Haz dry run por mes antes de abrir el rango.

```sql
WITH gp AS (
  SELECT
    FORMAT_DATE('%Y-%m', PARSE_DATE('%Y%m%d', event_date)) AS mes,
    device.web_info.hostname AS hostname,
    COUNT(*) AS eventos,
    COUNTIF(user_pseudo_id IS NULL) AS eventos_sin_id,
    COUNT(DISTINCT user_pseudo_id) AS usuarios,
    COUNT(DISTINCT CONCAT(user_pseudo_id,
      CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING))) AS sesiones,
    COUNTIF(event_name = 'view_item_list') AS view_item_list,
    COUNTIF(event_name = 'begin_checkout') AS begin_checkout,
    COUNTIF(event_name = 'purchase') AS purchase_eventos,
    COUNT(DISTINCT IF(event_name = 'purchase', ecommerce.transaction_id, NULL)) AS transacciones
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
    AND REGEXP_CONTAINS(device.web_info.hostname, r'^(bookings|onlinebookings)')
  GROUP BY mes, hostname
),
gb AS (
  SELECT
    FORMAT_DATE('%Y-%m', PARSE_DATE('%Y%m%d', event_date)) AS mes,
    device.web_info.hostname AS hostname,
    COUNT(*) AS eventos,
    COUNTIF(user_pseudo_id IS NULL) AS eventos_sin_id,
    COUNT(DISTINCT user_pseudo_id) AS usuarios,
    COUNT(DISTINCT CONCAT(user_pseudo_id,
      CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING))) AS sesiones,
    COUNTIF(event_name = 'view_item_list') AS view_item_list,
    COUNTIF(event_name = 'begin_checkout') AS begin_checkout,
    COUNTIF(event_name = 'purchase') AS purchase_eventos,
    COUNT(DISTINCT IF(event_name = 'purchase', ecommerce.transaction_id, NULL)) AS transacciones
  FROM `pr-mktg-analyt-prod.analytics_324768211.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
    AND REGEXP_CONTAINS(device.web_info.hostname, r'^(bookings|onlinebookings)')
  GROUP BY mes, hostname
)
SELECT
  COALESCE(gp.mes, gb.mes) AS mes,
  COALESCE(gp.hostname, gb.hostname) AS hostname,
  gp.eventos AS eventos_gp, gb.eventos AS eventos_gb,
  ROUND(100 * (1 - SAFE_DIVIDE(gb.eventos, gp.eventos)), 2) AS brecha_eventos_pct,
  gp.usuarios AS usuarios_gp, gb.usuarios AS usuarios_gb,
  ROUND(100 * (1 - SAFE_DIVIDE(gb.usuarios, gp.usuarios)), 2) AS brecha_usuarios_pct,
  gp.sesiones AS sesiones_gp, gb.sesiones AS sesiones_gb,
  ROUND(100 * (1 - SAFE_DIVIDE(gb.sesiones, gp.sesiones)), 2) AS brecha_sesiones_pct,
  gp.view_item_list AS vil_gp, gb.view_item_list AS vil_gb,
  gp.begin_checkout AS bc_gp, gb.begin_checkout AS bc_gb,
  gp.transacciones AS tx_gp, gb.transacciones AS tx_gb,
  ROUND(100 * (1 - SAFE_DIVIDE(gb.transacciones, gp.transacciones)), 2) AS brecha_tx_pct,
  gp.eventos_sin_id AS sin_id_gp, gb.eventos_sin_id AS sin_id_gb
FROM gp
FULL OUTER JOIN gb ON gp.mes = gb.mes AND gp.hostname = gb.hostname
ORDER BY mes, hostname
```

Lectura: las filas donde solo hay un lado (`NULL` en GP o en GB) son hostnames que una propiedad no recibe. Los usuarios y sesiones de esta tabla son por mes y hostname: no se suman entre filas.

Costo medido (dry run, 26-oct-2026): 1 día (1-oct-2026) de GP + GB = 176 MB escaneados. Extrapolación lineal (INFERENCIA) para ene–sep 2026 (273 días) ≈ 45–50 GB: haz dry run con el rango real y con un mes antes de abrirlo.

## 11. Tarea 1: cruce de compras por `transaction_id` entre GP y GB

```sql
WITH gp AS (
  SELECT DISTINCT ecommerce.transaction_id AS tx
  FROM `pr-mktg-analyt-prod.analytics_268229233.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
    AND event_name = 'purchase'
    AND ecommerce.transaction_id IS NOT NULL AND ecommerce.transaction_id != '(not set)'
    AND REGEXP_CONTAINS(device.web_info.hostname, r'^(bookings|onlinebookings)')
),
gb AS (
  SELECT DISTINCT ecommerce.transaction_id AS tx
  FROM `pr-mktg-analyt-prod.analytics_324768211.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
    AND event_name = 'purchase'
    AND ecommerce.transaction_id IS NOT NULL AND ecommerce.transaction_id != '(not set)'
    AND REGEXP_CONTAINS(device.web_info.hostname, r'^(bookings|onlinebookings)')
)
SELECT
  COUNT(*) AS transacciones_distintas_total,
  COUNTIF(gp.tx IS NOT NULL AND gb.tx IS NOT NULL) AS en_ambas,
  COUNTIF(gb.tx IS NULL) AS solo_general_palace,
  COUNTIF(gp.tx IS NULL) AS solo_general_booking
FROM gp
FULL OUTER JOIN gb ON gp.tx = gb.tx
```

## 12. Pruebas de unificación: mapa hostname → fuente y tabla unificada de prueba (ESCRIBE: solo con aprobación y solo en el dataset de pruebas)

Antes: confirmar con la usuaria el nombre del dataset de pruebas (`<SANDBOX>`, en `us-central1`) y mostrar el costo estimado con dry run.

```sql
-- 12a. Mapa: UNA fuente por hostname (se llena con la decisión de la Tarea 1)
CREATE TABLE IF NOT EXISTS `pr-mktg-analyt-prod.<SANDBOX>.mapa_hostname_fuente` (
  hostname STRING,
  dataset_fuente STRING,   -- property ID de la fuente elegida, por ejemplo '268229233'
  sitio STRING,
  tipo STRING,             -- contenido, booking, segmento...
  nota STRING
);
```

```sql
-- 12b. Prueba pequeña: un mes y dos fuentes (genera el resto de los UNION ALL por código)
CREATE OR REPLACE TABLE `pr-mktg-analyt-prod.<SANDBOX>.eventos_unificados_prueba`
PARTITION BY fecha
CLUSTER BY origen, hostname AS
SELECT
  '268229233' AS origen,
  PARSE_DATE('%Y%m%d', e.event_date) AS fecha,
  e.device.web_info.hostname AS hostname,
  e.* REPLACE (
    ARRAY(SELECT AS STRUCT p.* FROM UNNEST(e.event_params) AS p
          WHERE p.key NOT IN ('hash_email', 'user_login')) AS event_params
  )
FROM `pr-mktg-analyt-prod.analytics_268229233.events_*` AS e
JOIN `pr-mktg-analyt-prod.<SANDBOX>.mapa_hostname_fuente` AS m
  ON m.hostname = e.device.web_info.hostname AND m.dataset_fuente = '268229233'
WHERE e._TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'

UNION ALL

SELECT
  '<OTRA_PROPERTY_ID>' AS origen,
  PARSE_DATE('%Y%m%d', e.event_date) AS fecha,
  e.device.web_info.hostname AS hostname,
  e.* REPLACE (
    ARRAY(SELECT AS STRUCT p.* FROM UNNEST(e.event_params) AS p
          WHERE p.key NOT IN ('hash_email', 'user_login')) AS event_params
  )
FROM `pr-mktg-analyt-prod.analytics_<OTRA_PROPERTY_ID>.events_*` AS e
JOIN `pr-mktg-analyt-prod.<SANDBOX>.mapa_hostname_fuente` AS m
  ON m.hostname = e.device.web_info.hostname AND m.dataset_fuente = '<OTRA_PROPERTY_ID>'
WHERE e._TABLE_SUFFIX BETWEEN '<INICIO>' AND '<FIN>'
```

Validación hecha (26-oct-2026): la sintaxis de `SELECT ... e.* REPLACE (...)` con el JOIN al mapa compila (dry run, solo el SELECT, sin crear la tabla). Escanea todas las columnas: 1 día de General Palace ≈ 480 MB, así que un mes ≈ 14 GB y el histórico completo es considerablemente mayor; el JOIN no reduce lo escaneado. Calcula el costo con el rango real antes de crear.

Notas (POR VALIDAR al ejecutarlo): la exclusión de `hash_email` y `user_login` es un paso de privacidad pendiente de legal; si un dataset no tiene hostname (`NULL`) esos eventos no entran por el JOIN, y conviene contarlos aparte; confirma el costo con dry run antes de crear la tabla.

## 13. Reconciliación: unificado vs dataset de origen (mismo alcance: origen + hostname + día)

No se comparan totales sumados entre alcances (usuarios y sesiones no son aditivos). Una consulta por origen.

```sql
WITH u AS (
  SELECT
    fecha, hostname,
    COUNT(*) AS eventos,
    COUNT(DISTINCT user_pseudo_id) AS usuarios,
    COUNT(DISTINCT CONCAT(user_pseudo_id,
      CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING))) AS sesiones,
    COUNTIF(event_name = 'purchase') AS compras
  FROM `pr-mktg-analyt-prod.<SANDBOX>.eventos_unificados_prueba`
  WHERE origen = '<PROPERTY_ID>' AND fecha BETWEEN DATE '<INICIO_AAAA-MM-DD>' AND DATE '<FIN_AAAA-MM-DD>'
  GROUP BY fecha, hostname
),
s AS (
  SELECT
    PARSE_DATE('%Y%m%d', event_date) AS fecha,
    device.web_info.hostname AS hostname,
    COUNT(*) AS eventos,
    COUNT(DISTINCT user_pseudo_id) AS usuarios,
    COUNT(DISTINCT CONCAT(user_pseudo_id,
      CAST((SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id') AS STRING))) AS sesiones,
    COUNTIF(event_name = 'purchase') AS compras
  FROM `pr-mktg-analyt-prod.analytics_<PROPERTY_ID>.events_*`
  WHERE _TABLE_SUFFIX BETWEEN '<INICIO_AAAAMMDD>' AND '<FIN_AAAAMMDD>'
    AND device.web_info.hostname IN (
      SELECT hostname FROM `pr-mktg-analyt-prod.<SANDBOX>.mapa_hostname_fuente`
      WHERE dataset_fuente = '<PROPERTY_ID>')
  GROUP BY fecha, hostname
)
SELECT
  COALESCE(u.fecha, s.fecha) AS fecha,
  COALESCE(u.hostname, s.hostname) AS hostname,
  s.eventos AS eventos_origen, u.eventos AS eventos_unificado, u.eventos - s.eventos AS dif_eventos,
  s.usuarios AS usuarios_origen, u.usuarios AS usuarios_unificado, u.usuarios - s.usuarios AS dif_usuarios,
  s.sesiones AS sesiones_origen, u.sesiones AS sesiones_unificado, u.sesiones - s.sesiones AS dif_sesiones,
  s.compras AS compras_origen, u.compras AS compras_unificado, u.compras - s.compras AS dif_compras
FROM u
FULL OUTER JOIN s ON u.fecha = s.fecha AND u.hostname = s.hostname
ORDER BY fecha, hostname
```

Resultado esperado: diferencias de 0 en todas las columnas. Cualquier valor distinto de 0 se investiga antes de ampliar el rango. Si en el unificado se decide excluir pings sin ID o filtrar otros eventos, declara la diferencia esperada antes de correr la prueba.

---

## 14. Capa curada (fase 4, Palace Analytics)

Definiciones de partida cuando se creen vistas curadas:
- Usuarios (rango): `COUNT(DISTINCT user_pseudo_id)`.
- Sesiones (rango): `COUNT(DISTINCT CONCAT(user_pseudo_id, ga_session_id))`.
- Sitio: mapeo del CASE (plantilla 2) o `mapa_hostname_fuente`.
- Vistas particionadas por fecha y agrupadas por `user_pseudo_id`, con la columna `sitio` y la ruta ya extraída, para bajar el costo de las consultas por usuario.
