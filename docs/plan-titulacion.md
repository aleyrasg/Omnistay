# OmniStay — Plan de titulación y alcance

> Documento de planeación. Resume las decisiones de alcance, la arquitectura sugerida por el profesor y las fases de trabajo hasta la entrega de mayo de 2027.

**Estado:** propuesta aprobada con observaciones · en planeación (stack por definir)
**Entrega:** mayo de 2027
**Ramas que abarca el proyecto:** ingeniería de software, ciberseguridad (lo necesario) y machine learning con análisis de datos

---

## 1. Contexto

La propuesta del protocolo (`Protocolo_OmniStay_Puntos_1-5.pdf`) se expuso y quedó **aprobada con observaciones**. En la revisión, el profesor comentó que OmniStay es interesante como modelo de negocio y tiene un aporte técnico válido si se ofrece como **servicio** a los hoteles, y sugirió reforzar la parte de infraestructura.

El objetivo principal del equipo es **titularse cumpliendo objetivos técnicos en un tiempo realista**. Venderlo como servicio es un objetivo posterior.

Con dos personas y unos 6 meses (noviembre de 2026 a mayo de 2027), el alcance completo del protocolo —7 módulos— no es realista. Por eso se separa en:

- **Alcance de titulación:** lo que demuestra ingeniería en las tres ramas del proyecto.
- **Roadmap comercial:** lo necesario para venderlo, que no aporta a la evaluación de la tesis.

---

## 2. Notas del profesor: arquitectura del servicio

Las notas de la revisión describen una **arquitectura de tres capas con seguridad por capas**, montada en la nube.

Notas originales:

- Acceso a la aplicación por medio de una VPN
- Un firewall
- Tener un load balancing
- Clúster de servidores web
- Y atrás servidores de bases de datos
- Calculadora de AWS

Recorrido de una petición:

```
   Huéspedes · Staff · OTAs · WhatsApp          Administradores
                │                                      │
         [ 1. Firewall ]                        [ 2. VPN ]
                │                                      │
       [ 3. Balanceador de carga ]                     │
          ┌─────┼─────┐                                │
       [Web 1][Web 2][Web n]  ← 4. Clúster web ────────┤
          └─────┼─────┘                                │
   [ 5. Servidores de base de datos ] ← red privada ───┘
```

### 2.1 Firewall

Primera barrera de cualquier petición: deja pasar solo lo permitido y bloquea ataques comunes (inyección SQL, exceso de peticiones).

- Solo se expone HTTPS (443) y las rutas públicas: link del huésped, webhooks de WhatsApp y OTAs, login.
- Límite de peticiones por IP.
- Reglas internas entre capas: la base de datos solo acepta conexiones de los servidores web.

### 2.2 VPN

**No puede ser la puerta de entrada de toda la aplicación:** el huésped abre su link desde el celular, y WhatsApp y las OTAs mandan webhooks desde internet. Ninguno se conecta a una VPN. *(Punto a aclarar con el profesor.)*

Usos que sí tienen sentido:

- **Acceso administrativo (recomendado):** solo el equipo entra por VPN a servidores y base de datos; nada de eso queda expuesto a internet.
- **Acceso del personal del hotel (opcional):** las vistas de recepción y gerencia solo se abren desde la red del hotel, con VPN sitio a sitio. Requiere un router compatible en el hotel.

### 2.3 Balanceador de carga

Reparte las peticiones entre varios servidores web y deja de enviar tráfico a uno que falla.

- Debe soportar **WebSockets** (estado de habitaciones en tiempo real).
- Como un usuario puede caer en cualquier servidor, los cambios se comparten entre servidores con un mecanismo de mensajería común (pub/sub).
- Termina el certificado HTTPS.

### 2.4 Clúster de servidores web

Varias copias idénticas de la aplicación, repartidas en dos zonas de la nube para seguir funcionando aunque falle una.

- Servidores **sin estado**: sesiones y estado en tiempo real en un servicio compartido.
- **Autoescalado**: se agregan servidores cuando sube la carga (temporada alta, hoteles nuevos).
- Procesos pesados en servidores aparte (*workers*): pronósticos nocturnos, reentrenamiento de modelos.

### 2.5 Servidores de base de datos

