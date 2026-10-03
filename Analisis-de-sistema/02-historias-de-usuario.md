# 02 — Historias de Usuario

Formato: **Como** `<actor>` **quiero** `<necesidad>` **para** `<beneficio>`, con criterios de aceptación en formato *Dado / Cuando / Entonces*.

Prioridad: **Alta** (núcleo del sistema) · **Media** · **Baja** (puede diferirse).

---

## 1. Cliente

### HU-01 — Buscar libros
**Como** cliente **quiero** buscar títulos por nombre, autor, editorial, categoría y rango de precio **para** encontrar rápidamente el libro que necesito.
**Prioridad:** Alta
- **Dado** que estoy en el catálogo, **cuando** aplico uno o varios filtros, **entonces** veo solo los títulos que cumplen todos los criterios.
- **Dado** que la búsqueda no tiene coincidencias, **cuando** se muestran los resultados, **entonces** veo un estado vacío con sugerencias.

### HU-02 — Ver la ficha de un libro con su disponibilidad
**Como** cliente **quiero** ver la disponibilidad de un título de forma consolidada y por sucursal **para** saber dónde puedo recogerlo o si me lo pueden enviar.
**Prioridad:** Alta
- **Dado** que abro la ficha de un título, **cuando** hay stock vendible en al menos una sucursal, **entonces** veo la disponibilidad nacional y el detalle por sucursal.
- **Dado** que la disponibilidad consolidada es 0, **cuando** veo la ficha, **entonces** el botón de compra se deshabilita y muestra "No disponible actualmente".

### HU-03 — Mantener mi carrito
**Como** cliente **quiero** que mi carrito se conserve entre sesiones y dispositivos **para** continuar mi compra sin volver a empezar.
**Prioridad:** Alta
- **Dado** que agregué títulos a mi carrito, **cuando** inicio sesión desde otro dispositivo, **entonces** veo los mismos ítems.

### HU-04 — Elegir despacho a domicilio o recojo en sucursal
**Como** cliente **quiero** elegir entre despacho a domicilio o recojo en la sucursal de mi preferencia **para** recibir mi pedido de la forma más conveniente.
**Prioridad:** Alta
- **Dado** que estoy en el checkout, **cuando** elijo recojo en tienda, **entonces** solo veo las sucursales que pueden atender mi pedido.
- **Dado** que la sucursal elegida no tiene stock, **cuando** otra sucursal sí lo tiene, **entonces** el sistema me informa que el pedido requerirá un traslado y el plazo estimado.

### HU-05 — Pagar con Yape o tarjeta
**Como** cliente **quiero** pagar con Yape o tarjeta **para** completar mi compra de forma segura.
**Prioridad:** Alta
- **Dado** que confirmo el pago, **cuando** el stock vendible ya no alcanza, **entonces** el sistema no cobra e informa el problema.
- **Dado** que ocurre un reintento por falla de red, **cuando** se procesa la transacción, **entonces** el cobro se realiza una sola vez.

### HU-06 — Seguir mi pedido
**Como** cliente **quiero** ver el estado de mi pedido en cada etapa y recibir notificaciones **para** saber cuándo estará listo.
**Prioridad:** Alta
- **Dado** que tengo un pedido, **cuando** cambia de estado (confirmado, en preparación, en tránsito entre sucursales, despachado o listo para recojo, entregado), **entonces** lo veo reflejado y recibo una notificación.

### HU-07 — Consultar al Asistente de Compra
**Como** cliente **quiero** conversar con un asistente que me recomiende títulos y responda consultas frecuentes **para** decidir mi compra con menos esfuerzo.
**Prioridad:** Media
- **Dado** que pregunto por la disponibilidad de un título, **cuando** el asistente responde, **entonces** la respuesta se basa en datos reales del Catálogo e Inventario.
- **Dado** que el asistente no puede determinar la respuesta, **cuando** responde, **entonces** lo indica explícitamente y no inventa datos.

### HU-08 — Solicitar una devolución
**Como** cliente **quiero** solicitar la devolución de un pedido **para** recuperar mi dinero si el producto no es adecuado.
**Prioridad:** Baja
- **Dado** que mi pedido fue entregado, **cuando** solicito la devolución, **entonces** puedo seguir su estado hasta el reembolso.

---

## 2. Encargado de sucursal

### HU-09 — Consultar el inventario de mi sucursal
**Como** encargado de sucursal **quiero** ver el stock físico, las reservas y el stock vendible de mi tienda **para** saber qué puedo vender y qué reponer.
**Prioridad:** Alta
- **Dado** que inicio sesión, **cuando** abro el panel de inventario, **entonces** solo veo los datos de mi sucursal.
- **Dado** que intento acceder a otra sucursal, **cuando** envío la solicitud, **entonces** el sistema la rechaza.

