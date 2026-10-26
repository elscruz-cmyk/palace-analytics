# Conocimiento de datos y estado de validación (GA4 → BigQuery vs Looker)

Fecha de corte: 26-oct-2026. Marcas: **CONFIRMADO** (consulta a BigQuery o captura de Looker que compartió la usuaria) / **INFERENCIA** / **POR VALIDAR**. Las cifras marcadas "resumen previo" vienen de chats anteriores y NO tienen tabla de respaldo en el proyecto: hay que regenerarlas por código antes de citarlas como definitivas. Datos validados: ene–sep 2026 (octubre incompleto hasta el 1-nov).

---

## 1. BigQuery y General Palace

- **CONFIRMADO:** proyecto `pr-mktg-analyt-prod` (número 593644259676); ubicación por defecto `us-central1`.
- **CONFIRMADO:** General Palace = propiedad GA4 "General Palace - GA4", ID `268229233` (GA4 360), dataset `analytics_268229233`. 1,276 tablas `events_` (2023-04-12 a 2026-10-25), 1,043 `pseudonymous_users_`; sin `events_intraday_` ni `users_`.
- **CONFIRMADO:** hueco de 17 días en 2023 (29-jun a 15-jul).
- **CONFIRMADO (corrige al archivo anterior):** el export va ~1 día atrás (el 26-oct la última tabla era del 25-oct). Antes se veían ~2 días: verificar siempre con `list_table_ids`.
- **CONFIRMADO:** zona horaria de la propiedad UTC−6. `events_20260915` cubre de 2026-09-15 06:00 UTC a 2026-09-16 05:59 UTC. `event_date` está en esa zona; `event_timestamp` está en UTC (microsegundos). Nombre exacto de la zona: verificar en GA4 Admin.
- **CONFIRMADO:** los datasets `analytics_*` están en `us-central1` (51) y `US` (10). BigQuery no combina ubicaciones en una consulta. Para recuperar resultados de consultas lentas con `get_query_results` hay que indicar la ubicación.
- Inventario completo: archivo 03.

---

## 2. Mapa de sitios (campo calculado "Sitio web" de Looker)

`REGEXP_MATCH` anclado (coincidencia completa) sobre el hostname:

| Sitio web | Hostname (regex) |
|---|---|
| Beach Palace | `^beach\.palaceresorts\.com$` |
| Cozumel Palace | `^cozumel\.palaceresorts\.com$` |
| Playacar Palace | `^playacar\.palaceresorts\.com$` |
| Sun Palace | `^sun\.palaceresorts\.com$` |
| Moon Palace Nizuc | `^nizuc\.moonpalacecancun\.com$` |
| Moon Palace Punta Cana | `^thegrandpuntacana\.moonpalace\.com$` |
| Moon Palace Jamaica | `^jamaica\.moonpalace\.com$` |
| Moon Palace The Grand | `^thegrand\.moonpalace\.com$` |
| Le Blanc Cancun | `^cancun\.leblancsparesorts\.com$` |
| Le Blanc Los Cabos | `^los-cabos\.leblancsparesorts\.com$` |
| Palace Resorts Brand | `^(www\.)?palaceresorts\.com$` |
| Le Blanc Brand | `^(www\.)?leblancsparesorts\.com$` |
| Moon Palace Brand | `^(www\.)?moonpalace\.com$` |
| Moon Palace Cancun | `^(www\.)?moonpalacecancun\.com$` |
| No properties | cualquier otro hostname |

**Motores de reserva observados** (CONFIRMADO, no exhaustivo; ninguno está en el CASE, todos caen en "No properties"): `bookingsbeachpr`, `bookingscozumelpr`, `bookingssunpr`, `bookingsplayadelcarmenpr` (todos `.palaceresorts.com`); `onlinebookingspr` (en `.palaceresorts.com`, `.moonpalace.com` y `.leblancsparesorts.com`); `bookingscancunpr`, `bookingsthegrandpr`, `bookingsthegrandpuntacana`, `bookingsjamaicapr` y `bookingsnizucpr` (`.moonpalace.com`); `bookingscancunpr` y `bookingscabopr` (`.leblancsparesorts.com`); `bookingsmvg.thepalacecompany.com`; hostnames de pago (`...paypr...`) con ~0% de los eventos; y en Baglioni `bookings<ciudad>.baglionihotels.com` y `bookings.baglionihotels.com`.

