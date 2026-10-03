# Arquitectura Inicial del Sistema — BabelHub

Documento de arquitectura inicial basado en el **modelo C4** (niveles 1 y 2) y en el marco **IEEE 1016**. Los diagramas se elaboran con **Mermaid**.

## 1. Resumen

BabelHub se compone de dos sistemas independientes pero interoperables:

- **Sistema de E-commerce** (cliente final): catálogo, carrito, checkout, pagos, pedidos y asistente de compra.
- **Sistema de Inventario Multi-Sucursal** (personal interno): fuentes por sucursal, stock vendible, selección de fuente y traslados, según el patrón **Multi-Source Inventory**.

**Estilo arquitectónico:** servicios desacoplados de granularidad gruesa (no microservicios finos), comunicados por REST síncrono, con una base de datos por servicio, caché Redis y un proxy inverso como único punto de entrada. Se descarta una descomposición mayor porque, para 5–10 sucursales, solo añadiría complejidad de despliegue sobre infraestructura propia y limitada.

## 2. Contexto del sistema (C4 — Nivel 1)

```mermaid
flowchart LR
    CLI["Cliente"]
    ENC["Encargado de sucursal"]
    ADM["Administrador central"]
    LOG["Personal de logística"]

    BH["BabelHub<br/>E-commerce + Inventario Multi-Sucursal"]

    PAG["Pasarela de pago<br/>Niubiz o Culqi - Yape y tarjetas"]
    LLM["Proveedor de LLM<br/>ej. API de Claude"]
    COU["Operador logístico externo"]
    NOT["Servicio de notificaciones<br/>correo / SMS"]

    CLI -->|"Busca y compra"| BH
    ENC -->|"Gestiona stock y traslados"| BH
    ADM -->|"Administra catálogo, usuarios e indicadores"| BH
    LOG -->|"Da seguimiento a traslados y envíos"| BH

    BH -->|"Cobros"| PAG
    BH -->|"Consultas del asistente"| LLM
    BH -->|"Guías y tracking"| COU
    BH -->|"Avisos de estado"| NOT
```

## 3. Contenedores (C4 — Nivel 2)

```mermaid
flowchart TB
    CLI["Cliente"]
    STAFF["Encargado de sucursal<br/>Administrador central<br/>Logística"]

    subgraph EXT["Sistemas externos"]
        PAG["Pasarela de pago<br/>Niubiz o Culqi"]
        LLM["API de LLM externo"]
        COU["Operador logístico"]
        NOT["Notificaciones"]
    end

    subgraph VMA["VM de Aplicaciones"]
        NGX["Nginx<br/>Proxy inverso + TLS"]
        ECO["Servicio de E-commerce<br/>Catálogo, carrito, checkout, pagos, pedidos"]
        INV["Servicio de Inventario Multi-Fuente<br/>Fuentes, stock vendible, selección, traslados"]
        IDE["Servicio de Identidad y Seguridad<br/>Emisión y validación de JWT"]
        ASI["Servicio de Asistente de Compra<br/>Integración con LLM"]
        RED[("Redis<br/>Caché")]
    end

    subgraph VMD["VM de Datos"]
        DBV[("PostgreSQL Ventas")]
        DBI[("PostgreSQL Inventario")]
    end

    CLI -->|"HTTPS"| NGX
    STAFF -->|"HTTPS"| NGX
    NGX --> ECO
    NGX --> INV
    NGX --> IDE
    NGX --> ASI

    ECO -->|"REST: disponibilidad y reservas"| INV
    ASI -->|"REST: catálogo y pedidos"| ECO
    ASI -->|"REST: disponibilidad"| INV
    ECO -.->|"Valida JWT"| IDE
    INV -.->|"Valida JWT"| IDE
    ASI -.->|"Valida JWT"| IDE

    ECO --> RED
    INV --> RED
    ECO --> DBV
    INV --> DBI

    ECO -->|"Cobros"| PAG
    ECO -->|"Guías de envío"| COU
    ECO --> NOT
    INV --> NOT
    ASI -->|"Prompts"| LLM
```

