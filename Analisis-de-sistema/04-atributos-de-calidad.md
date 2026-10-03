# 04 — Atributos de Calidad

Los atributos se describen mediante **escenarios de calidad** (fuente, estímulo, entorno, artefacto, respuesta y medida). Las métricas son **metas propuestas** para una red de 5 a 10 sucursales y deberán validarse con las pruebas de carga (k6) en la etapa de implementación.

## Resumen y prioridad

| ID | Atributo | Prioridad | Justificación |
| --- | --- | :---: | --- |
| QA-01 | Consistencia de inventario (sin sobreventa) | **Crítica** | Es el problema central del proyecto: mostrar siempre stock real. |
| QA-02 | Seguridad y control de acceso | **Alta** | Cada encargado solo debe ver su sucursal; se procesan pagos y datos personales. |
| QA-03 | Rendimiento | **Alta** | La consulta de disponibilidad está en el camino crítico del checkout. |
| QA-04 | Disponibilidad | **Alta** | Las compras pueden originarse a cualquier hora. |
| QA-05 | Escalabilidad | Media | Se prevén 5–10 sucursales con crecimiento hacia 20. |
| QA-06 | Modificabilidad / Extensibilidad | Media | Se deben poder agregar sucursales y canales sin rediseñar. |
| QA-07 | Trazabilidad y auditabilidad | Media | Ajustes, traslados y operaciones sensibles deben ser verificables. |
| QA-08 | Portabilidad y despliegue | Media | Debe funcionar en VMs locales x86_64 y en Oracle Cloud ARM64. |
| QA-09 | Testabilidad | Media | Habilita el desarrollo guiado por especificaciones. |
| QA-10 | Usabilidad | Media | Interfaz responsiva y estados claros en cada pantalla. |

---

## QA-01 — Consistencia de inventario

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Dos clientes y/o el Asistente de Compra |
| **Estímulo** | Solicitan reservar la última unidad de un título en la misma sucursal casi al mismo tiempo |
| **Entorno** | Operación normal y picos de tráfico |
| **Artefacto** | Servicio de Inventario Multi-Fuente y su base de datos |
| **Respuesta** | Una reserva se concede y la otra se rechaza o se asigna a otra fuente |
| **Medida** | 0 casos de stock vendible negativo en las pruebas de concurrencia |

## QA-02 — Seguridad y control de acceso

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Encargado de una sucursal |
| **Estímulo** | Intenta consultar o modificar el inventario de otra sucursal |
| **Entorno** | Operación normal |
| **Artefacto** | Servicios de Inventario e Identidad (JWT con alcance de sucursal) |
| **Respuesta** | La solicitud se rechaza y se registra el intento |
| **Medida** | 100 % de los accesos fuera de alcance rechazados (HTTP 403) |

*Escenario complementario:* ningún dato de tarjeta se almacena en el sistema; los secretos (pagos, LLM) no residen en el repositorio y el tráfico externo viaja cifrado con TLS.

## QA-03 — Rendimiento

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Clientes en línea |
| **Estímulo** | Consultan la disponibilidad de títulos durante una campaña o feria del libro |
| **Entorno** | Pico de tráfico (hasta 50 usuarios concurrentes en la meta inicial) |
| **Artefacto** | Servicio de Inventario y caché Redis |
| **Respuesta** | La disponibilidad se devuelve desde caché con expiración corta o desde la BD |
| **Medida** | p95 < 500 ms en la consulta de disponibilidad; p95 < 3 s en la confirmación de checkout (sin contar la pasarela) |

## QA-04 — Disponibilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Fallo de un contenedor |
| **Estímulo** | El Servicio de Inventario deja de responder |
| **Entorno** | Operación normal |
| **Artefacto** | Docker Compose, Nginx y E-commerce |
| **Respuesta** | El contenedor se reinicia automáticamente; el E-commerce no confirma compras sin validar stock y muestra un mensaje claro; el catálogo sigue consultable desde caché |
| **Medida** | Recuperación automática en menos de 2 minutos; disponibilidad objetivo ≥ 99 % mensual (sin SLA contratado) |

## QA-05 — Escalabilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | La administración |
| **Estímulo** | Incorpora nuevas sucursales hasta llegar a 20 |
| **Entorno** | Operación normal |
| **Artefacto** | Servicio de Inventario |
| **Respuesta** | Cada sucursal se registra como una nueva fuente, sin cambios de código ni de esquema |
| **Medida** | 0 líneas de código modificadas; rendimiento dentro de la meta de QA-03 |

*Escenario complementario:* ante carga alta se puede asignar más CPU/memoria al contenedor de Inventario sin escalar el resto.

## QA-06 — Modificabilidad / Extensibilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Equipo de desarrollo |
| **Estímulo** | Cambia la estrategia de selección de fuente (por ejemplo, de cercanía a menor costo de traslado) |
| **Entorno** | Desarrollo |
| **Artefacto** | Módulo de selección de fuente en el Servicio de Inventario |
| **Respuesta** | El cambio queda aislado en ese módulo, sin afectar al E-commerce |
| **Medida** | Cambio localizado en un solo servicio; contrato REST sin modificaciones |

*Escenario complementario:* cambiar de pasarela de pago (Niubiz ↔ Culqi) solo afecta a una interfaz de pagos dentro del E-commerce.

## QA-07 — Trazabilidad y auditabilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Administrador central |
| **Estímulo** | Consulta quién realizó un ajuste de inventario o aprobó un traslado |
| **Entorno** | Operación normal |
| **Artefacto** | Registro de auditoría y de movimientos |
| **Respuesta** | Muestra usuario, sucursal, fecha, motivo y valores antes y después |
| **Medida** | 100 % de las operaciones sensibles con registro; el historial no puede modificarse |

## QA-08 — Portabilidad y despliegue

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Equipo de despliegue |
| **Estímulo** | Despliega el sistema en una VM x86_64 local y en una VM ARM64 de Oracle Cloud |
| **Entorno** | Local y nube |
| **Artefacto** | Imágenes Docker multi-arch y manifiestos de Docker Compose |
| **Respuesta** | El sistema se levanta con la misma configuración, variando solo variables de entorno |
| **Medida** | Despliegue con un único comando por VM; imágenes con variantes x86_64 y ARM64 |

## QA-09 — Testabilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Pipeline de pruebas |
| **Estímulo** | Se ejecutan las pruebas ante un cambio en el código |
| **Entorno** | Desarrollo |
| **Artefacto** | Servicios .NET con pruebas xUnit, Testcontainers, Playwright y k6 |
| **Respuesta** | Las pruebas validan la lógica (unitarias), la comunicación E-commerce ↔ Inventario (integración) y los flujos completos (E2E) |
| **Medida** | Cada criterio de aceptación de las historias de usuario tiene al menos una prueba asociada |

## QA-10 — Usabilidad

| Elemento | Descripción |
| --- | --- |
| **Fuente** | Cliente en dispositivo móvil |
| **Estímulo** | Busca un libro y completa una compra con recojo en sucursal |
| **Entorno** | Pantalla pequeña y conexión móvil |
| **Artefacto** | Interfaz web del cliente |
| **Respuesta** | Cada pantalla define sus estados (vacío, cargando, error, con datos) y mensajes claros |
| **Medida** | Compra completada en no más de 5 pasos desde el catálogo |