## 3. Configuración relevante del reporte de Looker

- **CONFIRMADO:** filtro de página "Solo propiedades": excluye `[CF] Sitios web` que contenga "No properties". **INFERENCIA:** si aplica a todo el reporte, no incluye los motores de reserva (donde ocurren `begin_checkout` y `purchase`). Confirmarlo.
- **CONFIRMADO (trampa):** la dimensión `Date (Year Month)` infla Total users (Beach, septiembre: 21,670 contra 18,970 con la dimensión nativa `Month`). Con `Date (Year Month)`, Looker quedaba 20–25% sobre BigQuery todos los meses; con `Month` la diferencia real es ~7%.
- Selector "Sitio web", selector de rango de fechas y comparación contra el año anterior en el encabezado.

---

## 4. Reglas fijas (no volver a discutir)

- Brecha = **1 − BigQuery / Looker** (equivale a 1 − 1/factor; factor = GA4/BQ). Una sola convención.
- Comparar con dimensión nativa `Month`, nunca `Date (Year Month)`.
- Usuarios no son aditivos (días, meses, sitios, propiedades). Sesiones = `CONCAT(user_pseudo_id, ga_session_id)` distintos.
- Eventos sin `user_pseudo_id` = pings sin consentimiento: no son usuarios ni sesiones. No se corrige para "igualar" a GA4.
- Looker "Transactions" = eventos `purchase` con ID (nativa). Event count sí incluye eventos sin ID.
- No mostrar `user_pseudo_id`; buscar por `transaction_id`. Un ID = un navegador/dispositivo. No hay `user_id`.
- Búsquedas idénticas no unen personas; server-side tagging no une dispositivos; el % de cambio de dispositivo no se puede medir con exactitud.
- Regex de hostname siempre anclado (`^...$`). Los motores de reserva caen en "No properties".
- Generar tablas por código.
- **Decisión de la usuaria (26-oct-2026):** en booking se compara Looker SIN el filtro "Sin admin-bengine". Los datos de Looker con ese filtro no se usan para validar ni reconciliar; se conservan solo como evidencia de que el filtro infla usuarios.

---

## 4b. Brecha por alcance y causa (resumen al 26-oct-2026)

Convención: brecha = 1 − BigQuery / Looker. No hay una cifra total única: se reporta por alcance (un promedio global ponderado se puede calcular por código, pendiente).

| Alcance | Total users | Sesiones | Respaldo |
|---|---|---|---|
| América, contenido | ~7.0–7.4% (14 sitios: 6.4–8.6%) | ~6.6% | 4 sitios en el CSV 06; el resto, resumen previo (regenerar antes de citar) |
| América, booking | 0.1–0.4% (máx 1.3%) | PENDIENTE | resumen previo |
| Baglioni, contenido | 27–37% | Florencia sept: 32.0% (General Baglioni) | resumen previo |
| Baglioni, booking | ±1.2% | ±2.9% | resumen previo |

- **Causa (INFERENCIA fuerte, no probada):** consentimiento denegado, ping sin ID y modelado de comportamiento de GA4. Evidencia: r 0.71–0.98 con la proporción de pings sin ID; pendiente combinada 0.68 (IC 95% 0.54–0.81); caso Jamaica en julio. Falta validar Reporting Identity y la documentación oficial sobre modelado y Consent Mode.
- **Booking (postura de la usuaria):** no hay banner de cookies, por eso la brecha es mínima. Matiz: General Booking tuvo eventos sin ID (Le Blanc, jun–jul) y que ningún `purchase` tenga `analytics_storage = No` puede significar compras sin contar. Si se agrega un banner, la brecha subirá: fijar línea base ahora.
- **Regla de deriva:** si la proporción de pings sin ID sobre usuarios cambia más de ~2 puntos contra la calibración, recalibrar.
- **Siguiente paso:** brecha entre booking de General Palace y General Booking (plantilla 10 del archivo 05).

---

## 5. Estado de validación (Looker vs BigQuery)