En red privada, sin acceso desde internet.

- Base principal + réplica en espera para recuperación ante fallas.
- Réplica de solo lectura para Analytics, reportes y entrenamiento de modelos.
- Respaldos automáticos y cifrado (datos personales de huéspedes).
- Aislamiento entre hoteles por `hotel_id`, reforzado con *Row-Level Security*.

### 2.6 Calculadora de AWS

Demuestra la viabilidad como negocio y conecta con **OE6** (costo).

- Tres escenarios, por ejemplo con 5, 20 y 100 hoteles.
- Costo mensual de la infraestructura y **costo por hotel** en cada uno, para mostrar cómo baja conforme crece el servicio.

### 2.7 Cómo demostrarlo en la tesis

| Pieza | Prueba |
| --- | --- |
| Balanceador y clúster | Apagar un servidor a propósito y mostrar que el servicio sigue funcionando |
| Autoescalado | Prueba de carga simulando muchos hoteles; medir tiempo de respuesta y servidores agregados |
| Base de datos | Forzar la caída de la base principal y medir el tiempo de recuperación |
| Firewall y VPN | Mostrar que la base de datos y los servidores no son accesibles desde internet |
| Costos | Tabla de los tres escenarios hecha con la calculadora |

---

## 3. Criterios de priorización

1. **El aporte técnico es la Inteligencia de insumos.** Es la contribución declarada en el protocolo y cubre la rama de ML y datos. Lo demás existe para alimentarla o protegerla.
2. **El BPM se construye solo hasta donde genere datos de consumo:** recepción, housekeeping con kit de reposición, F&B con recetas y almacén. No hace falta un PMS completo.
3. **Ciberseguridad "lo necesario":** la arquitectura del profesor más las protecciones básicas de una aplicación con datos personales.

### Riesgo principal: los datos

Los modelos de insumos necesitan meses de historial de consumo que no se tendrán de un hotel real. Se necesita un **generador de datos sintéticos** calibrado con lo que se obtenga en OE1 (entrevistas a hoteles de Jalisco). Si se consigue un hotel piloto que comparta datos reales, mejor. **Este trabajo debe empezar en diciembre.**

---

## 4. Alcance

### 4.1 Dentro del alcance de titulación

| Componente | Qué incluye | Rama |
| --- | --- | --- |
| Infraestructura | Firewall, VPN de administración, balanceador, clúster web, base de datos privada, infraestructura como código | Ciberseguridad |
| Seguridad de la aplicación | Login con roles, aislamiento por hotel, cifrado, bitácora | Ciberseguridad |
| BPM núcleo | Check-in/out, estado de habitaciones en tiempo real, housekeeping con kit de reposición, pedidos F&B con recetas, almacén | Software |
| Inteligencia de insumos | Dataset, ABC-XYZ, STL, modelos de pronóstico, punto de reorden, newsvendor, alertas de merma, SHAP | ML + Datos |
| Analytics | KPIs, ocupación proyectada, costo de insumos | Datos |

**Si sobra tiempo**, en este orden:

1. Asistente de WhatsApp con el número de prueba gratuito de Meta.
2. Segmentación RFM del CRM.
3. Pronóstico de ingresos del mes.

### 4.2 Fuera del alcance de titulación

| Funcionalidad | Por qué se pospone |
| --- | --- |
| Channel manager con OTAs | Booking, Expedia y Airbnb exigen certificación de partner; no depende del equipo y tarda meses |
| Timbrado CFDI real | Requiere contrato con un PAC; aporta poco a la ingeniería evaluada |
| Motor de reservas y pasarela de pago | Tema comercial, no técnico de la tesis |
| Kiosco | Es la misma PWA; no demuestra nada nuevo |
| Revenue: tarifas sugeridas y pronóstico de ingresos | Basta con KPIs y ocupación proyectada |
| Optimizador OR-Tools | Compite por tiempo con el modelo de insumos |
| Mantenimiento completo | Basta con registrar las refacciones usadas |

---

## 5. Fases de trabajo

Dos vías en paralelo: una persona en **infraestructura y seguridad**, otra en **datos y ML**; ambas se juntan en el BPM. Las fechas consideran vacaciones de fin de año.

