# 05 — Restricciones del Sistema

Una **restricción** es una condición impuesta que limita las decisiones de diseño y no puede negociarse libremente. Se clasifican por origen.

## 1. Restricciones técnicas

| ID | Restricción | Origen |
| --- | --- | --- |
| RT-01 | El backend se desarrolla con **.NET 10 (ASP.NET Core Web API)**. | Decisión del proyecto |
| RT-02 | La persistencia relacional usa **PostgreSQL**; cada servicio dispone de su propia instancia o esquema. | Decisión del proyecto |
| RT-03 | Se utiliza **Redis** como caché de disponibilidad (expiración corta) y de catálogo (expiración mayor). | Decisión del proyecto |
| RT-04 | La comunicación entre E-commerce e Inventario es **REST síncrona**, sin bus de eventos. | Simplicidad operativa para 5–10 sucursales |
| RT-05 | La seguridad usa **JWT / OAuth 2.0** con el alcance de sucursal codificado en el token. | Requisito de seguridad |
| RT-06 | El Asistente de Compra no accede directamente a las bases de datos; consulta las API de E-commerce e Inventario. | Seguridad y desacoplamiento |
| RT-07 | El entorno de desarrollo es JetBrains Rider y reside solo en el equipo local, nunca en el servidor. | Decisión del proyecto |
| RT-08 | Las pruebas usan xUnit, Testcontainers, Playwright y k6. | Estrategia de pruebas |

## 2. Restricciones de infraestructura y despliegue

| ID | Restricción | Origen |
| --- | --- | --- |
| RI-01 | El despliegue se realiza sobre **infraestructura propia (VMs Linux)**, sin servicios administrados de pago. | Alcance del proyecto |
| RI-02 | El sistema operativo es **Debian estable en instalación mínima, sin entorno gráfico**, en todas las VMs de servidor. | Recursos limitados de RAM |
| RI-03 | Los contenedores se gestionan con **Docker y Docker Compose**; Kubernetes queda descartado para 2–3 VMs. | Complejidad innecesaria |
| RI-04 | Las imágenes deben ser **multi-arch (x86_64 y ARM64)**. | Despliegue local y Oracle Cloud ARM |
| RI-05 | Entorno local de referencia: 2 vCPU y 4 GB de RAM por VM. | Recursos disponibles |
| RI-06 | Oracle Cloud Free Tier (ARM Ampere A1): 2 OCPU y 12 GB de RAM en total, repartidos en 2 instancias de 1 OCPU y 6 GB. | Límites del plan gratuito, según la propuesta; verificar al momento del despliegue |
| RI-07 | El único acceso externo permitido es HTTP/HTTPS y SSH (solo con llave pública). | Seguridad |
| RI-08 | No existe SLA contratado; la instancia pública se aloja en Oracle Cloud y el entorno local se reserva para desarrollo y pruebas. | Infraestructura propia |

## 3. Restricciones de diseño y arquitectura

| ID | Restricción | Origen |
| --- | --- | --- |
| RD-01 | BabelHub se compone de **dos sistemas independientes pero interoperables** (E-commerce e Inventario). | Visión del proyecto |
| RD-02 | El inventario sigue el patrón **Multi-Source Inventory**: una sucursal = una fuente; *stock vendible = físico − reservas activas*. | Caso de referencia |
| RD-03 | Se descarta una descomposición en más servicios (catálogo, carrito y pagos por separado) porque no aporta beneficios a esta escala. | Justificación de la propuesta |
| RD-04 | La documentación sigue el estándar **IEEE 1016** y el modelo **C4**. | Marco de documentación |
| RD-05 | Los diagramas de detalle (clases, secuencia, mockups) se expresan como especificación funcional por módulo bajo **Spec-Driven Development**; los diagramas C4 se mantienen para comunicar la arquitectura. | Metodología |
| RD-06 | Las claves de API (pagos, LLM) se gestionan como secretos de entorno, nunca en el repositorio. | Seguridad |

## 4. Restricciones de negocio

| ID | Restricción | Origen |
| --- | --- | --- |
| RN-01 | La red inicial comprende de **5 a 10 sucursales**, con capacidad de crecer hasta unas 20 sin cambiar de patrón. | Alcance |
| RN-02 | Cada sucursal solo puede operar sobre su propio inventario y pedidos, salvo autorización expresa para traslados. | Requisito de negocio |
| RN-03 | La pasarela de pago debe cobrar solo comisión por transacción, sin costo fijo mensual. | Restricción económica |
| RN-04 | Debe soportar despacho a domicilio y recojo en sucursal. | Caso de referencia (Crisol) |
| RN-05 | El costeo y licenciamiento comercial detallado queda fuera de alcance. | Alcance |

## 5. Restricciones legales y de cumplimiento

| ID | Restricción | Origen |
| --- | --- | --- |
| RL-01 | Se debe cumplir la normativa peruana de protección de datos personales (**Ley N.° 29733** y su reglamento). | Marco legal |
| RL-02 | Los datos de pago son manejados por la pasarela; BabelHub no almacena datos de tarjeta. | Seguridad y cumplimiento |
| RL-03 | Las operaciones sensibles deben quedar registradas en auditoría. | Control interno |

## 6. Restricciones académicas y de alcance

| ID | Restricción | Origen |
| --- | --- | --- |
| RA-01 | Entregable de arquitectura inicial para el curso Arquitectura de Software IS-488 (Laboratorio 02). | Curso |
| RA-02 | Quedan fuera de alcance: código a nivel de clases, contratos OpenAPI completos y costeo comercial detallado. | Alcance |
| RA-03 | La aplicación móvil (Android/iOS) y el módulo de analítica avanzada se consideran extensiones posteriores; el alcance inicial es web responsiva. | Priorización |
