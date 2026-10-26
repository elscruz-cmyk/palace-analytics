# Inventario de datasets GA4 en BigQuery y propiedades

Fuente: `INFORMATION_SCHEMA.TABLE_STORAGE` (tablas, fechas, GiB; consultada el 26-oct-2026) y una muestra de eventos de UN día por dataset (hostnames principales). Proyecto `pr-mktg-analyt-prod`. Marcas: **CONFIRMADO** / **INFERENCIA** / **POR VALIDAR**.

## 1. Cómo leer esto

- **El nombre de la propiedad de GA4 NO viene en BigQuery.** Los metadatos del dataset solo traen permisos y fechas. Los nombres de esta tabla son **INFERENCIA** a partir de los hostnames, salvo los 6 en **negritas**, que la usuaria confirmó en su mapa de propiedades. Reemplazar por los nombres exactos de GA4 Admin o del Excel `Propiedades_GA4` (20 propiedades; 4 sin dataset: 374580462, 511001955, 474508560 y 403515589, que **no** existen en BigQuery).
- **Frescura:** la última tabla al 26-oct es `events_20261025` (1 día de rezago). Verifica siempre con `list_table_ids` antes de consultar.
- **Nombres (CONFIRMADO por la usuaria):** General Booking (324768211) = todos los sitios de booking de América (Palace Resorts, Moon Palace, Le Blanc). "All Bookings" / Baglioni all bookings (358249801) = todos los sitios de booking de Baglioni.
- **Muestra de hostnames:** un solo día (25-oct; 19-oct, 20-oct y 9-jun para los rezagados o detenidos) y los 4 hostnames principales. Los datasets con 2 a 10 eventos al día tienen una inferencia débil.
- **Días faltantes** = días entre la primera y la última tabla menos las tablas que existen.
- Tipos: **Consolidada** (varios sitios), **Sitio + booking** (contenido y su motor), **Booking** (solo motor), **Marca** (sitio de marca), **Segmento** (sitio de segmento), **Paquetes**, **Otro**.
- Solo 3 datasets tienen `pseudonymous_users_`: 268229233 (1,043 tablas), 355572593 (1,043) y 387217775 (929, con 57 tablas `users_`); además 355522484 tiene 1. Ninguno tiene `events_intraday_`.

## 2. Tabla completa (61 datasets)

