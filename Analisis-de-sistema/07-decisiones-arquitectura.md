
| ID | Decisión arquitectónica | Driver relacionado | Justificación | Resultado |
| :--- | :--- | :--- | :--- | :--- |
| ADR-001 | Monolito modular | DA01-Escalabilidad;DA06 - Mantenibilidad | Organizar las funcionalidades en módulos independientes dentro de una misma aplicación desplegable. | Módulos de Catálogo, Carrito, Pedidos, Pagos y Usuarios. |
| ADR-002 | Clean Architecture | DA06-Mantenibilidad | Separar las reglas del negocio de los detalles tecnológicos. | Dominio, Aplicación, Infraestructura y Presentación. |
| ADR-003 | Estrategia de caché | DA02-Rendimiento | Reducir consultas repetitivas a la fuente de datos. | Caché para información de consulta frecuente. |
| ADR-004 | Integración de pagos mediante interfaces y adaptadores | DA04-Integración con pagos | Desacoplar los casos de uso del proveedor de pagos. | Contrato de pagos y adaptador para la pasarela externa. |