### HU-10 — Registrar ajustes de inventario
**Como** encargado de sucursal **quiero** registrar mermas, daños y correcciones de conteo **para** mantener el stock exacto y auditable.
**Prioridad:** Alta
- **Dado** que registro un ajuste, **cuando** lo confirmo, **entonces** debo indicar el motivo y el sistema guarda mi usuario, la fecha y la sucursal.

### HU-11 — Gestionar traslados entre sucursales
**Como** encargado de sucursal **quiero** una bandeja de traslados entrantes y salientes **para** aprobar, despachar y recibir mercadería con trazabilidad.
**Prioridad:** Alta
- **Dado** que se genera un traslado hacia o desde mi sucursal, **cuando** ingreso a la bandeja, **entonces** lo veo con su estado y puedo aprobarlo, marcarlo como despachado o recibido, según corresponda.
- **Dado** que recibo un traslado, **cuando** confirmo la recepción, **entonces** el stock de mi sucursal aumenta automáticamente.

### HU-12 — Preparar pedidos de recojo
**Como** encargado de sucursal **quiero** ver los pedidos asignados a mi tienda y marcarlos como "listo para recojo" **para** que el cliente sea notificado.
**Prioridad:** Alta
- **Dado** que marco un pedido como listo para recojo, **cuando** guardo el cambio, **entonces** el cliente recibe una notificación.

---

## 3. Administrador central

### HU-13 — Gestionar el catálogo
**Como** administrador central **quiero** dar de alta, editar y dar de baja títulos con nombre, autor, editorial, ISBN, categoría, precio e imágenes **para** mantener un catálogo único para todas las sucursales.
**Prioridad:** Alta
- **Dado** que creo un título, **cuando** falta un dato obligatorio, **entonces** el sistema no me permite publicarlo.
- **Dado** que modifico el precio de un título, **cuando** guardo el cambio, **entonces** queda registrado en auditoría.

### HU-14 — Gestionar usuarios y roles
**Como** administrador central **quiero** crear usuarios y asignarles rol y sucursal **para** controlar quién accede a qué información.
**Prioridad:** Alta
- **Dado** que creo un encargado de sucursal, **cuando** le asigno una sucursal, **entonces** su token solo incluye el alcance de esa sucursal.

### HU-15 — Ver indicadores consolidados
**Como** administrador central **quiero** ver indicadores de venta, inventario y traslados por sucursal y región **para** comparar desempeño y detectar patrones de demanda.
**Prioridad:** Media
- **Dado** que abro el panel de indicadores, **cuando** selecciono un período, **entonces** veo los datos consolidados y puedo desglosarlos por título, categoría, sucursal y región.

### HU-16 — Revisar la auditoría
**Como** administrador central **quiero** consultar quién hizo qué y cuándo en las operaciones sensibles **para** detectar errores o usos indebidos.
**Prioridad:** Media
- **Dado** que se ejecutó un cambio de precio, una anulación, un ajuste o una aprobación de traslado, **cuando** consulto la auditoría, **entonces** veo el registro con usuario, fecha y detalle.

---

## 4. Personal de logística

### HU-17 — Generar guías de envío
**Como** personal de logística **quiero** generar la guía de envío con el operador logístico **para** despachar los pedidos a domicilio.
**Prioridad:** Media
- **Dado** que un pedido a domicilio está preparado, **cuando** solicito la guía, **entonces** el pedido guarda el código de seguimiento y pasa a "despachado".

### HU-18 — Dar seguimiento a los traslados
**Como** personal de logística **quiero** ver los traslados en tránsito **para** coordinar su entrega y detectar retrasos.
**Prioridad:** Media
- **Dado** que existen traslados en tránsito, **cuando** abro el seguimiento, **entonces** veo origen, destino, ítems y estado actual.

---

## 5. Historia del sistema

### HU-19 — Selección automática de la fuente de despacho
**Como** sistema de inventario **quiero** determinar desde qué sucursal despachar cada pedido **para** evitar sobreventas y reducir los tiempos de entrega.
**Prioridad:** Alta
- **Dado** que llega un pedido, **cuando** la sucursal más cercana al cliente tiene stock vendible, **entonces** se reserva en esa sucursal.
- **Dado** que la más cercana no tiene stock, **cuando** otra sucursal sí lo tiene, **entonces** se reserva en esa fuente y se genera una solicitud de traslado o un despacho directo.
- **Dado** que ninguna sucursal tiene stock vendible, **cuando** se evalúa el pedido, **entonces** se rechaza la reserva.

---

## Trazabilidad con los actores

| Actor | Historias |
| --- | --- |
| Cliente | HU-01 a HU-08 |
| Encargado de sucursal | HU-09 a HU-12 |
| Administrador central | HU-13 a HU-16 |
| Personal de logística | HU-17, HU-18 |
| Sistema de inventario | HU-19 |
