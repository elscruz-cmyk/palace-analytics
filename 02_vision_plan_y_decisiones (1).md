# Visión, plan y decisiones: propiedad unificada y Palace Analytics

Fecha de corte: 26-oct-2026. Marcas: **CONFIRMADO** (consultado en BigQuery o dicho por la usuaria) / **INFERENCIA** / **POR VALIDAR**. Las secciones 1 y 2 vienen de lo que explicó la usuaria; las propuestas del asistente están marcadas como tales.

---

## 0. Qué hemos hecho y dónde estamos (al 26-oct-2026)

**Hecho hasta hoy**
1. Revisamos la diferencia de datos entre Looker (GA4) y BigQuery: el alcance (contenido vs booking; América vs Baglioni) y el tamaño de la brecha en Total users y sesiones.
2. Identificamos la causa más probable de la brecha (consentimiento de cookies y modelado de GA4) y qué parte de la medición no se pierde (los eventos).
3. Confirmamos que en booking la brecha es mínima.
4. Inventariamos los 61 datasets GA4 y los hostnames que recibe cada uno (archivo 03).
5. Fijamos las reglas de comparación: brecha = 1 − BigQuery / Looker; dimensión nativa `Month`; datos de booking de Looker SIN el filtro "Sin admin-bengine".
6. Diseñamos la ruta hacia el histórico unificado (secciones 3 a 6).

**Brecha de datos por alcance** (BigQuery debajo de Looker; brecha = 1 − BQ / Looker)

| Alcance | Total users | Sesiones | Estado |
|---|---|---|---|
| América, contenido | ~7.0–7.4% (14 sitios: 6.4–8.6%; 8.2% con Palace Resorts Brand) | ~6.6% (7.9% con Palace Resorts Brand) | Total users con datos en el CSV (4 sitios); el resto es resumen previo |
| América, booking (sin el filtro admin-bengine) | 0.1–0.4% (máx 1.3%) | PENDIENTE | Resumen previo, sin tabla |
| Baglioni, contenido | 27–37% (35.7% promedio de 8 sitios; Brand 56.9%); inestable | Florencia, septiembre: General Baglioni 32.0% (GA4 +47.0% sobre BQ); Palazzo 0.9% | Resumen previo |
| Baglioni, booking | ±1.2% | ±2.9% (Florencia: 0.2%) | Resumen previo |

No hay una sola cifra "total": la brecha depende del alcance (contenido o booking) y de la región, porque cambia con el consentimiento. Una cifra global ponderada se puede calcular por código (pendiente): se pondera la brecha de cada alcance, no se suman usuarios, porque los usuarios no son aditivos.

**Por qué pasa la diferencia.**
- *Postura de la usuaria:* la brecha viene de los usuarios que rechazan cookies.
- *Evidencia (CONFIRMADO):* la brecha sube y baja con la proporción de pings sin ID (r = 0.71 a 0.98 por propiedad; pendiente combinada 0.68 puntos de brecha por punto de pings). En Jamaica, cuando los pings casi se duplicaron en julio, la brecha pasó de ~5.6% a ~10.4%. Google documenta que el export a BigQuery no incluye modelado de comportamiento.
- *Estado:* **INFERENCIA fuerte, no probada.** Es correlación, no causalidad. Falta validar Reporting Identity de las propiedades y la documentación oficial de Google sobre modelado de comportamiento y Consent Mode.

**Booking: por qué hay menos diferencia.**
- *Postura de la usuaria:* en los motores de reserva no hay banner de cookies; por lo tanto no hay pérdida de datos por consentimiento.
- *Evidencia (CONFIRMADO):* la brecha de usuarios en booking es 0.1–0.4% (Baglioni ±1.2%) y en septiembre ningún `purchase` de General Palace tenía `analytics_storage = No`.
- *Matices (POR VALIDAR):* (1) que los motores no tengan banner es consistente con los datos, pero no se ha comprobado en el sitio; (2) "sin pérdida" es un poco fuerte: queda una diferencia pequeña (hasta 1.3% en usuarios; ±2.9% en sesiones de Baglioni booking) y General Booking sí tuvo eventos sin ID (Le Blanc, junio y julio); (3) queda abierta cómo llega el estado de consentimiento al motor desde el sitio de contenido.

**Riesgo a futuro (compartido con la usuaria).** Si más adelante se implementa un banner de cookies en booking, habrá pérdida de datos también ahí y la brecha subirá. Conviene fijar ahora la línea base de booking (pings sin ID sobre usuarios y brecha contra Looker) y vigilar la regla de deriva: un cambio de más de ~2 puntos en la proporción de pings sin ID obliga a recalibrar.

