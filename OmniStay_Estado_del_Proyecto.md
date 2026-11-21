# OmniStay — Estado del proyecto

> **Fuente de verdad:** `OmnistayV3.pdf` (v2 sin el Gemelo Digital + plan de implementación).
> Última actualización: 27 de octubre de 2026.

**Plataforma inteligente de gestión hotelera con BPM, CRM del huésped e inteligencia de insumos.**
Anteproyecto de titulación — CETI, materia Proyecto I (Prof. Carlos Darío Arenas Yerena).
Equipo: Isaac Alejandro Carrascal Cornejo · Mayra Alejandra Sánchez Gutiérrez.

---

## 1. Qué es OmniStay

Plataforma web multi-hotel para **hoteles boutique y cadenas pequeñas de México y Latinoamérica (20 a 200 habitaciones)** que une en un solo sistema:

- la automatización de los procesos operativos (BPM),
- el conocimiento del huésped (CRM),
- la atención omnicanal (link web, WhatsApp, kiosco),
- un motor de IA que **pronostica el consumo de insumos** según temporada, ocupación y afluencia, y dice cuándo y cuánto pedir,
- indicadores y revenue management para el gerente.

**Problema que ataca:** los hoteles PyME operan con herramientas dispersas (Excel, reservas aisladas, WhatsApp manual), fijan tarifas y compran insumos por intuición, y las soluciones enterprise (p. ej. Oracle OPERA, desde 8,400 USD/año solo el PMS) están fuera de su presupuesto.

**Diferenciador:** llevar a un hotel pequeño una inteligencia de insumos (pronóstico, reabastecimiento y control de merma) que hoy solo existe, de forma parcial, en sistemas enterprise — con técnicas interpretables y el gerente siempre aprobando las decisiones.

---

## 2. Lo que se ha hecho

### Documentos en la carpeta

**Carpeta raíz (versiones actuales)**

| Archivo | Contenido | Estado |
|---|---|---|
| `OmnistayV3.pdf` | Documento técnico v3 (40 págs., mismo formato que la v2): 7 módulos sin Gemelo Digital (incluye **7. Distribución y facturación**: channel manager propio, motor de reservas y CFDI 4.0), valor agregado centrado en inteligencia de insumos, pronóstico de ingresos y margen de contribución (módulo 5, Bloque 3) y **sección 8. Plan de implementación** (objetivos, fases, dependencias, datos, indicadores, **resultados esperados**, alcance, cambios vs. v2). | **Fuente de verdad.** |
| `Protocolo_Investigacion_OmniStay.docx` | Protocolo, parte 1: **1. Descripción del problema** (situación y causas, magnitud con datos cuantitativos, alcance cualitativo, fuentes). | Terminado. El subtítulo aún dice "CRM predictivo y Gemelo Digital Operativo". |
| `Protocolo_Investigacion_OmniStay_Parte2.docx` | Protocolo, parte 2: **2. Objetivos** (general y 7 específicos derivados del problema) y **3. Propuesta de solución** (incluye módulo 7 y **3.8 Resultados esperados**, Tablas 6–8). | Terminado (borrador para revisión). |
| `Protocolo_Investigacion_OmniStay_Parte3.docx` | Protocolo, parte 3: **4. Antecedentes técnicos** (OPERA, Cloudbeds, Mews, Little Hotelier, Hotelogix, eZee, Pxsol, Arpón, SiHoteles, Zavia + herramientas RoomPriceGenie, Asksuite, Apicbase, BirchStreet, Winnow; tablas de costo/segmento/funcionalidades; OmniStay vs. antecedentes) y **5. Estudios sobre el problema** + **Referencias** (APA). Tablas 9–15; incluye **4.7 Resultados esperados de OmniStay frente a los antecedentes**. | Terminado (borrador para revisión). Precios consultados en sep. 2026. |
| `Protocolo_OmniStay_Puntos_1-5.docx` | **Protocolo consolidado, puntos 1–5 del "Formato para la Presentación de Proyectos"** (27 págs.): portada (Ing. en Desarrollo de Software, registros, grupo 7N, asesores técnicos Prof. José Francisco Pérez Reyes y Prof. Alma Nayeli Rodríguez Vázquez), índice, 1. Descripción del problema (+1.4 bases y supuestos), 2. Antecedentes técnicos y referencias, 3. Objetivos con metas asociadas (por OE y por etapa), 4. Metodología (solución, fases, diagnóstico por visitas, hotel piloto, evaluación), 5. Contribución, Referencias APA. Tablas 1–18. | Terminado (borrador para revisión). Integra las tres partes anteriores. |
| `OmniStay_Estado_del_Proyecto.md` | Este resumen. | Vigente. |

