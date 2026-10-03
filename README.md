# BabelHub — Plataforma de Comercio Electrónico y Gestión de Inventario Multi-Sucursal para librerías

## 👨‍💻 Datos del Proyecto

| Información | Detalle |
| --- | --- |
| Estudiante | Diego Rodrigo Amiquero Antonio |
| Proyecto | Plataforma de e-commerce e inventario multi-sucursal para librerías |
| Tipo | Proyecto académico |
| Caso de estudio | Librerías Crisol (referencia funcional) + patrón Multi-Source Inventory (MSI) de Adobe Commerce |
| Entregable | Laboratorio 02 — Análisis del sistema y arquitectura inicial |

## Descripción

BabelHub se define como **dos sistemas independientes pero interoperables** para una librería con presencia en varias regiones del Perú (red inicial de 5 a 10 sucursales):

- **Sistema de E-commerce:** orientado al cliente final (catálogo, carrito, checkout, pagos, pedidos y asistente de compra con IA).
- **Sistema de Gestión de Inventario Multi-Sucursal:** orientado al personal interno, basado en el patrón **MSI**. Cada sucursal es una *fuente* de stock y el stock vendible se calcula como *stock físico − reservas activas*.

Ambos sistemas se comunican por API REST y se despliegan sobre **infraestructura propia** (VMs Debian con Docker), en entorno local y en Oracle Cloud Free Tier (ARM).

## Caso de estudio

Librerías Crisol como referencia funcional (red física + "Recojo en tienda") y el patrón Multi-Source Inventory de Adobe Commerce como referencia técnica.

## 📂 Estructura del Repositorio

```
├── README.md
├── analisis-de-sistema/
│   ├── 01-actores.md
│   ├── 02-historias-de-usuario.md
│   ├── 03-requisitos-funcionales.md
│   ├── 04-atributos-de-calidad.md
│   ├── 05-restricciones.md
│   └── 06-drivers-arquitectonicos.md
└── arquitectura/
    └── arquitectura-inicial.md
```

## 📁 Índice de Documentación

### 1. Análisis del Sistema

| N.° | Documento | Contenido |
| --- | --- | --- |
| 01 | [Actores del sistema](analisis-de-sistema/01-actores.md) | Actores directos (Cliente, Encargado de sucursal, Administrador central, Logística) y sistemas externos (pasarela de pago, LLM, operador logístico, notificaciones). |
| 02 | [Historias de usuario](analisis-de-sistema/02-historias-de-usuario.md) | Necesidades expresadas desde el usuario, con criterios de aceptación. |
| 03 | [Requisitos funcionales](analisis-de-sistema/03-requisitos-funcionales.md) | Capacidades y reglas de negocio por módulo. |
| 04 | [Atributos de calidad](analisis-de-sistema/04-atributos-de-calidad.md) | Escenarios de consistencia, rendimiento, disponibilidad, escalabilidad, seguridad y modificabilidad. |
| 05 | [Restricciones](analisis-de-sistema/05-restricciones.md) | Restricciones técnicas, de infraestructura, de negocio, legales y académicas. |
| 06 | [Drivers arquitectónicos](analisis-de-sistema/06-drivers-arquitectonicos.md) | Requisitos que condicionan la arquitectura y decisiones derivadas. |

### 2. Diseño de Arquitectura

| Documento | Contenido |
| --- | --- |
| [Arquitectura inicial](arquitectura/arquitectura-inicial.md) | Contexto y contenedores (C4 niveles 1 y 2), flujo de selección de fuente, modelo de datos conceptual y despliegue, con diagramas Mermaid. |

## 🛠️ Tecnologías y Herramientas

- **Lenguaje de marcado:** Markdown
- **Diagramado de arquitectura:** Mermaid.js (modelo C4 + IEEE 1016 como marco de documentación)
- **Stack previsto:** .NET 10 (ASP.NET Core Web API), PostgreSQL, Redis, Nginx, Oracle Cloud, Docker / Docker Compose, Debian estable sin entorno gráfico
- **Pruebas previstas:** xUnit, Testcontainers, Playwright, k6
- **Control de versiones:** Git & GitHub
- **Metodología de desarrollo posterior:** Spec-Driven Development (SDD)