**Siguiente paso, antes de unir los datasets.** Medir la diferencia de datos entre los datos de booking de General Palace y los de General Booking (América; el par equivalente de Baglioni es General Baglioni vs Baglioni all bookings; ver sección 3). Con eso se decide la fuente de booking del histórico unificado.

---

## 1. Qué se quiere lograr

**Palace Analytics como plataforma (visión de la usuaria, 26-oct-2026).** Una plataforma tipo GA4, totalmente personalizada a nuestro contexto y necesidades, integrada con un agente que responde a cualquier área dudas sobre tráfico del sitio, usuarios, canales y comportamiento de usuario. Viene a resolver los límites de Looker:
- Limita la cantidad de tablas y de cruces de información.
- Un filtro como "Sin admin-bengine" cambia cómo se mide y produce un dato completamente distinto.
- No se puede ver, a nivel `user_pseudo_id`, lo que hizo un usuario durante todo su viaje.

Aunque haya diferencia de datos por la pérdida generada por cookies, lo importante es tener claro cuánto se pierde y sacar provecho de los datos que sí tenemos. Después se ligará con otras fuentes (Campaign Manager y otras bases) para sumar ventas reales, atribución, etc. (fase 5).

**Palace Analytics, componentes.** Un agente de datos (nombre de trabajo "Brújula") y reportes interactivos para que cualquier área (Inbound, Social Media, Planning y otras) consulte en lenguaje natural ("¿cuántos usuarios vieron X?", "rendimiento de la campaña Black Friday", "tráfico del canal SEO", "experiencia más vista en la propiedad Y"), con las mismas definiciones de los reportes de Looker Studio.

**Por qué usar los datos de BigQuery y no la propiedad de GA4 / Looker.**
- BigQuery trae el dato crudo de cada evento; se puede seguir lo que hizo cada usuario identificado (recorridos, tiempo a compra, abandono por paso). Es ligado a `user_pseudo_id`: un navegador/dispositivo, no una persona (no hay `user_id`; quien rechaza cookies no tiene ID). **Precisión del asistente:** "ligar al usuario" significa a nivel de dispositivo, y no se puede medir con exactitud el cambio de dispositivo.
- Libertad para crear reportes a nuestra medida, sin los límites de Looker.
- No se pueden cargar datos a mano mes a mes: la fuente del agente debe ser BigQuery.
- Costo bajo: 6.25 USD/TiB, primer TiB al mes gratis (ver archivo 04 para costos de referencia).

**Infraestructura de medición unificada (previa a Palace Analytics).** Hoy todo está separado en propiedades distintas. El objetivo es una sola propiedad GA4 "conglomerado" (nombre por definir) donde lleguen los datos de sitios de contenido, sitios de booking y sitios de segmento (weddings, meetings, etc.), con cross-domain, para ver el viaje del usuario a nivel compañía sin que Google Analytics lo corte entre propiedades.

---

## 2. Cómo está organizada la medición hoy (según la usuaria, contrastado con BigQuery)

| Tipo de propiedad | Qué recibe | Estado |
|---|---|---|
| **General Palace** (268229233, GA4 360) | Sitios de contenido y de booking de América | CONFIRMADO: dataset con contenido de Moon Palace, Le Blanc, Palace Resorts y motores de reserva |
| **General Booking** (324768211) | Solo sitios de booking | CONFIRMADO que recibe motores de Moon Palace. POR VALIDAR si recibe los de Le Blanc y Palace Resorts (el traspaso previo vio eventos sin ID de Le Blanc en jun y jul) |
| **General Baglioni** (355572593) y **Baglioni all bookings** (358249801) | Equivalentes de Baglioni: contenido + booking, y solo booking | CONFIRMADO por hostnames; booking cuadra entre ambas al 0.2% en Florencia |
| Propiedades independientes por marca | Un dataset de sitio (con su booking) y otro solo de booking por cada sitio | CONFIRMADO (ver archivo 03) |
| Propiedades de sitios de segmento | Una propiedad por sitio: weddings, meetings, golf, productions, Palace Elite, Pro Agents, etc. | CONFIRMADO |

Resumen del inventario (CONFIRMADO, consultado el 26-oct): 61 datasets GA4 (51 en `us-central1`, 10 en `US`), 17 datasets `_temp` aparte (restos de nov-2023). El detalle por dataset está en el archivo 03.