**`Historico/` (referencias anteriores)**

| Archivo | Contenido |
|---|---|
| `Propuesta_de_Proyecto.pdf` | Propuesta formal entregada (6 págs.). Aún menciona CRM predictivo y Gemelo Digital. |
| `OmniStayOG.pdf` | Borrador original (24 págs.) con CRM predictivo, analítica predictiva y Gemelo Digital. |
| `OmniStay_v2_insumos.pdf` | Borrador v2 (33 págs.): IA enfocada en insumos, CRM por reglas; incluía el Gemelo Digital. Base de la v3. |

### Decisiones tomadas

1. **Se elimina el Gemelo Digital Operativo** (simulación con SimPy). El estado en tiempo real se conserva, pero como parte del BPM (Redis + WebSockets), no como módulo independiente.
2. **Se eliminan los modelos predictivos del CRM** (probabilidad de cancelación, lifetime value, clustering). El CRM trabaja con datos reales, indicadores calculados y segmentación RFM por reglas.
3. **La IA se concentra en insumos**: el consumo se genera a diario en cada habitación y cada orden, así que un hotel pequeño acumula datos suficientes en pocos meses; su impacto se mide en pesos.
4. **El sistema sugiere, el gerente decide**: ni compras ni cambios de tarifa sin aprobación humana.
5. **Sin IoT ni sensores**: mantenimiento correctivo y preventivo por calendario.
6. **Ocupación proyectada sin ML**: reservas confirmadas (on the books) + pickup histórico.
7. **Pronóstico de ingresos y margen de contribución del mes** (sí con IA): se queda en ingresos y margen de contribución; **no** se calcula utilidad neta porque nómina, servicios y costos fijos no se registran en la plataforma.
8. **Diagnóstico de OE1 mediante visitas** a hoteles independientes del segmento en Jalisco (observación de procesos, herramientas, integración y forma de decidir tarifas y compras).
9. **Hotel piloto** como fuente de datos para entrenar y evaluar los modelos (historial + consumo capturado con la plataforma). Limitación: resultados de un solo hotel.

---

## 3. Módulos del proyecto

### 3.1 BPM inteligente
Digitaliza y automatiza los cuatro macroprocesos.

- **Front Office** — Correo 24 h antes con link a la PWA para pre check-in (datos, términos, preferencias). Dashboard del recepcionista con llegadas, plano de habitaciones y solicitudes. Asignación de habitación con optimizador (preferencias, rotación de desgaste, rentabilidad). Solicitudes por WhatsApp/PWA clasificadas por intención → ticket o respuesta automática (Claude API). Check-out que consolida consumos, factura y envía por email.
- **Housekeeping** — Lista automática de habitaciones a limpiar; algoritmo greedy que reparte entre camareras (salidas primero, agrupado por piso, carga máxima). Tarjeta por habitación en el celular (en limpieza → lista para inspección), reporte de incidencias con foto que genera ticket de mantenimiento. **Kit de reposición precargado con botones +/−** (fuente de datos de amenities y blancos). Inspección de la supervisora y liberación en tiempo real (WebSockets).
- **Mantenimiento** — Correctivo (desde huésped, staff o recepción) con prioridad por reglas configurables y notificación push al técnico con la habilidad requerida. Preventivo por catálogo de equipos y frecuencias. Cierre con tiempo y materiales; analítica de fallas sin sensores. Refacciones descontadas del inventario y reabastecidas por el motor de insumos.
- **F&B** — Pedido en tablet/celular del mesero → Kitchen Display System (PWA). **Recetas predefinidas** que descuentan ingredientes por cada orden (stock en tiempo real, ocultar platillos sin stock, costo y margen por plato). Room service desde el link del huésped con cargo a la habitación. BEO digital para banquetes y eventos.
- **Hilo conductor:** estado de cada habitación en Redis (disponible, ocupada, en limpieza, bloqueada, con OT) propagado por WebSocket a todos los clientes.