### 3.1 Descripción de contenedores

| Contenedor | Tecnología | Responsabilidad |
| --- | --- | --- |
| **Servicio de E-commerce** | .NET 10, ASP.NET Core | Catálogo, carrito persistente, checkout, pagos, pedidos y notificaciones al cliente. Es el único contenedor con el que interactúa el cliente final. |
| **Servicio de Inventario Multi-Fuente** | .NET 10, ASP.NET Core | Registro de fuentes (sucursales), cálculo de stock vendible, reservas, selección de fuente, traslados y ajustes con auditoría. Expone la API consumida por E-commerce y el Asistente. |
| **Servicio de Identidad y Seguridad** | .NET 10, ASP.NET Core | Autenticación, roles y emisión/validación de JWT con el alcance de sucursal. |
| **Servicio de Asistente de Compra** | .NET 10, ASP.NET Core | Integración con un LLM externo; consulta las API de Catálogo e Inventario para fundamentar recomendaciones y respuestas. |
| **Caché** | Redis | Disponibilidad consolidada (expiración corta) y catálogo (expiración mayor). |
| **Proxy inverso** | Nginx | Punto de entrada único, enrutamiento y terminación TLS (Let's Encrypt). |
| **BD Ventas** | PostgreSQL | Datos del servicio de E-commerce. |
| **BD Inventario** | PostgreSQL | Datos del servicio de Inventario, separada por su alto volumen de escritura (reservas y traslados). |

> **Supuestos a confirmar:** (1) el Servicio de Identidad guarda sus datos en un esquema propio dentro de la instancia PostgreSQL de Ventas; (2) el Asistente de Compra y Redis se ubican en la VM de Aplicaciones, ya que la propuesta original no los asigna explícitamente a una VM.

### 3.2 Reglas de comunicación

- **E-commerce ↔ Inventario:** REST síncrono. Sin bus de eventos.
- **Asistente → E-commerce e Inventario:** solo por API; nunca accede a las bases de datos.
- **Todos los servicios:** validan el JWT emitido por el Servicio de Identidad.
- **Secretos** (claves de pago y de LLM): variables de entorno, nunca en el repositorio.

## 4. Flujo principal: compra con selección de fuente

```mermaid
sequenceDiagram
    autonumber
    actor C as Cliente
    participant N as Nginx
    participant E as E-commerce
    participant I as Inventario
    participant P as Pasarela de pago
    participant S as Notificaciones

    C->>N: Confirma checkout (despacho o recojo)
    N->>E: Solicitud autenticada con JWT
    E->>I: Consulta disponibilidad real (stock vendible)
    I-->>E: Disponibilidad por sucursal

    alt Sin stock vendible en ninguna sucursal
        E-->>C: No disponible actualmente
    else Hay stock en al menos una sucursal
        E->>I: Reserva temporal con expiración
        I->>I: Selecciona fuente (cercanía y disponibilidad)
        I-->>E: Fuente asignada
        E->>P: Cobro con clave de idempotencia
        P-->>E: Pago aprobado
        E->>I: Confirma la reserva

        opt La sucursal más cercana no tiene stock
            I->>I: Genera solicitud de traslado
            I->>S: Notifica sucursal origen y destino
        end

        E->>S: Notifica al cliente (pedido confirmado)
        E-->>C: Pedido confirmado
    end
```

## 5. Estados del pedido y del traslado

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Confirmado
    Confirmado --> EnPreparacion
    EnPreparacion --> EnTransito: Requiere traslado
    EnTransito --> EnPreparacion: Traslado recibido
    EnPreparacion --> ListoParaRecojo: Recojo en sucursal
    EnPreparacion --> Despachado: Despacho a domicilio
    ListoParaRecojo --> Entregado
    Despachado --> Entregado
    Entregado --> Devuelto
    Entregado --> [*]
    Devuelto --> [*]
```

```mermaid
stateDiagram-v2
    direction LR
    [*] --> Solicitado
    Solicitado --> Aprobado
    Solicitado --> Rechazado
    Aprobado --> EnTransito: Origen despacha
    EnTransito --> Recibido: Destino confirma
    Recibido --> [*]
    Rechazado --> [*]
```

## 6. Modelo de datos conceptual

### 6.1 BD de Inventario

```mermaid
erDiagram
    SOURCE ||--o{ SOURCE_ITEM : "tiene"
    SOURCE_ITEM ||--o{ RESERVATION : "se reserva en"
    SOURCE ||--o{ TRANSFER : "origen"
    SOURCE ||--o{ TRANSFER : "destino"
    TRANSFER ||--|{ TRANSFER_ITEM : "contiene"
    SOURCE_ITEM ||--o{ STOCK_ADJUSTMENT : "ajustado por"
    SOURCE_ITEM ||--o{ STOCK_MOVEMENT : "registra"

    SOURCE {
        uuid id PK
        string name
        string region
        string status
    }
    SOURCE_ITEM {
        uuid id PK
        uuid source_id FK
        string isbn
        int physical_qty
    }
    RESERVATION {
        uuid id PK
        uuid source_item_id FK
        string order_ref
        int qty
        string status
        datetime expires_at
    }
    TRANSFER {
        uuid id PK
        uuid origin_source_id FK
        uuid dest_source_id FK
        string status
        datetime created_at
    }
    TRANSFER_ITEM {
        uuid id PK
        uuid transfer_id FK
        string isbn
        int qty
    }
    STOCK_ADJUSTMENT {
        uuid id PK
        uuid source_item_id FK
        string reason
        int delta
        string user_id
        datetime created_at
    }
    STOCK_MOVEMENT {
        uuid id PK
        uuid source_item_id FK
        string type
        int qty
        datetime created_at
    }
```

El **stock vendible** se calcula como `physical_qty − SUM(qty de reservas activas)` por fuente y título.

### 6.2 BD de Ventas

```mermaid
erDiagram
    PRODUCT ||--o{ CART_ITEM : "se agrega a"
    CART ||--|{ CART_ITEM : "contiene"
    ORDER ||--|{ ORDER_ITEM : "contiene"
    ORDER ||--o{ PAYMENT : "se paga con"
    ORDER ||--o{ ORDER_STATUS_HISTORY : "registra"
    PRODUCT ||--o{ ORDER_ITEM : "se vende en"

    PRODUCT {
        string isbn PK
        string title
        string author
        string publisher
        string category
        decimal price
    }
    CART {
        uuid id PK
        string customer_id
    }
    CART_ITEM {
        uuid id PK
        uuid cart_id FK
        string isbn FK
        int qty
    }
    ORDER {
        uuid id PK
        string customer_id
        string delivery_type
        uuid pickup_source_id
        string status
    }
    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        string isbn FK
        int qty
        decimal unit_price
    }
    PAYMENT {
        uuid id PK
        uuid order_id FK
        string idempotency_key
        string status
        decimal amount
    }
    ORDER_STATUS_HISTORY {
        uuid id PK
        uuid order_id FK
        string status
        datetime changed_at
    }
```

> Es un modelo **conceptual**; el detalle (tipos, índices y restricciones) se definirá en las especificaciones de cada módulo.

## 7. Arquitectura de despliegue

```mermaid
flowchart TB
    NET(("Internet"))

    subgraph PROXY["VM de Proxy - opcional, 3ra VM"]
        NGXB["Nginx balanceador<br/>TLS Let's Encrypt"]
    end

    subgraph APP["VM de Aplicaciones - Debian mínimo"]
        NGX["Nginx"]
        ECO["E-commerce"]
        INV["Inventario"]
        IDE["Identidad"]
        ASI["Asistente de Compra"]
        RED[("Redis")]
    end

    subgraph DAT["VM de Datos - Debian mínimo"]
        DBV[("PostgreSQL Ventas")]
        DBI[("PostgreSQL Inventario")]
        BKP["Backup periódico<br/>BD de Inventario"]
    end

    NET -->|"HTTPS 443"| NGX
    NET -.->|"Opcional"| NGXB
    NGXB -.-> NGX
    NGX --> ECO
    NGX --> INV
    NGX --> IDE
    NGX --> ASI
    ECO --> RED
    INV --> RED
    ECO --> DBV
    INV --> DBI
    DBI --> BKP
```

| VM | Contenedores | Entorno local | Oracle Cloud Free Tier (ARM) |
| --- | --- | --- | --- |
| **Aplicaciones** | Nginx, E-commerce, Inventario, Identidad, Asistente, Redis | 2 vCPU / 4 GB | 1 OCPU / 6 GB |
| **Datos** | PostgreSQL Ventas, PostgreSQL Inventario | 2 vCPU / 4 GB | 1 OCPU / 6 GB |
| **Proxy** (opcional) | Nginx balanceador, TLS | 1 vCPU / 1–2 GB | — |

**Criterios de despliegue**

- Debian estable en instalación mínima, **sin entorno gráfico**, en todas las VMs.
- Docker con imágenes **multi-arch** (x86_64 y ARM64) y Docker Compose con un manifiesto por entorno.
- La instancia pública corre en Oracle Cloud; el entorno local se reserva para desarrollo y pruebas.

## 8. Seguridad (visión inicial)

| Medida | Descripción |
| --- | --- |
| Acceso por SSH | Solo con llave pública, sin contraseña. |
| Firewall del SO | `nftables` o `ufw`, exponiendo únicamente HTTP/HTTPS y SSH. |
| TLS | Gestionado con Let's Encrypt en el proxy inverso. |
| Autenticación y autorización | JWT / OAuth 2.0 con rol y alcance de sucursal codificados en el token. |
| Respaldos | Backup periódico de la BD de Inventario, de mayor impacto operativo si se pierde. |
| Secretos | Claves de pago y de LLM como variables de entorno, fuera del repositorio. |
| Datos de pago | Procesados por la pasarela; BabelHub no almacena datos de tarjeta. |
| Protección de datos personales | Cumplimiento de la Ley N.° 29733. |

## 9. Decisiones de arquitectura y atributos de calidad

| Atributo | Cómo lo atiende la arquitectura |
| --- | --- |
| Consistencia de inventario | Servicio y BD dedicados, reservas transaccionales y stock vendible = físico − reservas. |
| Rendimiento | Caché Redis para disponibilidad y catálogo. |
| Disponibilidad | Reinicio automático de contenedores, instancia pública en Oracle Cloud y backups. |
| Escalabilidad | Cada sucursal es una fuente nueva; recursos asignables por contenedor. |
| Modificabilidad | Límites claros entre servicios, contratos REST estables y pagos tras una interfaz. |
| Seguridad | JWT con alcance de sucursal, TLS, firewall y secretos fuera del código. |
| Portabilidad | Imágenes multi-arch y Docker Compose. |

El detalle de cada decisión (AD-01 a AD-10) está en [`06-drivers-arquitectonicos.md`](../analisis-de-sistema/06-drivers-arquitectonicos.md).

## 10. Estrategia de pruebas

| Nivel | Herramienta | Qué valida |
| --- | --- | --- |
| Unitarias | xUnit | Lógica aislada (ej. cálculo de stock vendible). |
| Integración | xUnit + Testcontainers | BD propia de cada servicio y comunicación REST E-commerce ↔ Inventario. |
| End-to-End | Playwright | Búsqueda → compra con recojo → cambio de estado del pedido. |
| Carga | k6 | Picos de tráfico, especialmente sobre el Servicio de Inventario. |

## 11. Pendientes para la siguiente etapa

- Especificaciones ejecutables por funcionalidad bajo **Spec-Driven Development** (selección de fuente, cálculo de stock vendible, traslados).
- Detalle de la interfaz REST E-commerce ↔ Inventario (contratos OpenAPI).
- Manifiestos de Docker Compose por entorno (local y Oracle Cloud).
- Especificación funcional por pantalla (cliente, sucursal y administración central).
- Definición del proveedor de notificaciones y del operador logístico.