| Fase | Fechas | Entregables | Rama |
| --- | --- | --- | --- |
| **0. Cierre de planeación** | 12 nov – 12 dic 2026 | Atender observaciones; congelar alcance; elegir stack; entrevistas OE1 a 3–5 hoteles; modelo de datos; diagrama de arquitectura | Todas |
| **1. Base técnica e infraestructura** | 15 dic 2026 – 30 ene 2027 | Repositorio, CI/CD, infraestructura como código; red pública y privada; firewall; balanceador; 2 servidores web; base de datos privada; VPN de administración; login con roles; aislamiento por hotel. En paralelo: primera versión del generador de datos sintéticos | Ciberseguridad + Datos |
| **2. BPM núcleo** | febrero 2027 (4 semanas) | Check-in/out; estado de habitaciones en tiempo real; housekeeping con kit de reposición; pedidos F&B con recetas que descuentan inventario; movimientos de almacén | Software |
| **3. Inteligencia de insumos** | marzo – mediados de abril 2027 (6 semanas) | Dataset final; análisis exploratorio; ABC-XYZ y STL; modelos Poisson, LightGBM cuantílico y Croston contra un método de referencia; validación temporal; WAPE, pinball y sesgo; SHAP; punto de reorden y newsvendor; alertas de merma | ML + Datos |
| **4. Analytics y pruebas de infraestructura** | mediados – fin de abril 2027 (2–3 semanas) | Dashboard del gerente; pruebas de carga; prueba de falla de servidor y base de datos; escaneo de vulnerabilidades (p. ej. OWASP ZAP); tres escenarios en la calculadora de AWS | Datos + Ciberseguridad |
| **5. Evaluación y documento** | mayo 2027 | Comparación de indicadores contra la situación inicial (OE7); redacción de resultados; defensa. Funciona también como colchón | Todas |

---

## 6. Ciberseguridad: lo necesario

- **Red:** firewall o WAF; servidores web detrás del balanceador; base de datos solo en red privada; VPN solo para administración.
- **Aplicación:** HTTPS, contraseñas con hash, JWT con roles, límite de peticiones, revisión contra el OWASP Top 10.
- **Datos:**
  - Cifrado de la base de datos y de los respaldos.
  - Secretos fuera del código.
  - *Row-Level Security* por hotel.
  - Bitácora de accesos.
- **Legal:** aviso de privacidad conforme a la ley mexicana de protección de datos personales.

**Costo de la nube:** usar créditos educativos (AWS Academy o AWS Educate) y apagar la infraestructura cuando no haya demostraciones; con infraestructura como código se levanta de nuevo en minutos.

---

## 7. Roadmap comercial (después de la titulación)

1. Piloto real con un hotel de Jalisco.
2. Channel manager certificado con las OTAs, motor de reservas directas y pasarela de pago.
3. CFDI 4.0 con un PAC.
4. WhatsApp en producción, kiosco y CRM con acciones automáticas.
5. Revenue: tarifas sugeridas y pronóstico de ingresos y margen.
6. Optimizador OR-Tools y mantenimiento preventivo.
7. Operación del servicio: alta de hoteles, cobro de suscripciones, soporte, monitoreo, respaldos entre regiones y endurecimiento de la seguridad.

---

## 8. Lo que ya está hecho

- [x] Protocolo de investigación (puntos 1 a 5) redactado.
- [x] Propuesta expuesta y aprobada con observaciones.
- [x] Revisión con el profesor sobre el aporte técnico (notas de la sección 2).
- [x] Carpeta del repositorio creada.

## 9. Próximos pasos

- [ ] Validar con el profesor el recorte de alcance (sección 4) y el uso de la VPN (sección 2.2).
- [ ] Ajustar el protocolo: OE3 y parte de OE4 pasan a trabajo futuro; agregar un objetivo de infraestructura segura y escalable.
- [ ] Definir el stack tecnológico.
- [ ] Repartir las dos vías de trabajo entre los integrantes.
- [ ] Preparar y agendar las entrevistas OE1 con hoteles de Jalisco.
- [ ] Buscar un hotel piloto que comparta datos.
- [ ] Solicitar créditos educativos de AWS.
- [ ] Actualizar el README: todavía menciona "CRM predictivo" y "gemelo digital", que ya no forman parte del alcance.