### 3.2 CRM del huésped
- Perfil 360° construido **a partir de eventos**, sin formularios: canal de reserva, antelación, tarifa, solicitudes, consumos, hora de check-out, quejas, calificación de la OTA.
- Integración con OTAs (Booking.com, Expedia) por webhooks; unificación del huésped por email.
- Perfil materializado en la tabla `guests`, recalculado por un worker de Celery tras cada checkout.
- **Segmentación RFM por reglas** (umbrales configurables por hotel):

| Segmento | Regla de ejemplo | Tratamiento |
|---|---|---|
| A | ≥ 3 estancias en 24 meses y gasto > P75 | VIP: bienvenida personalizada, prioridad de upgrade, alerta en recepción |
| B | ≥ 2 estancias en 24 meses | Huésped frecuente, ofertas de regreso |
| C | 1 estancia en 12 meses | Seguimiento post-estadía, invitación a reservar directo |
| D | 1 visita hace > 12 meses | Comunicación de bajo costo |

- **Motor de reglas** (`crm_rules`: disparador, condición JSONB, acción, plantilla) para acciones automáticas: WhatsApp de bienvenida, oferta de regreso a los 90 días, bandera de upgrade.
- Alimenta al pronóstico de insumos con preferencias de llegadas próximas (almohada hipoalergénica, cuna, toallas extra, desayuno sin gluten) y la mezcla de huéspedes esperada.

### 3.3 Inteligencia de insumos (núcleo predictivo)
Responde cada día: qué insumos importan más en esta temporada, cuánto se va a consumir y cuándo y cuánto pedir.

**Insumos cubiertos:** amenities, blancos, alimentos, bebidas, limpieza y lavandería, refacciones.

**Captura del consumo real** (sin fricción, dentro del BPM): recetas del KDS, kit de reposición de housekeeping, movimientos de lavandería (enviadas/recibidas/bajas), entradas y salidas de almacén con conteo cíclico semanal, refacciones por OT.

**Variables del modelo:** ocupación proyectada, huéspedes-noche (adultos/niños), llegadas y salidas, estancia promedio, mezcla de huéspedes, temporada/día de la semana/feriados, eventos y grupos, clima (API), menú vigente y desayuno incluido.

**Pipeline:**

| Paso | Qué hace | Técnica |
|---|---|---|
| 1. Insumos clave por temporada | Prioriza por valor y predictibilidad | ABC-XYZ por temporada + descomposición STL |
| 2. Pronóstico (14 días) | Amenities y desayunos | Regresión de Poisson (GLM) con huéspedes-noche como exposición |
| | Alimentos y bebidas de alta rotación | LightGBM cuantílico global (P50/P75/P90) + explosión de receta |
| | Bebidas de baja rotación, refacciones | Croston / TSB |
| | Blancos y toallas | Cálculo de lavado + tasa de reúso; nivel par + Poisson de bajas por ciclo |
| | Químicos de limpieza y lavandería | Regresión lineal sobre kg lavados y limpiezas |
| | Explicación | SHAP sobre LightGBM → mensajes en lenguaje natural |
| 3. Reabastecimiento | Cuándo pedir | Punto de reorden: PR = DL + z·σ·√L |
| | Cuánto de perecederos | Newsvendor, cuantil crítico Cu/(Cu+Co) |
| | Orden por proveedor | Programación entera con OR-Tools (mínimos, entregas, presupuesto, espacio) |
| 4. Merma | Consumo real vs. esperado | Residuos > 2σ por varios días → alerta; conteo cíclico vs. stock registrado |