| Métrica | América contenido | América booking | Baglioni contenido | Baglioni booking |
|---|---|---|---|---|
| Total users | Validado: BQ ~7.0–7.4% debajo (14 sitios 6.4–8.6%; 8.2% con Palace Resorts Brand) | Validado **sin** el filtro "Sin admin-bengine": 0.1–0.4% (máx 1.3%). Con el filtro: Looker +13–20% (cifra NO usable; ver sección 4) | Brecha 27–37% (8 sitios; 35.7% prom.; Brand 56.9%), inestable | ±1.2% |
| Sesiones | ~6.6% debajo (7.9% con PRB) | PENDIENTE | Revisado | ±2.9% (Florencia: todo cuadra a 0.2%) |
| Page views | 0% de diferencia (Beach, Le Blanc Cabos, Punta Cana); resto pendiente | PENDIENTE | PENDIENTE | PENDIENTE |
| Transacciones | Beach booking: purchase ÷ users; sep 42 / 6,200 = 0.68% | Revisado | Revisado | Revisado |
| Active users, key events, ingresos (y moneda), campañas, canales, dimensiones | PENDIENTE | PENDIENTE | PENDIENTE | PENDIENTE |

Solo "América contenido, Total users, 4 propiedades" tiene datos en el proyecto (archivo `06_datos_validacion_total_users.csv`, 36 filas). El resto es resumen previo y sin tabla: **subir esas tablas al proyecto nuevo cuando se regeneren**.

### 5.1 Total users, contenido de América (CONFIRMADO; n = 9 meses por propiedad)

| Propiedad | Factor GA4/BQ | BQ debajo de GA4 | CV | Predicción 95% (mes nuevo) | Pings sin ID / usuarios BQ |
|---|---|---|---|---|---|
| Beach Palace | 1.068 | 6.4% | 1.4% | 1.032–1.104 | 7.8% |
| Cozumel Palace | 1.094 | 8.6% | 1.7% | 1.049–1.139 | 11.0% |
| Sun Palace | 1.084 | 7.7% | 1.6% | 1.042–1.125 | 9.9% |
| Moon Palace Jamaica | 1.073 | 6.8% | 2.3% | 1.014–1.132 | 10.6% |

- BQ por debajo de GA4 en 126 de 126 meses-sitio; estable por sitio (CV 1.4–2.3%); el nivel cambia entre propiedades (ANOVA p = 0.032; Kruskal-Wallis p = 0.051): usar un factor por propiedad, no uno universal. El volumen no determina el factor.
- La brecha acompaña a la proporción de pings sin ID (r = 0.71 a 0.98 por propiedad; pendiente combinada 0.68, IC 95% 0.54–0.81 puntos de brecha por punto de pings). **INFERENCIA:** causa = modelado de GA4 para usuarios sin consentimiento (Google documenta que BigQuery no incluye modelado). Es correlación, no causalidad.
- Caso Jamaica: en julio la proporción de pings casi se duplicó (~8% a ~15%) y la brecha pasó de ~5.6% a ~10.4%.
- Saltos de pings por investigar: Cozumel (jun), Jamaica (jul), Baglioni Brand (desde abril), Punta Cana (Portugal, DV360 display, abr–may), Palace Resorts Brand → DV360.
- **Regla de deriva (práctica, no estándar):** si la proporción de pings sin ID sobre usuarios cambia más de ~2 puntos frente al periodo de calibración, recalibrar.
- Limitaciones: p-valores combinados sobreestimados (series no independientes); cifras de Looker transcritas de capturas; Looker estima usuarios con un algoritmo aproximado; Cozumel y Sun son sensibles a quitar un mes.
- Criterio de aceptación propuesto (PENDIENTE de decidir antes de ver resultados): mismo signo todos los meses, CV ≤ 3%, sin tendencia con tiempo o volumen, rango de predicción dentro del margen que tolere el negocio.
- Ref.: Beach, septiembre: GA4 18,970 vs BigQuery 17,685.

### 5.2 Booking América y el filtro "Sin admin-bengine" (resumen previo)

- Con el filtro, Looker cuenta a cada usuario una vez por página distinta vista (Beach, 15-sep: 240 únicos vs 293 en Looker). Sin el filtro, Looker = BQ. Beach, septiembre: Looker sin filtro 6,142; con filtro 7,345; BQ 6,200. **Etiqueta correcta:** BQ queda 15.6% debajo de Looker con filtro (= 1 − 6,200/7,345); equivale a Looker +18.5% sobre BQ. No mezclar las dos bases (la presentación del 14-oct las mezcla).
- `/admin-bengine` en BigQuery: 152 usuarios en el año (29 solo ahí); 4,466 eventos de 53.1M; excluirla cambia 0–7 usuarios por mes por motor.
- Mecanismo interno de Looker **no probado ni documentado**. En el consolidado de Palace Resorts (ene–abr) el filtro **reduce** hasta −50%: sin explicar. "BE Clever" vs "tabla 1" difieren ene–jun: sin explicar.
- Beach: del 16-abr al 31-may, `purchase` con `transaction_id` = "(not set)" (29 eventos).