| Property ID | Ubic. | Nombre (inferido; **negritas** = confirmado) | Tipo | Tablas `events_` | Primera | Última | Faltan | GiB | Hostnames principales (% de eventos del día) |
|---|---|---|---|---|---|---|---|---|---|
| 250049161 | us-central1 | Beach Palace – Booking | Booking | 1,197 | 2023-07-17 | 2026-10-25 | 0 | 6.4 | bookingsbeachpr.palaceresorts.com 100% |
| 256001573 | us-central1 | Cozumel Palace | Sitio + booking | 1,197 | 2023-07-17 | 2026-10-25 | 0 | 10.3 | cozumel.palaceresorts.com 64%, bookingscozumelpr 35% |
| 257831619 | us-central1 | Beach Palace | Sitio + booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 10.6 | beach.palaceresorts.com 81%, bookingsbeachpr 19% |
| 262041870 | us-central1 | Fundación Palace | Otro | 1,047 | 2023-12-14 | 2026-10-25 | 0 | 0.4 | www.fundacionpalace.org 100% (47 eventos) |
| 268217022 | us-central1 | Le Blanc Brand | Marca | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 16.6 | www.leblancsparesorts.com 98%, onlinebookingspr.leblancsparesorts.com 2% |
| 268220046 | us-central1 | Moon Palace Jamaica | Sitio + booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 42.5 | jamaica.moonpalace.com 75%, bookingsjamaicapr 25% |
| 268224738 | us-central1 | Palace Resorts Brand | Marca | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 18.2 | www.palaceresorts.com 98%, onlinebookingspr.palaceresorts.com 2% |
| **268229233** | us-central1 | **General Palace (360)** | Consolidada | 1,276 | 2023-04-12 | 2026-10-25 | 17 | 545.6 | thegrandpuntacana.moonpalace.com 16%, www.moonpalacecancun.com 12%, thegrand.moonpalace.com 10%, jamaica.moonpalace.com 7% |
| 268240031 | us-central1 | Le Blanc Cancún | Sitio + booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 43.2 | cancun.leblancsparesorts.com 66%, bookingscancunpr 34% |
| 268246216 | us-central1 | Sun Palace | Sitio + booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 11.9 | sun.palaceresorts.com 84%, bookingssunpr 16% |
| 268255275 | us-central1 | Le Blanc Los Cabos | Sitio + booking | 1,198 | 2023-07-15 | 2026-10-25 | 1 | 52.2 | los-cabos.leblancsparesorts.com 79%, bookingscabopr 20% |
| 268270551 | us-central1 | Playacar Palace | Sitio + booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 10.7 | playacar.palaceresorts.com 81%, bookingsplayadelcarmenpr 18% |
| 268272953 | us-central1 | Moon Palace Cancún | Sitio + booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 115.3 | www.moonpalacecancun.com 68%, bookingscancunpr.moonpalace.com 32% |
| 268273835 | us-central1 | Moon Palace Brand | Marca | 1,197 | 2023-07-16 | 2026-10-25 | 1 | 30.8 | www.moonpalace.com 99%, onlinebookingspr.moonpalace.com 1% |
| 268280899 | us-central1 | Moon Palace The Grand | Sitio + booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 67.4 | thegrand.moonpalace.com 72%, bookingsthegrandpr 27% |
| 282021628 | us-central1 | MVG (casi inactivo) | Otro | 960 | 2023-11-22 | **2026-10-19** | 103 | 3.4 | mvg.thepalacecompany.com 100% (2 eventos el 19-oct) |
| 283033027 | us-central1 | Paquetes Palace Resorts | Paquetes | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 3.2 | paquetes.palaceresorts.com 100% |
| 283039175 | us-central1 | Paquetes Le Blanc | Paquetes | 700 | 2023-07-17 | 2026-10-25 | **497** | 0.1 | paquetes.leblancsparesorts.com 100% (113 eventos) |
| 283045662 | us-central1 | Paquetes Moon Palace | Paquetes | 845 | 2023-07-15 | 2026-10-25 | **354** | 1.5 | paquetes.moonpalace.com 100% |
| 290285685 | us-central1 | MVG (casi inactivo) | Otro | 971 | 2023-11-22 | **2026-10-20** | 93 | 6.6 | mvg.thepalacecompany.com 100% (2 eventos el 20-oct) |
| 296656687 | us-central1 | Meetings | Segmento | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 4.1 | meetings.thepalacecompany.com 100% |
| 324662642 | us-central1 | Playacar Palace – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 6.0 | bookingsplayadelcarmenpr.palaceresorts.com 100% |
| 324667354 | us-central1 | Cozumel Palace – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 5.6 | bookingscozumelpr.palaceresorts.com 100% |
| 324679071 | us-central1 | Moon Palace Cancún – Booking | Booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 50.3 | bookingscancunpr.moonpalace.com 100% |
| 324685410 | us-central1 | Sun Palace – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 6.0 | bookingssunpr.palaceresorts.com 100% |
| 324701026 | us-central1 | Le Blanc Cancún – Booking | Booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 12.8 | bookingscancunpr.leblancsparesorts.com 100% |
| 324708313 | us-central1 | Palace Resorts – Online booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 3.2 | onlinebookingspr.palaceresorts.com 100% |
| 324709282 | us-central1 | Moon Palace – Online booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 4.4 | onlinebookingspr.moonpalace.com 100% |
| 324709506 | us-central1 | Le Blanc Los Cabos – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 12.9 | bookingscabopr.leblancsparesorts.com 100% |
| 324734509 | us-central1 | Moon Palace Jamaica – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 19.4 | bookingsjamaicapr.moonpalace.com 100% |
| 324741351 | us-central1 | Weddings | Segmento | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 12.6 | weddings.thepalacecompany.com 100% |
| 324742481 | us-central1 | Moon Palace Nizuc – Booking | Booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 6.3 | bookingsnizucpr.moonpalace.com 100% |
| 324743081 | us-central1 | Palace Pro Agents | Segmento | 1,068 | 2023-11-22 | 2026-10-25 | 1 | 24.9 | www.palaceproagents.com 72%, globalsales.palaceresorts.com 28% |
| 324744621 | us-central1 | Palace Elite | Segmento | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 111.2 | palaceelite.com 66%, www.palaceelite.com 34% |
| 324751406 | us-central1 | Le Blanc – Online booking | Booking | 1,199 | 2023-07-15 | 2026-10-25 | 0 | 2.0 | onlinebookingspr.leblancsparesorts.com 100% (281 eventos) |
| 324754630 | us-central1 | Moon Palace The Grand – Booking | Booking | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 29.6 | bookingsthegrandpr.moonpalace.com 100% |
| **324768211** | us-central1 | **General Booking** | Consolidada (booking) | 1,198 | 2023-07-16 | 2026-10-25 | 0 | 232.4 | bookingsthegrandpuntacana 26%, bookingscancunpr 22%, bookingsthegrandpr 15%, bookingsjamaicapr 10% (todos .moonpalace.com) |
| 338182137 | us-central1 | Palace Productions | Segmento | 1,045 | 2023-12-14 | 2026-10-25 | 2 | 0.2 | productions.thepalacecompany.com 100% |
| 355522484 | us-central1 | Baglioni London – Booking (**detenida 9-jun**) | Booking | 549 | 2023-11-22 | **2026-06-09** | 382 | 1.6 | bookingslondon.baglionihotels.com 100% (3 eventos) |
| 355536914 | us-central1 | Baglioni Brand | Marca | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 10.3 | www.baglionihotels.com 99%, bookings.baglionihotels.com 1% |
| 355568055 | us-central1 | Baglioni Venice | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 7.8 | venice 90%, bookingsvenice 10% |
| 355572592 | us-central1 | Baglioni Maldives | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 13.8 | maldives 94%, bookingsmaldives 6% |
| **355572593** | us-central1 | **General Baglioni** | Consolidada | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 84.9 | maldives 24%, venice 14%, www 11%, rome 11% (.baglionihotels.com) |
| 355594790 | us-central1 | Baglioni Puglia | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 4.5 | puglia 96%, bookingspuglia 4% |
| 355598688 | us-central1 | Baglioni Sardinia | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 8.3 | sardinia 93%, bookingssardinia 7% |
| 355603351 | us-central1 | Baglioni Milan | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 5.0 | milan 91%, bookingsmilan 9% |
| 355607875 | us-central1 | Baglioni Rome | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 8.3 | rome 87%, bookingsrome 13% |
| **355610299** | us-central1 | **Palazzo Firenze** | Sitio + booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 2.8 | florence.baglionihotels.com 90%, bookingsflorence 10% |
| 358235082 | **US** | Baglioni London – Booking (**detenida 9-jun**) | Booking | 563 | 2023-11-22 | **2026-06-09** | 368 | 0.9 | bookingslondon.baglionihotels.com 100% (10 eventos) |
| 358235411 | US | Baglioni Puglia – Booking | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 1.9 | bookingspuglia.baglionihotels.com 100% |
| 358235982 | US | Baglioni – Booking genérico | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 0.7 | bookings.baglionihotels.com 100% (127 eventos) |
| **358247106** | US | **Relais Santa Croce** | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 1.0 | bookingsflorence.baglionihotels.com 100% |
| **358249801** | US | **Baglioni all bookings** | Consolidada (booking) | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 24.7 | bookingsmaldives 27%, bookingsrome 20%, bookingsvenice 18%, bookingssardinia 12% |
| 358250361 | US | Baglioni Venice – Booking | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 3.9 | bookingsvenice.baglionihotels.com 100% |
| 358255235 | US | Baglioni Milan – Booking | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 2.0 | bookingsmilan.baglionihotels.com 100% |
| 358259859 | US | Baglioni Rome – Booking | Booking | 1,068 | 2023-11-22 | 2026-10-25 | 1 | 3.5 | bookingsrome.baglionihotels.com 100% |
| 358263145 | US | Baglioni Sardinia – Booking | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 4.0 | bookingssardinia.baglionihotels.com 100% |
| 358269177 | US | Baglioni Maldives – Booking | Booking | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 6.9 | bookingsmaldives.baglionihotels.com 100% |
| 374573695 | us-central1 | City32 / Grupo Camino Real (fuera de marca) | Otro | 1,069 | 2023-11-22 | 2026-10-25 | 0 | 0.3 | www.city32.mx 99%, www.grupocaminoreal.com 1% |
| 387217775 | us-central1 | MVG – Booking | Booking | 929 | 2024-04-10 | 2026-10-25 | 0 | 9.0 | bookingsmvg 49%, mvg-bpg 34%, mvg 17% (.thepalacecompany.com) |
| 395231481 | us-central1 | Palace Golf | Segmento | 1,047 | 2023-12-14 | 2026-10-25 | 0 | 0.2 | golf.thepalacecompany.com 100% |

