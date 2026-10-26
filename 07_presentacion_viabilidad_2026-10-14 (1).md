# Presentación de viabilidad: BigQuery como base del agente de datos y de los reportes interactivos

Versión en texto del artifact publicado el 14-oct-2026 (10 láminas), para que el proyecto la tenga como contexto.
Artifact original: https://claude.ai/artifact/4gQ1oFStAoV44bzAuK17Ad (versión leída el 26-oct-2026; si el artifact cambia, regenerar este archivo).
Autoría: Customer Insights. Las cifras son las de la presentación; las diferencias con el estado actual están en la sección "Notas de vigencia" al final.

---

## Lámina 1. Portada
**BigQuery como base del agente de datos y de los reportes interactivos**
Viabilidad, diferencias de datos, costo y plan de trabajo. Customer Insights · 14 de octubre de 2026.

## Lámina 2. Objetivo
**Partimos de Looker y vamos a un análisis a la medida de nuestro contexto.**
- Objetivo: usar BigQuery como fuente de un agente de datos y de reportes interactivos, con las mismas definiciones de Looker.
- Hoy: el análisis de tráfico del sitio se hace con reportes de Looker, ya filtrados y configurados.
- Ahora queremos ir más lejos: un agente de conversación y un análisis profundo, adecuado a nuestras necesidades de medición: comportamiento de usuario, insights, datos limpios y pérdida de datos clara.

## Lámina 3. Diferencias de datos que tenemos que tener claras
Brecha de Total users entre BigQuery y Looker:

| Alcance | Diferencia |
|---|---|
| América, contenido | 7.4% |
| América, booking | ±1.5% |
| Baglioni, contenido | 35.7% |
| Baglioni, booking | ±1.2% |

Qué causa la diferencia:
- Quien no acepta cookies llega como un ping sin ID: BigQuery lo ve como evento, no como usuario ni sesión.
- Looker (GA4) estima a esos usuarios con modelado de comportamiento; BigQuery solo trae lo que se observó.
- A más pings sin ID, más diferencia. En booking casi no hay pings, por eso casi no hay diferencia.

## Lámina 4. En los sitios de contenido hay más diferencia de datos; en booking, casi no
Diferencia por marca y sitio (Total users, %):

**América**
- Contenido, Palace Resorts: Beach Palace 6.4; Cozumel Palace 8.6; Sun Palace 7.7; promedio Palace Resorts 7.6.
- Contenido, Moon Palace: Jamaica 6.8.
- Contenido, Le Blanc: pendiente.
- Promedio de contenido (4 sitios medidos): 7.4.
- Booking: usuarios (14 de 14 grupos) ±1.5.

**Baglioni**
- Contenido: Venice 26.6; Sardinia 30.8; Rome 30.7; Maldives 32.5; Milan 34.9; Puglia 36.5; Florence 36.4; Brand 56.9 (el valor atípico); promedio de 8 sitios 35.7.
- Booking: usuarios ±1.2; sesiones ±2.9.

Qué podemos hacer hoy:
- Los datos son reales: BigQuery cuenta lo que ocurrió con usuarios identificados.
- Cada usuario con identificador es observado, no estimado, y permite seguir su comportamiento durante toda su interacción en el sitio.

Qué falta:
- Replicar en BigQuery los filtros, campos calculados y métricas de los reportes actuales de Looker Studio.
- Documentar el contexto de cómo leemos los datos.

## Lámina 5. Qué ganamos con BigQuery
- **Datos crudos de cada evento:** se puede seguir todo lo que hizo cada usuario identificado.
- **Libertad para crear reportes:** a nuestra medida y contexto, sin quedar limitados a las tablas y gráficas de Looker.
- **Consulta de bajo costo:** Google Cloud cobra BigQuery por datos leídos: 6.25 USD por TiB, y el primer TiB al mes es gratis. Las consultas leyeron en promedio 5.6 GiB: unos 0.03 USD.
- **Desventaja:** brecha de datos contra Looker en contenido: 7.4% en América y 35.7% en Baglioni.

