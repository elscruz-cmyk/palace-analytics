# Cómo cargar el proyecto "Palace Analytics / Medición unificada" (léeme primero)

Este paquete reemplaza al "kit de arranque" anterior y reúne lo trabajado en las conversaciones de validación GA4 vs BigQuery, más el plan nuevo: propiedad GA4 unificada, histórico unificado en BigQuery y Palace Analytics. Fecha de corte: 26-oct-2026.

## 1. Orden de carga

| Paso | Qué hacer | Archivo |
|---|---|---|
| 1 | Crea el proyecto (nombre sugerido: "Palace Analytics - Medición unificada") | n/a |
| 2 | Pega el contenido en **Instrucciones del proyecto** (en Claude Code, úsalo como `CLAUDE.md`) | `01_instrucciones_del_proyecto.md` |
| 3 | Súbelo como conocimiento del proyecto: visión, plan, decisiones pendientes | `02_vision_plan_y_decisiones.md` |
| 4 | Súbelo: inventario de los 61 datasets, nombres inferidos, hostnames, huecos | `03_inventario_datasets_y_propiedades.md` |
| 5 | Súbelo: reglas, validaciones y hallazgos de datos | `04_conocimiento_de_datos_y_validaciones.md` |
| 6 | Súbelo: plantillas SQL (incluye las de la unificación) | `05_plantillas_sql.md` |
| 7 | Súbelo: datos de la validación de Total users (36 filas) | `06_datos_validacion_total_users.csv` |
| 7b | Súbelo: texto de la presentación de viabilidad del 14-oct (el artifact, en formato que el proyecto puede leer), con notas de vigencia | `07_presentacion_viabilidad_2026-10-14.md` |
| 8 | Sube lo que solo tú tienes (sección 3), conforme lo reúnas | tus documentos |
| 9 | Abre el primer chat del proyecto y pega el mensaje de inicio (sección 2) | n/a |

**Qué reemplaza de la numeración anterior:** el 02 anterior (contexto del ecosistema) y el 03 anterior (validación de Total users) quedaron integrados en el 04 nuevo; el 05 anterior (plantillas) quedó integrado y ampliado en el 05 nuevo; el CSV 04 anterior es ahora el 06. La presentación de viabilidad del 14-oct está en https://claude.ai/artifact/4gQ1oFStAoV44bzAuK17Ad y su contenido ya viene como archivo 07 (con notas de vigencia). Hay tres correcciones pendientes en el artifact: etiqueta de la lámina del filtro admin-bengine, cifras de 14 sitios y los 300 USD por confirmar.

## 2. Mensaje de inicio (pégalo en el primer chat)

```
Contexto: este proyecto tiene dos frentes. (1) Infraestructura de medición unificada: pasar de decenas de propiedades GA4 a UNA propiedad conglomerado, con un histórico unificado en BigQuery (pr-mktg-analyt-prod). (2) Palace Analytics: agente de datos y reportes interactivos sobre BigQuery, con las mismas definiciones de Looker. Lee las instrucciones del proyecto y los archivos 02 a 07 antes de responder.

Visión de Palace Analytics: plataforma tipo GA4 adaptada a nuestro contexto, con un agente para todas las áreas; supera los límites de Looker (tablas, cruces, filtros como admin-bengine que cambian la medición, ver el viaje por user_pseudo_id). Después se ligará con Campaign Manager y otras fuentes (ventas reales, atribución).

Estado: inventario de los 61 datasets hecho (archivo 03, nombres inferidos por hostname). Total users validado en contenido y booking de América y Baglioni (archivo 04). Brecha de BigQuery vs Looker por alcance (archivo 02, sección 0): Total users América contenido ~7%, booking 0.1–0.4%, Baglioni contenido 27–37%, Baglioni booking ±1.2%. Causa: INFERENCIA fuerte (consentimiento y modelado de GA4). En booking casi no hay brecha porque no hay banner de cookies; si se implementa uno, habrá pérdida.
Siguiente paso, antes de unificar: diferencia entre booking en General Palace y booking en General Booking (América, 324768211; el par de Baglioni es General Baglioni vs All Bookings, 358249801). Luego pruebas de unificación y día de corte.

Hoy quiero (elige uno):
A) Tarea 1, paso 1: matriz de cobertura de hostnames de booking entre General Palace y General Booking.
B) Tarea 1, paso 2: diferencias por mes y hostname (usuarios, sesiones, eventos, compras) entre las dos.
C) Tarea 1, paso 3: cruce de compras por transaction_id entre las dos.
D) Fijar umbrales de aceptación (verde / amarillo / rojo) ANTES de ver resultados.
E) Definir el mapa hostname → dataset fuente y diseñar la prueba pequeña de unificación.
F) Regenerar por código las tablas de booking América y Baglioni que hoy solo existen como resumen.

Primero confirma la última tabla disponible de cada dataset (list_table_ids), haz dry run antes de escanear y declara fuente, rango, filtros, frescura y estado de validación.
Reglas: brecha = 1 − BigQuery / Looker; no mostrar user_pseudo_id; no aplicar factores correctores; no escribir en los datasets analytics_*; generar tablas por código; separar CONFIRMADO / INFERENCIA / POR VALIDAR.
```