**Regla de partida:** los datos de contenido existen solo en las propiedades de sitio y en General Palace (General Booking no los trae). Para los sitios de segmento y demás sitios con una sola propiedad no hay duda de fuente.

---

## 3. Decisión clave pendiente (Tarea 1 de Asana): es el SIGUIENTE paso, antes de unir los datasets

**Nota de nombres (CONFIRMADO por la usuaria, 26-oct-2026):** son dos propiedades distintas. **General Booking (324768211)** trae todos los sitios de booking de América (Palace Resorts, Moon Palace, Le Blanc). **All Bookings / Baglioni all bookings (358249801)** trae todos los sitios de booking de Baglioni. Por tanto hay dos pares de comparación: América = booking en General Palace vs General Booking (Tarea 1 de Asana); Baglioni = booking en General Baglioni vs All Bookings (el par de Florencia ya cuadra ~0.2%, falta el resto).

Para los **sitios de booking de América** hay dos fuentes posibles dentro del histórico unificado: los datos de booking de **General Palace** o los de **General Booking**. Para decidir hace falta medir la diferencia entre ambas.

Propuesta de método (asistente, POR VALIDAR con la usuaria):
1. **Matriz de cobertura de hostnames.** Qué hostnames de booking llegan a cada propiedad y desde qué fecha. Las dos están en `us-central1`, así que se pueden comparar en una sola consulta (plantilla en archivo 05).
2. **Diferencia por mes y por hostname**, de enero a septiembre de 2026 como mínimo: eventos, eventos sin ID, usuarios, sesiones, `view_item_list`, `begin_checkout`, `purchase`, transacciones distintas e ingresos (verificando moneda).
3. **Cruce de compras por `transaction_id`:** cuántas están en ambas, solo en General Palace y solo en General Booking. Es la prueba más fuerte de cuál pierde compras.
4. **Contra el sistema de reservas** (fuente de verdad por definir): cuál de las dos se parece más.
5. **Criterios (fijar ANTES de ver resultados):** dirección consistente, cobertura de hostnames completa, diferencia estable por mes (CV), error en compras. Los umbrales los define la usuaria.
6. **Convención:** brecha = 1 − A / B con B = la fuente de referencia que se declare en la consulta.

Datos de contexto que ya se conocen: General Palace empieza el 12-abr-2023 (hueco de 17 días del 29-jun al 15-jul de 2023); General Booking empieza el 16-jul-2023 sin huecos; General Booking pesa 232.4 GiB y General Palace 545.6 GiB. En Baglioni, el booking de General Baglioni y el de all bookings cuadran al 0.2% (septiembre, Florencia).

---

## 4. Plan por fases

| Fase | Qué se hace | Entregable | Criterio de salida | Estado |
|---|---|---|---|---|
| 0. Organización | Inventario de datasets, nombres, hostnames, huecos, ubicaciones | Archivo 03 | Todos los datasets identificados | **Hecho** (nombres exactos POR VALIDAR contra GA4 Admin y el Excel `Propiedades_GA4`) |
| 1. Fuente de booking América (Asana 1) | Diferencia promedio General Palace-booking vs General Booking | Matriz de cobertura + diferencias + decisión | Fuente elegida y documentada | Pendiente |
| 2. Pruebas de unificación (Asana 2) | Unir los datasets en BigQuery y reconciliar contra cada propiedad individual | Tabla unificada de prueba + reporte de reconciliación | Todas las métricas dentro del umbral fijado de antemano | Pendiente |
| 3. Implementación de la unificación (Asana 3) | Día de corte: bajar los IDs de medición y dejar uno solo; periodo en paralelo previo | Propiedad conglomerado en producción | Mediciones equivalentes durante el paralelo; reversión probada | Pendiente; sin fecha |
| 4. Palace Analytics | Capa curada en BigQuery, definiciones de Looker, agente y reportes | Agente "Brújula" + reportes | Números dorados verificados | Puede avanzar en paralelo sobre General Palace (INFERENCIA) |
| 5. Integración con otras fuentes | Ligar Campaign Manager y otras bases (ventas reales, atribución, etc.) con la capa curada de Palace Analytics | Modelo de datos unido (Floodlight, reservas, costos) y reportes de atribución | Compras de BigQuery conciliadas contra el sistema de reservas y Campaign Manager, dentro de un umbral fijado antes | Futuro; por definir |