Tenemos claro qué medimos y qué se pierde:
- **Se mide completo:** los eventos, con o sin usuario. Por ejemplo, `click_to_booking` coincide con Looker en 72 de 72 meses.
- **Se pierde:** Total users, Active users, New users y Sessions de quien no acepta cookies. Los pings sin consentimiento permiten medir esa pérdida.

Lo que obtendremos: comportamiento por tipo de usuario; insights relevantes; datos limpios; porcentaje de pérdida medido.

## Lámina 6. Qué se abre con estos datos (ideas)
1. **Segmentos de comportamiento:** agrupar usuarios por cómo navegan y ver cada segmento de forma visual.
2. **Antes de reservar y sin reservar:** qué páginas y acciones tuvieron quienes reservaron y qué hicieron quienes no.
3. **Intención de reserva:** predecir qué visitantes están por reservar y activarlos en Google Ads.
4. **Pronóstico de demanda:** por hotel y mes de entrada, con estacionalidad; se puede complementar con la demanda de búsqueda de Google (Travel Analytics Center).
5. **Alertas de anomalías:** detectar tráfico basura y cambios de medición antes de que ensucien los reportes.
6. **Por qué cambió una métrica:** identificar qué canal, sitio o dispositivo explica una subida o una bajada.
7. **Índice de calidad de datos:** un semáforo por sitio, con alerta cuando algo se desvía.
8. **Agente que comparte contexto:** pasa datos y contexto a otros agentes para estrategias de campaña.
9. **Cruce con el sistema de reservas:** conciliar lo medido con las reservas reales.

Referencias citadas: Google Cloud (casos de uso de Google Analytics con BigQuery y solución Marketing Analytics Jumpstart: propensión de compra, segmentación y activación en Google Ads); Travel Analytics Center (acceso para socios de Google Travel); funciones de BigQuery ML.

## Lámina 7. El proyecto: un agente y un reporte de comportamiento de usuario
**Brújula:** agente para que otras áreas consulten el tráfico. Preguntas de ejemplo:
- ¿Cuáles fueron las páginas más vistas del mes?
- ¿Qué páginas vieron los usuarios que reservaron antes de hacerlo, y cuáles los que no reservaron?
- ¿Qué canal tiene más reservas concretadas?

Horas por actividad (promedio):

| Actividad | Horas |
|---|---|
| Alineación con Looker y validación de datos | 141 |
| Investigación | 36 |
| Desarrollo del agente | 38 |
| Desarrollo del reporte de comportamiento de usuario | 26 |
| Pruebas | 22 |
| Documentación y capacitación | 10 |
| **Total** | **≈ 273** |

## Lámina 8. Cuánto costaría desarrollarlo y operarlo
**Desarrollo (una vez): Claude.** Con "tus 300 USD de créditos mensuales" (supuesto por confirmar, ver notas):

| Modelo | Costo estimado |
|---|---|
| Sonnet 5.5 | ≈ 68 USD |
| Opus 5.5 | ≈ 137 USD |
| Fable 5.1 | ≈ 341 USD |

Alcanza con Sonnet 5.5 u Opus 5.5; con Fable 5.1 no. BigQuery durante el desarrollo: 4–21 USD (consultas de construcción y pruebas).

**Operación al mes: consultas al agente**

| Preguntas al mes | BigQuery | Claude (Sonnet 5.5) |
|---|---|---|
| 200 | ≈ 7 USD | ≈ 18 USD |
| 500 | ≈ 28 USD | ≈ 45 USD |
| 1,000 | ≈ 62 USD | ≈ 90 USD |

**Operación al mes: reporte de comportamiento de usuario (solo BigQuery):** ≈ 1 USD con actualización semanal; ≈ 5 USD con actualización diaria.