**Arranque en frío:** día 1 reglas estándar (mín./máx.) → primeras semanas modelo global multi-hotel → ~6 meses modelo propio (si tiene menor error).
**MLOps básico:** reentrenamiento semanal, validación temporal, métricas WAPE, pérdida pinball y sesgo; un modelo solo reemplaza al anterior si mejora.
**Almacenamiento:** `inventory_items`, `stock_movements` (todo movimiento: compra, consumo, merma, baja, ajuste), tabla de consumo diario, `demand_forecasts`, `purchase_suggestions`.
**Vista del gerente:** semáforo de días de cobertura, sugerencias por proveedor para aprobar con un clic, insumos clave de la temporada, gráfica real vs. pronóstico, explicación SHAP ("30% más toallas de alberca porque se pronostican 34 °C y hay puente").

### 3.4 IA Core
- **Optimizador operativo (OR-Tools):** asignación de habitaciones (matching), ruteo de housekeeping (VRP simplificado, recalculado ante cambios), prioridad de órdenes de trabajo (scoring).
- **Motor de pronóstico de insumos:** servicio Python con modelos serializados; job nocturno que actualiza ocupación proyectada, pronostica 14 días, recalcula puntos de reorden y deja listas las sugerencias. Modelos global + por hotel.
- **Asistente conversacional:** clasificador de intención (pregunta informativa, solicitud, queja, saludo/ambiguo) → Claude API con system prompt por hotel para preguntas; `[ESCALAR]` o solicitudes/quejas → ticket en el BPM.
- **Orquestador de alertas:** ejecuta reglas del CRM; anomalías operativas (backlog de solicitudes, tiempos de limpieza +40%, huésped con 3 solicitudes abiertas); alertas de inventario (punto de reorden sin pedido, caducidad próxima, consumo anómalo). Criterio de 2σ.

### 3.5 Analytics & Revenue Management
- **KPIs automáticos:** ocupación, ADR, RevPAR, TRevPAR; lead time, cancelación y costo por reserva por canal; **KPIs de insumos:** costo por habitación ocupada (CPOR), costo de alimentos/bebidas %, rotación de inventario, % de merma, días de cobertura.
- **Ocupación proyectada (30 días):** on the books + pickup histórico (tabla `booking_pace`).
- **Pronóstico de ingresos y margen de contribución del mes (IA):** LightGBM cuantílico global que pronostica el ingreso diario (habitaciones y F&B) de los próximos 90 días con reservas en libros, ADR de lo confirmado, mismo periodo del año anterior, temporada/feriados, eventos, mezcla de canales y F&B por huésped-noche. Cierre del mes = real acumulado + pronóstico de días restantes; rango con el error mensual histórico (no sumando P10/P90 diarios). Margen de contribución = ingreso − insumos − alimentos y bebidas por receta − comisiones de OTA. Explicación SHAP; alerta si el cierre esperado cae bajo el mismo mes del año anterior; tabla `revenue_forecasts`. Evaluación: WAPE mensual vs. referencia "mismo mes del año anterior".
- **Precio dinámico por reglas** (umbrales de ocupación → multiplicador), siempre aprobado por el gerente.
- **Overbooking seguro** con tasas históricas de cancelación y no-show.
- **Reporte diario 8:00** por WhatsApp y email (ocupación, revenue, cancelaciones, incidencias, insumos en punto de reorden); alertas de desviación de KPIs (2σ).
- **Vista multi-hotel** para super admin.
- Métricas pre-calculadas en `daily_metrics` (también alimenta las variables del pronóstico).

