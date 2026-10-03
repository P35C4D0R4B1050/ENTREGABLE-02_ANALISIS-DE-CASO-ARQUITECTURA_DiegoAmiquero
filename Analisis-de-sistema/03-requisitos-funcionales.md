# 03 — Requisitos Funcionales

Los requisitos se agrupan por sistema y módulo. La columna **HU** vincula cada requisito con las historias de usuario del documento `02-historias-de-usuario.md`.

Prioridad: **Alta** · **Media** · **Baja**.

---

## A. Sistema de E-commerce

### A.1 Catálogo de productos

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-01 | El sistema deberá permitir a la administración central publicar, editar y dar de baja títulos con nombre, autor, editorial, ISBN, categoría, precio e imágenes como datos obligatorios. | Alta | HU-13 |
| RF-02 | El sistema deberá validar que el ISBN de cada título sea único. | Alta | HU-13 |
| RF-03 | El sistema deberá permitir buscar productos por título, autor, editorial, categoría y rango de precio. | Alta | HU-01 |
| RF-04 | El sistema deberá permitir filtrar la búsqueda por disponibilidad. | Media | HU-01 |
| RF-05 | El sistema deberá mostrar al cliente la disponibilidad de un título de forma consolidada (nacional) y por sucursal. | Alta | HU-02 |
| RF-06 | El sistema deberá deshabilitar la compra de un título cuya disponibilidad consolidada sea 0 y mostrar "No disponible actualmente". | Alta | HU-02 |
| RF-07 | El sistema deberá usar un sistema de categorización único y compartido entre todas las sucursales. | Media | HU-13 |
| RF-08 | La interfaz del cliente deberá ser intuitiva y responsiva para dispositivos móviles y de escritorio. | Alta | HU-01 |

### A.2 Carrito y checkout

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-09 | El sistema deberá mantener el carrito del usuario de forma persistente entre sesiones y dispositivos. | Alta | HU-03 |
| RF-10 | El sistema deberá permitir elegir entre despacho a domicilio y recojo en la sucursal de preferencia del cliente. | Alta | HU-04 |
| RF-11 | El sistema deberá consultar al Servicio de Inventario la disponibilidad real (stock vendible, no solo físico) antes de confirmar el pago. | Alta | HU-05 |
| RF-12 | El sistema deberá informar al cliente cuando el pedido requiera un traslado entre sucursales. | Media | HU-04 |

### A.3 Pagos

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-13 | El sistema deberá integrar pagos con Yape y tarjetas a través de un agregador (Niubiz o Culqi). | Alta | HU-05 |
| RF-14 | El sistema deberá garantizar que el pago se procese una única vez por transacción (idempotencia). | Alta | HU-05 |
| RF-15 | El sistema no deberá almacenar datos sensibles de tarjetas; estos serán gestionados por la pasarela. | Alta | HU-05 |

### A.4 Pedidos

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-16 | El sistema deberá mostrar el estado del pedido en cada etapa: confirmado, en preparación, en tránsito entre sucursales (si aplica), despachado o listo para recojo, entregado y devuelto. | Alta | HU-06 |
| RF-17 | El sistema deberá notificar al cliente ante cada cambio de estado de su pedido. | Alta | HU-06 |
| RF-18 | El sistema deberá notificar a la sucursal responsable ante cada cambio de estado del pedido. | Media | HU-12 |
| RF-19 | El sistema deberá permitir al encargado marcar un pedido como "listo para recojo". | Alta | HU-12 |
| RF-20 | El sistema deberá integrarse con un operador logístico externo para generar guías de envío y obtener el seguimiento en despachos a domicilio. | Media | HU-17 |
| RF-21 | El sistema deberá gestionar devoluciones y reembolsos con trazabilidad y actualizar el stock de la sucursal receptora. | Baja | HU-08 |

### A.5 Asistente de Compra (IA)

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-22 | El sistema deberá ofrecer un chat que recomiende títulos y responda consultas frecuentes (disponibilidad y estado de pedido). | Media | HU-07 |
| RF-23 | El asistente deberá fundamentar sus respuestas en datos reales consultados al Catálogo e Inventario mediante sus API, sin acceder directamente a las bases de datos. | Media | HU-07 |
| RF-24 | El asistente deberá indicar explícitamente cuando no cuente con información suficiente. | Media | HU-07 |