### 5.3 Florencia (septiembre, florence.baglionihotels.com; resumen previo)

- Sesiones GA4: General Baglioni 10,723 vs Palazzo 7,353. BQ: 7,295 y 7,287. No es recolección ni filtro de Data Studio (GA4 = Data Studio). **INFERENCIA:** modelado de GA4. General Baglioni cumple los umbrales de Google (27/27 días); Palazzo no (+47.0% vs +0.9% sobre BQ).
- Booking cuadra al 0.2% con BQ: General Baglioni 2,552; Palazzo 2,296; Relais 2,561; All bookings 2,562. Excepción: el filtro "Sin admin-bengine" suma +210 (2,772) en All bookings.
- Palazzo no recibe `page_view` ni `user_engagement` del booking, solo un subconjunto de eventos (revisar GTM).
- El `user_pseudo_id` es el mismo en las 4 propiedades (99.6–100%): sumar usuarios o sesiones entre ellas duplica personas.
- Conservación (booking que también estuvo en contenido): General Baglioni 1,620 (76.5% / 27.1% del contenido); Palazzo 1,513 (76.0% / 25.3%); contenido Palazzo + booking Relais 1,617* (76.6% / 27.1%). *Mínimo confirmado (1 de 2,112 claves perdida al cruzar ubicaciones).
- Pings sin ID / usuarios BQ: 7.8–11.0% (América) vs 27–37% (Baglioni contenido). POR VALIDAR si es otra definición o consentimiento distinto. No aplicar factores de América a Europa.

---

## 6. Hallazgos por tema

### 6.1 Total users de septiembre 2026 en General Palace (CONFIRMADO; no validado contra Looker)

| Alcance | Total users | Active users |
|---|---|---|
| Toda la propiedad (todos los hostnames) | 789,430 | 779,248 |
| Con "Solo propiedades" (excluye "No properties") | 697,289 | n/d |
| Solo hostnames "No properties" | 287,670 | 285,897 |

Por sitio (sep): Moon Palace Brand 130,814; Punta Cana 127,722; Moon Palace Cancun 117,669; The Grand 92,781; Jamaica 54,969; Le Blanc Los Cabos 50,629; Palace Resorts Brand 47,710; Nizuc 42,749; Le Blanc Cancun 38,154; Beach 17,685; Le Blanc Brand 16,669; Sun 13,726; Cozumel 12,280; Playacar 9,903. Suman 773,460; los distintos con "Solo propiedades" son 697,289: la diferencia (76,171) son usuarios que visitaron más de un sitio. ~195,529 usuarios aparecen tanto en un sitio mapeado como en "No properties" (INFERENCIA: cross-domain conserva el `user_pseudo_id`).

### 6.2 Embudo de septiembre en motores de reserva (usuarios distintos en el mes; no es un embudo por sesión)

| Motor | Usuarios | Ven resultados | Inician checkout | Compran | Checkout → compra |
|---|---|---|---|---|---|
| bookingscancunpr.moonpalace.com | 64,230 | 58,627 | 13,219 (20.6%) | 582 | 4.4% |
| bookingsthegrandpr.moonpalace.com | 45,939 | 42,101 | 9,477 (20.6%) | 495 | 5.2% |
| bookingsthegrandpuntacana.moonpalace.com | 42,760 | 39,438 | 8,166 (19.1%) | 338 | 4.1% |

- Más del 94% de quienes inician checkout no compran el mismo mes. POR VALIDAR: parte puede ser medición (cross-domain, pasarela de pago, consentimiento) y no abandono; contrastar con el sistema de reservas.
- `add_to_cart` no apareció en estos 3 motores en septiembre. Tasa de `click_to_booking` en sitios de marca (sep): moonpalace.com 11.5%, palaceresorts.com 13.2%, Nizuc 17.2%, Jamaica 36.4%, The Grand 35.4%.
- No se verificó la moneda de `purchase_revenue`: no usar ingresos sin validar moneda y contrastar con reservas.