Recomendación: empezar por **D y luego A**. Sin umbrales fijados antes, ningún veredicto de la Tarea 1 o de las pruebas queda con criterio; A es barata (dry run ≈ 0.2 GB por día) y define qué comparar en B y C.

## 3. Lo que solo tú puedes aportar (por prioridad)

### Prioridad 1: sin esto no se avanza bien
- [ ] **Nombres exactos de las propiedades de GA4** (GA4 Admin o el Excel `Propiedades_GA4`): propiedad → Property ID → dataset → hostnames → marca. Los 4 IDs del Excel sin dataset (374580462, 511001955, 474508560, 403515589) no existen en BigQuery: confirma si se dieron de baja o no exportan.
- [ ] **Decisión de umbrales de aceptación** (archivo 02, sección 7) y la **fuente de verdad** para reservas e ingresos.
- [ ] **Mapa de contenedores de GTM y IDs de medición por sitio** (qué contenedor envía a qué propiedad). Es la base del día de corte.
- [ ] **Configuración de cross-domain actual** por propiedad (lista de dominios y condiciones; el límite es 100).
- [ ] **Catálogo de eventos y parámetros** estándar y personalizados, y cuáles son eventos clave/conversiones en cada propiedad.
- [ ] **Reporting Identity** de General Palace, General Baglioni, Palazzo y General Booking (GA4 Admin → Reporting Identity).
- [ ] **Límite de eventos al mes del contrato 360** y límite de export diario.
- [ ] **Diccionario de métricas de Looker**, todos los campos calculados del reporte (ya tenemos "Sitio web"; faltan canal, tipo de campaña, medios) y la configuración de filtros y controles.

### Prioridad 2
- [ ] Qué es "MVG" (datasets 282021628, 290285685 y 387217775) y si los 4 datasets casi inactivos salen del alcance.
- [ ] Configuración de consentimiento: Consent Mode básico o avanzado, versiones de Cookiebot y fechas de cambios. Historial de versiones de GTM publicadas (explica saltos como el de pings de Jamaica en julio y Cozumel en junio).
- [ ] Zona horaria y moneda por propiedad.
- [ ] Convención de campañas y canales (`utm_campaign`, agrupación de canales, Black Friday y Cyber Monday, SEO).
- [ ] 10 preguntas reales por área (Inbound, Social, Planning) con respuesta esperada, y 15 a 20 "números dorados" verificados en Looker.
- [ ] Extracto del sistema de reservas por `transaction_id`, sin datos personales.
- [ ] Postura de legal sobre los parámetros `hash_email` y `user_login`.
- [ ] Tablas de Looker de booking América y de Baglioni, **SIN** el filtro "Sin admin-bengine", para regenerar la validación (las cifras con filtro solo se necesitan si quieres actualizar la lámina de la presentación).

### Prioridad 3
- [ ] Roles y permisos de BigQuery del agente (solo descripción, nunca credenciales); topes de costo (presupuesto mensual y bytes máximos por consulta).
- [ ] Formato de respuesta preferido por área.
- [ ] Nombre de la propiedad conglomerado, del dataset de pruebas y fecha tentativa del día de corte.

**No subas:** contraseñas, llaves, tokens, credenciales, datos personales de huéspedes, correos ni teléfonos.

## 4. Si lo vas a trabajar en Claude Code

- Usa el contenido de `01_instrucciones_del_proyecto.md` como `CLAUDE.md` y deja los archivos 02 a 06 en una carpeta (por ejemplo `docs/`) que el CLAUDE.md referencie.
- Acceso a BigQuery: con el conector MCP de BigQuery (`execute_sql_readonly` con `dryRun: true` primero) o con `bq` / `gcloud` en tu máquina. En la sesión anterior `bq` 2.1.32 falló con un error interno al hacer dry run; es POR VALIDAR en tu entorno.
- Cuenta de solo lectura sobre `pr-mktg-analyt-prod`, más permiso de escritura únicamente en el dataset de pruebas (cuando lo crees).
- Guarda en el repositorio cada tabla de validación regenerada y actualiza el estado en el archivo 04.

## 5. Pendientes técnicos y huecos conocidos (resumen; detalle en 02 y 04)

1. Matriz de cobertura de hostnames de booking (qué llega a General Palace y a General Booking).
2. Pings: 7.8–11% (América) vs 27–37% (Baglioni contenido); saltos de Cozumel (jun), Jamaica (jul), Baglioni Brand (abril).
3. Datasets detenidos desde el 9-jun-2026 (355522484 y 358235082) y con cientos de días sin tabla (283039175, 283045662).
4. Booking de Baglioni en `US`: copiar o usar General Baglioni como fuente.
5. Validación pendiente: sesiones de booking, page views de 11 sitios, active users, key events, ingresos (moneda), campañas, canales y dimensiones.
6. 5 eventos `purchase` sin `transaction_id`; `transaction_id` perdido en Beach (16-abr a 31-may); `payment_type`/`paymentType`; contenido de `ga_cookie`.
7. Octubre: decidir tratamiento al cerrar (1-nov).

*Análisis propio, no es una revisión certificada. Si respalda decisiones formales, validar con una persona de estadística y con la documentación oficial de Google.*