Referencias de cálculo: 50 mil tokens de entrada y 15 mil de salida por hora de trabajo; por pregunta al agente, 2 consultas a BigQuery (5.6 GiB cada una) y 30 mil tokens de entrada y 3 mil de salida; cada actualización del reporte lee ≈ 27 GiB.

## Lámina 9. Qué tener presente y por dónde empezar
Consideraciones:
- Tener clara la diferencia de datos.
- Monitorear los costos.
- Fijar umbrales de aceptación de diferencia de datos antes de ver resultados.

Siguientes pasos:
1. Definir umbrales.
2. Replicar en BigQuery las métricas, dimensiones, filtros y campos calculados de Looker.

## Lámina 10. Para tu información: el filtro "Sin admin-bengine" infla los datos de usuarios en Looker
**Looker con el filtro activo: Total users** (motor de reserva, septiembre 2026, Beach Palace, América):

| Medición | Total users |
|---|---|
| Looker sin filtro | 6,142 |
| Looker con filtro | 7,345 |
| BigQuery | 6,200 |
| Sin filtro vs BigQuery | −58 (−0.9%) |
| Con filtro vs BigQuery | +1,145 (+15.6%) (ver nota de etiqueta) |

**BigQuery: usuarios con un evento de admin-bengine** (enero a septiembre de 2026):
- América: 56 usuarios de 8,131,084 (0.0007%); 885 eventos.
- Baglioni: 25 usuarios de 314,241 (0.008%); 73 eventos.
- Al aplicar el filtro en BigQuery, América pasa de 8,131,084 a 8,131,075 usuarios (−9) y Baglioni de 314,241 a 314,240 (−1). En septiembre: América −1 y Baglioni 0.

**Por qué pasa:** al agregar un filtro de Page path, Looker cuenta a cada persona una vez por cada página distinta que vio, en lugar de una sola vez. Ejemplo real, Beach Palace, 15 de septiembre:

| Páginas que vio | Usuarios | Looker los cuenta |
|---|---|---|
| 1 | 191 | 191 |
| 2 | 45 | 90 |
| 3 | 4 | 12 |
| **Total** | **240** | **293** |

Sin el filtro, Looker y BigQuery coinciden: diferencia de 0.1% a 0.4%.

---

## Notas de vigencia (agregadas el 26-oct-2026; no forman parte de la presentación original)

1. **Etiqueta de la lámina 10.** "+1,145 (+15.6%)" mezcla bases: 1,145 sobre los 6,200 de BigQuery es +18.5%. El 15.6% sale de 1 − 6,200/7,345, que es la convención del proyecto (BigQuery queda 15.6% debajo de Looker con el filtro). Usar una sola base.
2. **Cifras de Total users de las láminas 3 y 4.** Al 26-oct el traspaso ya reporta contenido de América en 14 sitios con 6.4–8.6% (8.2% con Palace Resorts Brand), y booking de América en 0.1–0.4% (máximo 1.3%) sin el filtro. La presentación sigue mostrando 4 sitios medidos, Le Blanc "pendiente" y ±1.5% en booking. Esas cifras nuevas vienen de un resumen de chat sin tabla de respaldo: regenerar por código antes de citarlas.
3. **Decisión del 26-oct sobre el filtro.** Los datos de Looker con el filtro "Sin admin-bengine" ya no se usan para validar ni reconciliar. La lámina 10 se mantiene solo como explicación de por qué no se usan.
4. **Créditos y costos.** Los "300 USD de créditos mensuales" son un supuesto por confirmar (y por confirmar que sean USD), igual que los supuestos de horas y tokens. Los topes de gasto de BigQuery se explicaron pero no están en la presentación.
5. **Alcance.** La presentación se escribió antes de definir la propiedad unificada. Para ese frente (propiedad conglomerado, histórico unificado, día de corte) ver el archivo 02; la presentación cubre la parte de Palace Analytics (agente y reportes).