### 6.3 Calidad del evento `purchase` (todas las propiedades de General Palace, septiembre)

- 2,766 eventos; `user_id` 0; `user_pseudo_id` 100% (2,571 distintos); `transaction_id` 2,761 (99.8%), todos distintos; 5 sin `transaction_id` (revisar hostname).
- `user_properties` vacío; `analytics_storage = No`: 0 eventos (**INFERENCIA:** las compras de quien rechazó cookies no llegan a BigQuery, así que compras e ingresos podrían estar subestimados).
- Parámetros: `ga_session_id`, `ga_session_number`, `ga_cookie` (2,550 eventos; contenido POR VALIDAR), `hotel_name`, `start_date`, `end_date`, `number_of_adults`, `number_of_children`, `nights`, `room_nights`, `creation_date`, `payment_type` y `paymentType` (mismo significado), `language_code`, `coupon`, `tpc_site`, `conversion_rate`, `currency`, `value`, `version`. 2 eventos con esquema camelCase (`hotelID`, `numRooms`, `checkIn`): implementación antigua o de pruebas.
- Llave natural para cruzar con el sistema de reservas: `transaction_id`.

### 6.4 Recorridos de usuario

- Método: reconstruir todos los eventos de un `user_pseudo_id` en orden. Muestra de 5 compradores de Le Blanc el 15-sep (regla fija, no representativa): la misma sesión cruza el sitio de marca y el motor (cross-domain funciona en estos casos); `begin_checkout` se dispara cada vez que carga `/checkout` (vaivén `/rooms` ↔ `/checkout`: infla el conteo de eventos, no el de usuarios); tras comprar vuelven a `/confirmation` en una sesión nueva sin segundo `purchase`.
- Moon Palace Cancún, septiembre (resumen previo): 582 compradores; 54% compró el mismo día de su primer evento; 79% pasó por contenido; 72% de EE. UU.; 59% desktop. 13,219 iniciaron checkout y 580 reservaron (4.4%); desktop 6.3% vs mobile 3.0%; >1 día 9.4% (mismo día 3.7%); 3+ checkouts 13.1%.
- Usuarios sin ecommerce (Moon Cancún, sep): 145,098 usuarios; 81,115 sin ecommerce (55.9%). Campaña Display MF: 19,652 usuarios, 97% de una sesión (INFERENCIA: baja calidad; confirmar en DV360).
- Baglioni, paso 1 (`view_item_list`), 1-ago a 12-oct: 52,277 usuarios; 46,102 no avanzan (88.2%); 6,175 a checkout; 512 compran. Italia: 19,006 usuarios, 4.2 compras/1,000 (EE. UU. 15.0). Celular: 56% del paso 1, 5.2/1,000 (desktop 15.7).
- POR VALIDAR: ~46% de quienes tocaron el motor sin ver resultados no tiene `page_view`; el tiempo activo cae ~81 s → ~40 s desde la semana del 31-ago; `click_offers_hotel_flight` 23.5% desktop vs 0.0% mobile.
- Límites: un ID = un navegador/dispositivo; quien rechaza cookies no tiene ID; el orden depende de `event_timestamp` (desempatar con `batch_ordering_id`).

### 6.5 Viabilidad de una propiedad 360 única (CONFIRMADO en análisis previo, 15 propiedades de `us-central1`, sin Baglioni bookings)

- Cabe en volumen: marzo 2026, 35.6M eventos; usuarios unidos 1.88M vs 2.27M sumados (−17%); sesiones 2.93M vs 3.02M (−2.9%); 8,966 compras. Histórico 870M eventos en 1,267 días; pico diario 1.65M; 2026 ≈ 351M al año. Export diario 360: 20 mil millones.
- Riesgos: cross-domain (límite 100 condiciones; 96 hostnames en marzo); 145 claves de parámetro en 7 días vs 125 dimensiones registrables; eventos por mes del contrato: no visto.
- El histórico no se une dentro de GA4 (Measurement Protocol, 72 h): vive en BigQuery con columna de propiedad de origen.
- Limpiar antes: 23 de 96 hostnames mandan a 2+ propiedades; nombres duplicados (`convertion_rate`/`conversion_rate`, `agancy_id`, `totalAdults`/`total_adults`/`total_guests`); 13 claves con texto y número; parámetros `hash_email` y `user_login` (no se revisaron valores; validar con legal); Baglioni ~19% de eventos sin ID por GDPR.