---

## B. Sistema de Gestión de Inventario Multi-Sucursal

### B.1 Fuentes y stock

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-25 | El sistema deberá registrar cada sucursal como una fuente (source) independiente con su propia cantidad física por título. | Alta | HU-09 |
| RF-26 | El sistema deberá calcular el stock vendible en tiempo real como *stock físico − reservas activas*. | Alta | HU-09, HU-19 |
| RF-27 | El sistema deberá exponer una API REST consultable por el E-commerce y el Asistente de Compra. | Alta | HU-02, HU-07 |
| RF-28 | El sistema deberá crear reservas cuando el E-commerce las solicite y liberarlas si el pedido se cancela o expira. | Alta | HU-19 |
| RF-29 | El sistema deberá impedir que el stock vendible de una fuente sea negativo (sin sobreventa), incluso ante solicitudes concurrentes. | Alta | HU-19 |

### B.2 Selección de fuente

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-30 | El sistema deberá determinar automáticamente desde qué sucursal despachar un pedido, priorizando cercanía al cliente y disponibilidad de stock. | Alta | HU-19 |
| RF-31 | El sistema deberá identificar en qué sucursales hay stock vendible cuando la más cercana no lo tenga. | Alta | HU-19 |

### B.3 Traslados entre sucursales

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-32 | El sistema deberá generar solicitudes de traslado con origen, destino, ítems y estado. | Alta | HU-11 |
| RF-33 | El sistema deberá permitir aprobar, despachar y recibir traslados, con trazabilidad completa de los cambios de estado. | Alta | HU-11 |
| RF-34 | El sistema deberá notificar a la sucursal de origen y a la de destino al generarse un traslado. | Media | HU-11 |
| RF-35 | El sistema deberá actualizar el stock de origen y destino al despachar y recibir el traslado. | Alta | HU-11 |
| RF-36 | El sistema deberá permitir al personal de logística consultar los traslados en tránsito. | Media | HU-18 |

### B.4 Ajustes y auditoría de inventario

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-37 | El sistema deberá registrar ajustes de inventario con motivo (merma, daño o corrección de conteo) y usuario responsable, por sucursal. | Alta | HU-10 |
| RF-38 | El sistema deberá mantener un historial inmutable de movimientos de stock por sucursal. | Media | HU-10, HU-16 |

---

## C. Seguridad y administración (ambos sistemas)

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-39 | El sistema deberá autenticar a los usuarios mediante un servicio de identidad centralizado que emita tokens JWT. | Alta | HU-14 |
| RF-40 | El sistema deberá asignar roles diferenciados: cliente, encargado de sucursal, administrador central y logística. | Alta | HU-14 |
| RF-41 | El token deberá incluir el rol y el alcance de sucursal; un encargado solo podrá acceder al inventario y pedidos de su sucursal. | Alta | HU-09, HU-14 |
| RF-42 | El sistema deberá registrar en auditoría las operaciones sensibles: cambios de precio, anulaciones de pedido, ajustes de inventario y aprobación de traslados. | Media | HU-16 |
| RF-43 | El sistema deberá cumplir con la normativa peruana de protección de datos personales (Ley N.° 29733). | Alta | — |

---

## D. Analítica y reportería

Este módulo se considera **opcional y separable** del sistema transaccional; puede implementarse en una fase posterior.

| ID | Requisito | Prioridad | HU |
| --- | --- | --- | --- |
| RF-44 | El sistema deberá permitir a cada sucursal consultar sus indicadores de venta, inventario y traslados. | Media | HU-15 |
| RF-45 | El sistema deberá ofrecer tableros consolidados nacionales con comparación entre sucursales y regiones. | Media | HU-15 |
| RF-46 | El sistema deberá permitir la navegación de indicadores por título, categoría, sucursal, región y período. | Baja | HU-15 |
| RF-47 | El sistema deberá proveer una interfaz de autoservicio para construir reportes sin depender del equipo técnico. | Baja | HU-15 |

---

## Resumen por prioridad

| Prioridad | Cantidad |
| --- | :---: |
| Alta | 30 |
| Media | 14 |
| Baja | 3 |
| **Total** | **47** |