Totales: 56,687 tablas `events_` en `us-central1` y 10,183 en `US`. "MVG" es una sigla que no se pudo identificar (POR VALIDAR). "Segmento" para Palace Pro Agents, Palace Elite y Productions es INFERENCIA. La usuaria mencionó además sitios de ventas ("sales"); el candidato es `globalsales.palaceresorts.com` (en 324743081).

## 3. Dónde llega cada hostname de booking (muestra del 25-oct, solo top 4 por dataset; incompleta)

Para la Tarea 1 (General Palace vs General Booking) esta tabla es el punto de partida. La columna "General Palace" está por validar con la matriz de cobertura (archivo 05, plantilla 10): la muestra solo veía los 4 hostnames más grandes de ese dataset.

| Hostname de booking | Datasets que lo reciben (CONFIRMADO en la muestra) | General Palace (268229233) |
|---|---|---|
| bookingsbeachpr.palaceresorts.com | 250049161, 257831619 | POR VALIDAR |
| bookingscozumelpr.palaceresorts.com | 256001573, 324667354 | POR VALIDAR |
| bookingssunpr.palaceresorts.com | 268246216, 324685410 | POR VALIDAR |
| bookingsplayadelcarmenpr.palaceresorts.com | 268270551, 324662642 | POR VALIDAR |
| onlinebookingspr.palaceresorts.com | 268224738, 324708313 | POR VALIDAR |
| onlinebookingspr.moonpalace.com | 268273835, 324709282 | POR VALIDAR |
| onlinebookingspr.leblancsparesorts.com | 268217022, 324751406 | POR VALIDAR |
| bookingscancunpr.leblancsparesorts.com | 268240031, 324701026 | POR VALIDAR |
| bookingscabopr.leblancsparesorts.com | 268255275, 324709506 | POR VALIDAR |
| bookingsjamaicapr.moonpalace.com | 268220046, 324734509, 324768211 | POR VALIDAR |
| bookingscancunpr.moonpalace.com | 268272953, 324679071, 324768211 | POR VALIDAR |
| bookingsthegrandpr.moonpalace.com | 268280899, 324754630, 324768211 | POR VALIDAR |
| bookingsthegrandpuntacana.moonpalace.com | 324768211 | POR VALIDAR |
| bookingsnizucpr.moonpalace.com | 324742481 | POR VALIDAR |
| bookingsmvg.thepalacecompany.com | 387217775 | POR VALIDAR |