### 6.6 [CF] Looker traducidos a SQL (sin validar contra Looker)

- [CF] Medios: la rama DV360 puede dejar pasar canales no DSP (precedencia AND/OR); "GPMAx"/"BPMAx" con "A" mayúscula; varias ramas caen al ELSE. Falta la fórmula de [CF] Medios.
- [CF] Canales: la regla OTA's busca "referral" en la fuente (probable error: debería ser el medio). Faltan validar Canales, Estrategias y Mercado.
- La fórmula completa de [CF] Canales vive en un archivo previo: guardarla como `campos_calculados_looker.md` cuando la usuaria la comparta.

### 6.7 Costos de referencia

- BigQuery on-demand: 6.25 USD/TiB (primer TiB al mes gratis). Consulta típica del agente: ≈5.6 GiB ≈ 0.034 USD. Almacenamiento: General Palace 545.6 GiB lógicos (≈9 USD/mes), General Baglioni 84.9 GiB (≈1.4 USD/mes), General Booking 232.4 GiB.
- Operación mensual de Palace Analytics (200 / 500 / 1,000 preguntas): BigQuery ≈ 7 / 28 / 62 USD; Claude Sonnet 5.5 ≈ 18 / 45 / 90 USD. Reporte de comportamiento ≈ 1 USD semanal o ≈ 5 USD diario. Desarrollo ≈ 273 horas (presentación del 14-oct; supuestos de la usuaria por confirmar, incluidos los 300 USD de créditos).

---

## 7. Errores a no repetir

- Cifras mal tecleadas (Moon Palace Cancún sep 6.2%, no 5.4%; Punta Cana ago 8.6%, no 8.7%): tablas por código.
- General Booking SÍ tuvo eventos sin ID (Le Blanc, jun y jul).
- Son 4,466 eventos de 53.1M (no 52.9M).
- Hipótesis descartadas: suma de usuarios diarios, filtro a nivel de gráfica "funciona", hostnames de pago, rutas, usuarios compartidos entre motores.
- Retirado: Paid Search 3.2 s de interacción; "−17.2%" de Florencia; 2,111 sesiones (el correcto es 2,552).
- Etiquetar la diferencia con una sola base (1 − BQ/Looker).

## 8. Pendientes técnicos

1. Pings: 7.8–11% (América) vs 27–37% (Baglioni contenido). Saltos de Cozumel (jun), Jamaica (jul), Baglioni Brand (abril), Punta Cana (abr–may).
2. Prueba de control del filtro admin-bengine (`/zzz-no-existe`); otros tipos de filtro. Baja prioridad: ya no se usan datos con el filtro; solo si hace falta explicar el mecanismo a quien mantiene el reporte.
3. Por qué Transactions excluye eventos sin ID y Event count los incluye.
4. Palazzo manda solo un subconjunto de eventos del booking (GTM). Qué es `codename` en el filtro de contenido. Abrir usuarios "solo booking" (498 General Baglioni, 478 Palazzo).
5. `transaction_id` perdido (16-abr a 31-may, Beach); 5 `purchase` sin `transaction_id`; 3 compras de diferencia entre propiedades de Baglioni; 34 segundas compras; `payment_type`/`paymentType`; 2 eventos camelCase; contenido de `ga_cookie`.
6. Caída del consolidado de motores a ~500 usuarios por mes desde marzo; Italia y Maldives en checkout; `begin_checkout` en cada carga de `/checkout` (medir a escala); tiempo activo; `page_view` del motor; `click_offers_hotel_flight` en mobile; auto-referidos.
7. Reporting Identity (General Palace, General Baglioni, Palazzo, General Booking) y documentación oficial de Google sobre modelado de comportamiento y Consent Mode.
8. Piloto de cruce con el sistema de reservas (extracto por `transaction_id`, sin datos personales).
9. Medir qué % de compras viene de IDs con `first_visit` el mismo día (señal de fragmentación de identidad; evaluar User-ID).
10. Octubre: decidir tratamiento al cerrar (1-nov).
11. Legal: `hash_email`, `user_login`.

*Análisis propio, no es una revisión certificada. Si respalda una decisión formal, validar con una persona de estadística y con la documentación oficial de Google.*