Orden propuesto de subpasos dentro de la fase 2 (asistente):
1. Mapa `hostname → dataset fuente` (decisión de fase 1 + sitios con una sola fuente).
2. Prueba pequeña: un mes y 2 a 3 datasets.
3. Reconciliar (misma propiedad de origen + hostname + día).
4. Ampliar a todos los datasets de `us-central1`.
5. Resolver Baglioni en `US` (copiar entre ubicaciones o usar General Baglioni como fuente de booking; ver sección 6).
6. Histórico completo (julio 2023 a la fecha) y vistas por día.

**Sobre la fase 5 (POR VALIDAR).** Google Cloud documenta un conector de BigQuery Data Transfer Service para cargar datos de Campaign Manager en BigQuery (https://docs.cloud.google.com/bigquery/docs/doubleclick-campaign-transfer). Solo se pudo leer el título y la descripción de esa página: los tipos de datos, requisitos, límites y versión hay que revisarlos antes de diseñar. El esquema del export de GA4 ya trae campos de campaña de CM360, DV360, Google Ads y SA360 (por ejemplo `cm360_campaign` dentro de `session_traffic_source_last_click`); que estén poblados en nuestras propiedades no está verificado. Definir también cuál es la fuente de verdad de ventas (sistema de reservas, Campaign Manager o GA4).

**Ejemplo de prueba de la usuaria:** si en el unificado hay 100,000 sesiones de weddings, debe haber 100,000 en la propiedad de weddings para el mismo periodo. **Precisión:** la prueba aplica por alcance (propiedad de origen + hostname + periodo); los totales sumados entre sitios o propiedades no deben coincidir porque los usuarios y las sesiones no son aditivos (en marzo de 2026, unir 15 propiedades dio 1.88M usuarios contra 2.27M sumados, −17%).

---

## 5. Diseño del histórico unificado (propuesta del asistente, POR VALIDAR)

- **Qué es.** Una tabla (o vista) con todos los eventos de los datasets elegidos, con columna `origen` (property ID del dataset de procedencia), partida por fecha y agrupada por `origen` y `hostname`.
- **Duplicados por hostname (CONFIRMADO en el inventario).** Un mismo hostname alimenta varias propiedades: por ejemplo `bookingscancunpr.moonpalace.com` llega a 324679071, 268272953 y 324768211, y de 96 hostnames de marzo 23 mandaban a 2 o más propiedades. Unir todo sin regla cuenta los eventos varias veces. Solución: tabla `mapa_hostname_fuente` con UNA fuente por hostname; el unificado solo toma los eventos de ese hostname desde su dataset fuente.
- **`user_pseudo_id` compartido.** El mismo ID aparece en varias propiedades (99.6 a 100% en Baglioni/Florencia), lo que permite reconstruir recorridos entre sitios en la tabla unificada, siempre con la regla de la fuente única.
- **Ubicación.** Los datasets de `us-central1` se pueden unir en una sola consulta. Los 10 de `US` son todos de booking de Baglioni; opciones: (a) usar General Baglioni (`us-central1`) como fuente de booking de Baglioni, si la diferencia es la observada (0.2%); (b) copiar los datasets de `US` a `us-central1`. Decisión pendiente.
- **Tamaño.** 15 propiedades de `us-central1` de marzo 2026: 35.6M eventos al mes; histórico de 870M eventos en 1,267 días; pico diario 1.65M; 2026 ≈ 351M eventos al año. Cabe sin problema en BigQuery (CONFIRMADO en el análisis previo).
- **Esquema.** El export de GA4 tiene el mismo esquema en todas las propiedades, así que se puede unir con `UNION ALL`. Los nombres de parámetros no son consistentes entre sitios: `convertion_rate` / `conversion_rate`, `agancy_id`, `totalAdults` / `total_adults` / `total_guests`, `payment_type` / `paymentType`, 13 claves con texto en unos y número en otros. Hay que documentar un catálogo único de eventos y parámetros.
- **Privacidad.** Quitar de `event_params` las claves `hash_email` y `user_login` hasta validarlo con legal.
- **Escritura.** Solo en un dataset dedicado, nunca en `analytics_*`.
- **Límite de datos de origen.** Los pings sin ID (sin consentimiento) llegan sin `user_pseudo_id`: aparecen en conteos de eventos, no en usuarios ni sesiones. En Baglioni son ~19% de los eventos (GDPR).

---

## 6. Qué implica el "día de corte" (Tarea 3)

Lista de verificación previa (POR VALIDAR en cada punto):
- Mapa hostname → propiedad destino y qué hostnames hoy envían a 2 o más propiedades (23 de 96 en marzo).
- **Cross-domain:** el límite de GA4 es 100 condiciones; en marzo había 96 hostnames. Hay poco margen.
- **Parámetros:** 145 claves de parámetro en 7 días contra 125 dimensiones personalizadas registrables (Google). Priorizar cuáles se registran.
- **Límite de eventos del contrato 360:** no visto. Pedir el límite mensual y el límite de export diario (20 mil millones).
- **Reporting Identity** de cada propiedad (General Palace, General Baglioni, Palazzo, General Booking).
- **Consent Mode / Cookiebot:** modo (básico o avanzado), versión y fechas de cambios. Europa necesita validarse por separado.
- **Eventos clave, audiencias y dimensiones personalizadas** de cada propiedad: migrarlos o redefinirlos.
- **Zona horaria y moneda por propiedad.** La de General Palace es UTC−6.
- **Medir en paralelo** durante 4 a 8 semanas antes de bajar los IDs viejos (propuesta).
- **Plan de reversión.** Cómo volver a los IDs anteriores si algo falla; cuánto tiempo se conservan.
- **Qué podría dejar de funcionar:** reportes de Looker y Data Studio ligados a propiedades individuales, audiencias de Google Ads y Campaign Manager, conversiones importadas, integraciones con afiliados y medios de terceros, filtros por hostname.
- Los datos de la propiedad nueva no son retroactivos: el histórico previo vive en el unificado de BigQuery, no dentro de GA4 (Measurement Protocol tiene una ventana de 72 h).

---

## 7. Decisiones tomadas y pendientes (de la usuaria)

**Tomada (26-oct-2026):** la validación de Total users de booking contra Looker se hace SIN el filtro "Sin admin-bengine"; los datos con ese filtro no se usan. Este filtro solo existe en Looker: no afecta al histórico unificado en BigQuery.

**Pendientes:**

1. Fuente de booking de América en el unificado: General Palace o General Booking (Tarea 1).
2. Umbrales de aceptación (verde / amarillo / rojo) para la diferencia en cada métrica, ANTES de ver resultados.
3. Fuente de verdad para reservas e ingresos (sistema de reservas vs GA4).
4. Incluir o no los pings sin ID en métricas de eventos.
5. Booking de Baglioni en `US`: copiar o usar General Baglioni.
6. Nombre de la propiedad conglomerado y del dataset de pruebas.
7. Fecha del día de corte y duración del periodo en paralelo.
8. Si Looker debe incluir los motores de reserva (hoy caen en "No properties").
9. Zona horaria y definición de mes para todo el agente.
10. Quién mantiene el diccionario de métricas y la tabla de estado de validación.
11. Qué datasets quedan fuera del alcance: los 4 casi inactivos (282021628, 290285685, 355522484 y 358235082), 374573695 (City32 / Grupo Camino Real, fuera de marca) y otros a definir.
12. Si se piensa implementar un banner de cookies en los motores de reserva y cuándo (cambia la brecha de booking; fijar antes la línea base).
13. Alcance de la fase 5 (Campaign Manager y otras fuentes): qué fuentes, en qué orden y cuál es la fuente de verdad de ventas.
14. Confirmar el alcance de la Tarea 1: par de América (General Palace vs General Booking) primero y par de Baglioni (General Baglioni vs All Bookings) después, o ambos a la vez.

## 8. Riesgos principales

| Riesgo | Efecto | Mitigación propuesta |
|---|---|---|
| Hostnames duplicados entre propiedades | El unificado cuenta eventos varias veces | Mapa hostname → fuente única; pruebas por alcance |
| Límite de 100 condiciones cross-domain | No caben todos los hostnames | Inventario previo; reducir hostnames de pago y auxiliares |
| Parámetros sin registrar en la propiedad nueva | Se pierde información en los reportes de GA4 (en BigQuery sí llegan) | Catálogo único y registro por prioridad |
| Consentimiento distinto por región | La brecha con Looker cambia (27–37% en contenido de Baglioni vs ~7% en América) | Validar Europa por separado; no aplicar factores de América |
| Corte sin periodo en paralelo | No hay forma de comprobar equivalencia | 4–8 semanas en paralelo y reversión probada |
| Costos de BigQuery | Consultas grandes sin control | Dry run, tope de bytes, vistas particionadas |
| Datos personales en parámetros | Riesgo legal | Excluir `hash_email` y `user_login`; validar con legal |
| Banner de cookies que se agregue después en booking | Pérdida de datos nueva en booking: la brecha, hoy ~0.1–1.3%, subiría | Fijar ahora la línea base (pings sin ID sobre usuarios y brecha); regla de deriva de ~2 puntos; avisar antes de implementarlo |
