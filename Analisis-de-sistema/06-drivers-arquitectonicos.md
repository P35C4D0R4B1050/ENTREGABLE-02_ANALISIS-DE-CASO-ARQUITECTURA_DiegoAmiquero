# 06 — Drivers Arquitectónicos

Los **drivers arquitectónicos** son los requisitos funcionales, atributos de calidad y restricciones que más condicionan el diseño. Aquí se seleccionan los de mayor impacto y se muestra qué decisión de arquitectura motiva cada uno.

## 1. Objetivo de negocio

Proveer a una librería con presencia en varias regiones del Perú dos sistemas interoperables: un **e-commerce donde el stock mostrado sea siempre real** y un **inventario multi-sucursal** que coordine despachos y traslados, desplegados sobre infraestructura propia.

## 2. Drivers funcionales (los de mayor impacto)

| ID | Driver | Requisitos | Impacto en la arquitectura |
| --- | --- | --- | --- |
| DF-01 | Inventario multi-fuente con stock vendible en tiempo real | RF-25, RF-26, RF-28, RF-29 | Servicio de Inventario propio, con BD dedicada y control de concurrencia en las reservas. |
| DF-02 | Selección automática de la fuente de despacho | RF-30, RF-31 | Módulo de selección de fuente aislado dentro del Servicio de Inventario. |
| DF-03 | Traslados entre sucursales con trazabilidad | RF-32 a RF-36 | Máquina de estados de traslados con historial de movimientos. |
| DF-04 | Validación de disponibilidad antes del pago y pago idempotente | RF-11, RF-14 | Integración REST síncrona E-commerce → Inventario; clave de idempotencia en pagos. |
| DF-05 | Acceso restringido por sucursal | RF-39 a RF-41 | Servicio de Identidad que emite JWT con rol y alcance de sucursal; validación en cada servicio. |
| DF-06 | Asistente de compra fundamentado en datos reales | RF-22 a RF-24 | Servicio de Asistente que consume las API de Catálogo e Inventario y se conecta a un LLM externo. |

## 3. Drivers de calidad (priorizados)

| Prioridad | Atributo | Escenario | Táctica arquitectónica |
| :---: | --- | --- | --- |
| 1 | Consistencia de inventario | QA-01 | Reservas transaccionales sobre la BD de Inventario; restricción de stock no negativo; transacciones con bloqueo o control de concurrencia. |
| 2 | Seguridad | QA-02 | JWT con alcance de sucursal, TLS en el proxy, secretos de entorno, firewall y SSH con llave pública. |
| 3 | Rendimiento | QA-03 | Caché Redis para disponibilidad (expiración corta) y catálogo (expiración mayor). |
| 4 | Disponibilidad | QA-04 | Reinicio automático de contenedores, instancia pública en Oracle Cloud, backups de la BD de Inventario. |
| 5 | Escalabilidad | QA-05 | Cada sucursal es una fuente; contenedores independientes con recursos asignables por servicio. |
| 6 | Modificabilidad | QA-06 | Límites claros entre servicios; contratos REST estables; pagos tras una interfaz. |
| 7 | Portabilidad | QA-08 | Imágenes Docker multi-arch y Docker Compose por entorno. |

## 4. Restricciones que condicionan el diseño

| Restricción | Efecto |
| --- | --- |
| Infraestructura propia con pocos GB de RAM (RI-01, RI-05, RI-06) | Se evitan servicios pesados y orquestadores; Debian mínimo sin GUI; pocos contenedores. |
| Sin bus de eventos (RT-04) | Comunicación REST síncrona entre servicios. |
| Red de 5–10 sucursales (RN-01) | No se justifican más servicios; se mantiene una arquitectura de cinco servicios/componentes principales. |
| Docker multi-arch (RI-04) | Se eligen imágenes base con soporte ARM64 (PostgreSQL, .NET, Redis, Nginx). |
| Spec-Driven Development (RD-05) | Los contratos y criterios de aceptación deben ser precisos y verificables. |

## 5. Decisiones arquitectónicas derivadas

| ID | Decisión | Driver | Alternativa descartada | Razón |
| --- | --- | --- | --- | --- |
| AD-01 | Separar **E-commerce** e **Inventario** como servicios independientes | DF-01, QA-01, QA-05 | Un monolito con inventario embebido | El inventario tiene lógica compleja y mayor volumen de escritura; permite escalarlo y probarlo por separado. |
| AD-02 | **No** separar catálogo, carrito y pagos en servicios distintos | RN-01, RI-05 | Microservicios finos | Añadiría coordinación y puntos de falla sin beneficio a esta escala. |
| AD-03 | **Dos bases PostgreSQL** (Ventas e Inventario) | QA-01, QA-04 | Una sola BD compartida | Aísla la carga de escritura de reservas y traslados y permite políticas de backup distintas. |
| AD-04 | **REST síncrono** entre servicios | RT-04 | Bus de eventos | Menor complejidad operativa para el volumen previsto. |
| AD-05 | **Servicio de Identidad** centralizado con JWT | DF-05, QA-02 | Autenticación dentro de cada servicio | Evita duplicar la lógica de seguridad y permite el alcance por sucursal. |
| AD-06 | **Redis** como caché | QA-03 | Consultar siempre a la BD | Reduce la carga en picos de tráfico. |
| AD-07 | **Asistente de Compra** como servicio separado que consume API | DF-06, RT-06 | Acceso directo a las BD | Mantiene el aislamiento y limita el alcance del LLM a datos autorizados. |
| AD-08 | **Docker Compose** sobre VMs Debian mínimas | RI-02, RI-03 | Kubernetes | No se justifica para 2–3 VMs. |
| AD-09 | **Nginx** como proxy inverso único | RI-07, QA-02 | Un API Gateway dedicado | Cubre enrutamiento y TLS con bajo consumo de recursos. |
| AD-10 | **Reserva temporal con expiración** al iniciar el pago, confirmada al aprobarse el cobro (*propuesta a validar*) | QA-01, DF-04 | Reservar solo después de pagar | Evita la ventana de sobreventa entre la validación y la confirmación del pago. |

## 6. Riesgos y trade-offs

| Riesgo | Mitigación |
| --- | --- |
| La consulta síncrona a Inventario puede volverse un cuello de botella o un punto único de falla. | Caché Redis, reinicio automático, pruebas de carga con k6 y mensajes claros al usuario si no hay validación. |
| Datos de disponibilidad en caché pueden quedar desactualizados. | Expiración corta y validación definitiva contra la BD al reservar. |
| Los recursos limitados (RAM/CPU) afectan el rendimiento en picos. | Debian mínimo, imágenes livianas y asignación de recursos por contenedor. |
| La dependencia de un LLM externo introduce latencia, costo y riesgo de respuestas incorrectas. | El asistente se fundamenta solo en datos de las API, indica cuándo no sabe y su falla no bloquea la compra. |
| Sin SLA ni alta disponibilidad real en infraestructura propia. | Instancia pública en Oracle Cloud, backups periódicos y plan de recuperación documentado. |