### 3.6 Canal huésped omnicanal
- **Link PWA** único por reserva (`/g/:token`, expira checkout + 24 h): pre check-in, servicios, room service, reportes, chat, resumen de consumos y pago. Meta: check-in en < 2 min.
- **WhatsApp Business Cloud API:** bienvenida, recordatorio 24 h antes, conversación con el asistente, encuesta de 1 pregunta post-checkout que alimenta al CRM.
- **Kiosco:** la misma PWA en tablet fullscreen; QR de la reserva, firma digital; llave digital con cerradura electrónica como fase 2.

### 3.7 Distribución y facturación (incluido)
- **Channel manager propio:** inventario único por tipo de habitación y noche en Redis con descuento atómico (cero overbooking por desfase); sincronización ARI con Booking.com, Expedia, Airbnb y Despegar vía sus APIs de conectividad (certificación como socio de conectividad); tarifas aprobadas desde Revenue; bitácora con reintentos (`channel_mappings`, `ari_updates`).
- **Motor de reservas directas** en la PWA, sin comisión, con pasarela de pago.
- **Facturación CFDI 4.0:** datos fiscales (RFC, nombre, régimen, C.P., uso CFDI) en el pre check-in; CFDI del folio al check-out, timbrado por PAC, XML + PDF por email y WhatsApp; factura global y cancelaciones (`invoices`).

### Cómo se conectan
BPM genera los datos operativos → CRM los convierte en conocimiento del huésped → Inteligencia de insumos pronostica consumo y compras → IA Core optimiza decisiones en tiempo real → Canal huésped conecta con el cliente → Distribución vende en todas las OTAs y factura cada estancia → Analytics convierte todo en reportes y decisiones para el gerente.

---

## 4. Arquitectura y stack

| Capa | Tecnología |
|---|---|
| Presentación | PWA en React (una sola base de código: recepción, staff móvil, gerente, huésped, kiosco) |
| API | FastAPI — REST + WebSocket, JWT por rol, rate limiting |
| Módulos de negocio | Servicios FastAPI: Front Office, Housekeeping, Mantenimiento, F&B, Almacén |
| IA | Python: scikit-learn, LightGBM, statsmodels, statsforecast, SHAP, OR-Tools; Claude API |
| Datos | PostgreSQL (reservas, procesos, inventario, historial, usuarios); Redis (estado en tiempo real, notificaciones, sesiones); Celery (tareas de fondo) |
| Integraciones | WhatsApp Business Cloud API, APIs de conectividad de OTAs (Booking, Expedia, Airbnb, Despegar), PAC para CFDI 4.0, pasarela de pago, SendGrid, API de clima |

Multi-tenancy por `hotel_id` en todas las tablas y consultas.

---

## 5. Lo que se va a implementar (plan)

### Objetivos (Protocolo, parte 2 — derivados del problema de la parte 1)

**Objetivo general:** Reducir la brecha tecnológica y de gestión que separa a los hoteles boutique, independientes y cadenas pequeñas de México (20–200 habitaciones) de los hoteles de cadena, sustituyendo su operación fragmentada (herramientas aisladas, atención manual, intuición) por una gestión integrada y basada en datos, mediante una solución diseñada para el segmento y con un costo compatible con su escala de ingresos.

| OE | Objetivo específico | Aspecto del problema |
|---|---|---|
| OE1 | Caracterizar la operación actual (caso Jalisco): herramientas, integración, procesos manuales, decisiones de tarifas y compras | Situación y causas; alcance local |
| OE2 | Eliminar la fragmentación de la información operativa con un estado operativo único en tiempo real | Herramientas dispersas; sin estado compartido |
| OE3 | Reducir la atención manual del canal WhatsApp | WhatsApp 92.6% de penetración, operado a mano |
| OE4 | Sustituir decisiones de tarifa e ingresos por experiencia por decisiones basadas en datos (incluye pronóstico de ingresos y margen de contribución del mes) | Brecha de ocupación de hasta 46 pp |
| OE5 | Aplicar IA viable con datos de un hotel pequeño para anticipar el consumo de insumos | Sin modelos de IA en la operación PyME |
| OE6 | Asegurar un costo de adopción compatible con el segmento | PMS enterprise desde 8,400 USD/año |
| OE7 | Evaluar el efecto de la solución sobre los indicadores del problema | Magnitud del problema |