Además aparecen hostnames de pago con 0% de los eventos (`bookingscancunpaypr.leblancsparesorts.com`, `bookingsjamaicapaypr.moonpalace.com`, `bookingsthegrandpaypr.moonpalace.com`) y `packages.palaceresorts.com` / `paquetes.*` (1% o menos) mezclados en los datasets de sitio.

Baglioni: el booking de cada ciudad llega a su dataset de sitio (355xxxxxx, `us-central1`), a su dataset de solo booking (358xxxxxx, `US`) y a las consolidadas (General Baglioni y all bookings). El booking de Florencia llega además a Palazzo Firenze (355610299) y a Relais Santa Croce (358247106).

## 4. Observaciones

1. **Huecos y rezagos.** 282021628 (rezago de 6 días) y 290285685 (5 días) casi no tienen tráfico. 355522484 y 358235082 se detuvieron el mismo día (9-jun-2026): son los dos que reciben `bookingslondon`. 283039175 y 283045662 tienen cientos de días sin tabla (POR VALIDAR si es export pausado, vínculo roto o días sin eventos).
2. **Candidatos a dejar fuera del alcance:** los 4 casi inactivos y 374573695 (City32).
3. **Sin dataset propio en la muestra:** el contenido de Moon Palace Nizuc y de Punta Cana no apareció en ningún dataset individual (solo 1 día y top 4). Probablemente solo viven en General Palace; POR VALIDAR.
4. **Ubicaciones:** los 10 datasets de `US` son todos de booking de Baglioni; los demás están en `us-central1`.
5. **Nombres duplicados del mismo sitio:** hay más de un dataset por sitio (sitio + booking, solo booking y consolidadas). Ningún conteo de usuarios o sesiones debe sumarse entre ellos.