### Fases de desarrollo (iterativo-incremental)

| Fase | Actividades | Objetivos |
|---|---|---|
| 1. Análisis y modelado | Diagnóstico de la operación; requisitos; BPMN de los 4 macroprocesos; casos de uso por rol | OE1 |
| 2. Arquitectura y base técnica | Arquitectura y BD multi-hotel; auth por rol; Redis + WebSocket | OE2, OE6 |
| 3. BPM y optimizador | Front Office, Housekeeping, Mantenimiento, F&B (KDS); asignación, rutas, prioridad de OT | OE2 |
| 4. CRM, canales y facturación | Perfil 360°, RFM, motor de reglas; link PWA, WhatsApp + asistente, kiosco; channel manager propio, motor de reservas, CFDI 4.0 | OE2, OE3 |
| 5. Inteligencia de insumos | Captura de consumo; dataset de entrenamiento; ABC-XYZ, STL; modelos de pronóstico | OE5 |
| 6. Reabastecimiento y Analytics | Punto de reorden, newsvendor, OR-Tools, merma; KPIs, ocupación proyectada, tarifas por reglas, pronóstico de ingresos y margen de contribución, reportes | OE4, OE5 |
| 7. Evaluación y documentación | Medición de indicadores; validación temporal (WAPE, pinball, sesgo vs. método de referencia); estimación de costos; pruebas funcionales y de usabilidad | OE6, OE7 |

### Resultados esperados (escenario optimista, hotel de referencia de 50 hab., 58.1% de ocupación, ~442 reservas/mes)

| OE | Resultado esperado | Base |
|---|---|---|
| OE2 | 100% de reservas en un solo inventario; 0 sobreventas por desfase; estado visible en < 1 s | Reglas de diseño |
| OE3 | ≥ 80% de conversaciones de WhatsApp sin intervención del personal | HiJiffy: hasta 90%, 80–94% en clientes |
| OE4 | +2% a +4% de ingresos por habitaciones; error de ocupación proyectada ≤ 10% a 30 días | Pekgün et al. (2013); Weatherford y Kimes (2003) |
| OE5 | −21% desperdicio de cocina (optimista −50%); −2% a −8% costo de alimentos; quiebres ≤ 5% de ciclos | Champions 12.3 (2018); Winnow; nivel de servicio 95% |
| OE6 | Costo de operación ≈ 52 USD (≈ 934 MXN) al mes por hotel: infraestructura 6.30 + WhatsApp 11.27 + Claude Haiku 2.43 + CFDI 31.91 | Precios públicos; 18 MXN/USD |
| OE7 | Check-in < 2 min; reporte diario < 30 s | Metas de diseño |

**Fuera de alcance:** Gemelo Digital / simulación de escenarios, modelos predictivos del huésped, sensores IoT, cerraduras electrónicas (fase posterior).

---

## 6. Pendientes

- [ ] Actualizar el subtítulo de `Protocolo_Investigacion_OmniStay.docx` (quitar "CRM predictivo y Gemelo Digital Operativo").
- [x] Channel manager propio y CFDI 4.0 incluidos en el producto (decisión tomada).
- [x] Fuente de datos para entrenar y evaluar los modelos: hotel piloto (decidido).
- [x] Puntos 1–5 del formato del protocolo consolidados en `Protocolo_OmniStay_Puntos_1-5.docx`.
- [ ] Puntos 6–9 del formato: 6. Programa de actividades (calendarizado, avance porcentual trimestral), 7. Infraestructura y apoyo técnico, 8. Conocimientos y habilidades, 9. Marco lógico (fin, propósito, productos esperados).
