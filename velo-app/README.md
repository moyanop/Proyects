<div align="center">

# 🛵 Velo — Plataforma de Delivery

**Marketplace de delivery de comida que conecta comercios, clientes y repartidores**

Pedidos en tiempo real · Pagos con Mercado Pago · Tracking GPS · 3 apps móviles · Panel de administración

---

[![React Native](https://img.shields.io/badge/React_Native_0.81-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo_SDK_54-000020?style=for-the-badge&logo=expo&logoColor=white)](https://expo.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python_3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL_15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis_7-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?style=for-the-badge&logo=socketdotio&logoColor=white)](https://socket.io/)

<br>

[![App Store](https://img.shields.io/badge/App_Store-0D96F6?style=for-the-badge&logo=app-store&logoColor=white)]()
[![Play Store](https://img.shields.io/badge/Google_Play-414141?style=for-the-badge&logo=google-play&logoColor=white)]()
[![Web](https://img.shields.io/badge/veloapp.com.ar-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white)](https://veloapp.com.ar)

</div>

---

## 📋 Tabla de Contenidos

- [Sobre el proyecto](#-sobre-el-proyecto)
- [Funcionalidades Principales](#-funcionalidades-principales)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [Diagrama de Flujo](#-diagrama-de-flujo)
- [Diagrama de Base de Datos](#-diagrama-de-base-de-datos)
- [Stack Tecnológico](#-stack-tecnológico)
- [Capturas de Pantalla](#-capturas-de-pantalla)

---

## 💡 Sobre el proyecto

Velo es una plataforma completa de delivery de comida diseñada para el mercado argentino. Conecta tres actores clave: **clientes** que realizan pedidos, **comercios** que venden sus productos y **repartidores** que realizan las entregas. La plataforma está publicada en **App Store** y **Google Play Store**, y opera en producción bajo el dominio **veloapp.com.ar**.

El sistema se compone de un monorepo con **3 aplicaciones móviles independientes**, un **panel web de administración** con sub-panel para comercios, y un **backend con comunicación en tiempo real vía Socket.IO**.

Problemas que resuelve:

- **Operación en tiempo real**: Socket.IO con Redis permite tracking de pedidos, notificaciones instantáneas y coordinación entre los tres actores.
- **Pagos integrados**: Mercado Pago con OAuth por comercio, split de pagos automático (subtotal al comercio, comisión a la plataforma) y soporte de efectivo/transferencia.
- **Logística inteligente**: Cálculo de costos de envío con Google Maps Distance Matrix, zonas de cobertura configurables y estimación de tiempos.
- **Escalabilidad**: Worker dedicado con APScheduler y leader election vía Redis para entornos multi-nodo.

---

## ✨ Funcionalidades Principales

**App Velo — Cliente (iOS & Android)**

- Exploración de comercios por localidad, carrito con extras/combos y checkout (Mercado Pago, efectivo, transferencia).
- Tracking de pedido en tiempo real vía Socket.IO con calificaciones y notificaciones push.
- Gestión de direcciones con geocodificación y sistema de promociones/cupones.

**App Velo Comercio — Comerciante (iOS & Android)**

- Recepción y gestión de pedidos en tiempo real con chat integrado y dashboard de métricas.
- CRUD de productos con opciones, extras, imágenes, horarios de atención y zonas de cobertura.
- Vinculación con Mercado Pago vía OAuth y gestión de promociones.

**App Velo Repartidores — Driver (iOS & Android)**

- Dashboard de pedidos asignados con escáner QR para verificación de entrega.
- Chat con el cliente e historial de entregas con calificaciones.

**Panel Web de Administración**

- Dashboard de analíticas, gestión de comercios/usuarios/repartidores y control de pedidos.
- Campañas de promociones, notificaciones push masivas y configuración de comisiones.
- Sub-panel para comercios con gestión autónoma de productos, pedidos y configuración.

---

## 🏗️ Arquitectura del Sistema

```mermaid
graph TB
    subgraph MobileApps["📱 Apps Móviles (Expo SDK 54)"]
        CLIENT["Velo<br/>App Cliente"]
        MERCHANT["Velo Comercio<br/>App Comerciante"]
        DRIVER["Velo Repartidores<br/>App Driver"]
    end

    subgraph WebPanel["🖥️ Panel Web"]
        ADMIN["Admin Dashboard<br/>React 19 + Vite"]
        MERCHANT_WEB["Sub-panel Comercio<br/>React 19 + Vite"]
    end

    subgraph Backend["⚙️ Backend (FastAPI)"]
        API["API REST<br/>28 módulos de endpoints"]
        SOCKET["Socket.IO Server<br/>Eventos en tiempo real"]
        WORKER["Worker (APScheduler)<br/>Tareas programadas"]
        SERVICES["Services Layer"]
        REPOS["Repository Layer"]
    end

    subgraph Data["🗄️ Datos"]
        PG[("PostgreSQL 15+<br/>44 modelos")]
        REDIS[("Redis 7+<br/>Pub/Sub + Cache")]
    end

    subgraph External["🔗 Servicios Externos"]
        MP["Mercado Pago<br/>OAuth + Webhooks"]
        GMAPS["Google Maps<br/>Geocoding + Rutas"]
        FCM["Firebase<br/>Push + Auth"]
        VISION["Cloud Vision<br/>Moderación"]
    end

    CLIENT --> |REST + WebSocket| API
    MERCHANT --> |REST + WebSocket| API
    DRIVER --> |REST + WebSocket| API
    ADMIN --> |REST| API
    MERCHANT_WEB --> |REST| API

    API --> SERVICES
    SOCKET --> SERVICES
    SERVICES --> REPOS
    REPOS --> PG

    API --> SOCKET
    SOCKET --> REDIS
    WORKER --> SERVICES
    WORKER --> REDIS

    API --> MP
    API --> GMAPS
    API --> FCM
    API --> VISION
```

---

## 🔄 Diagrama de Flujo

Flujo principal de un pedido desde la perspectiva del cliente:

```mermaid
flowchart TD
    A["Cliente abre la app"] --> B{"¿Está autenticado?"}
    B -- No --> C["Login / Registro / Google"]
    C --> D["Verificación + Completar perfil"]
    D --> B
    B -- Sí --> E["Explorar comercios"]
    E --> F["Selecciona comercio"]
    F --> G["Arma el pedido<br/>Productos + Extras + Opciones"]
    G --> H["Carrito de compras"]
    H --> I{"¿Tiene dirección?"}
    I -- No --> J["Agregar dirección<br/>Geocodificación Google Maps"]
    J --> I
    I -- Sí --> K["Checkout"]
    K --> L{"Método de pago"}
    L -- Mercado Pago --> M["WebView Mercado Pago"]
    M --> N["Webhook confirma pago"]
    L -- Efectivo / Transferencia --> N
    N --> O["Pedido creado: PENDIENTE"]

    O --> P["Socket.IO → Comercio recibe pedido"]
    P --> Q["Comercio CONFIRMA"]
    Q --> R["Estado: PREPARANDO"]
    R --> S["Estado: LISTO"]
    S --> T["Repartidor asignado"]
    T --> U["Estado: EN CAMINO<br/>Tracking en tiempo real"]
    U --> V["Repartidor escanea código QR<br/>Verificación 4 dígitos"]
    V --> W["Estado: ENTREGADO"]
    W --> X["Cliente califica pedido"]

    style A fill:#4CAF50,color:#fff
    style O fill:#2196F3,color:#fff
    style W fill:#FF9800,color:#fff
    style N fill:#7C4DFF,color:#fff
```

**Máquina de estados del pedido:**

```mermaid
stateDiagram-v2
    [*] --> esperando_pago
    esperando_pago --> pendiente : Pago confirmado
    esperando_pago --> cancelado : Timeout / Fallo
    pendiente --> confirmado : Comercio acepta
    pendiente --> cancelado : Comercio rechaza
    confirmado --> preparando : Comercio prepara
    preparando --> listo : Pedido listo
    listo --> en_camino : Repartidor retira
    en_camino --> entregado : Código QR verificado
    confirmado --> cancelado : Cancelación
    preparando --> cancelado : Cancelación
```

---

## 🗃️ Diagrama de Base de Datos

Modelo entidad-relación simplificado con las entidades core del sistema (44 modelos en total, se muestran las principales):

```mermaid
erDiagram
    Usuario ||--o{ Pedido : "realiza / vende"
    Usuario ||--o{ Direccion : "posee"
    Usuario ||--o{ Calificacion : "califica"
    Usuario ||--o{ Notificacion : "recibe"
    Usuario ||--|| Comercio : "es dueño"
    Usuario ||--|| Repartidor : "es"

    Comercio ||--o{ Producto : "ofrece"
    Comercio ||--o{ Pedido : "recibe"
    Comercio ||--o{ HorarioComercio : "opera en"
    Comercio ||--o{ ZonaCobertura : "cubre"
    Comercio ||--o{ Promocion : "crea"
    Comercio }o--|| RubroComercio : "pertenece a"

    Producto }o--|| Categoria : "clasificado en"
    Producto ||--o{ DetallePedido : "vendido en"
    Producto ||--o{ ProductoImagen : "tiene"
    Producto ||--o{ GrupoOpciones : "personalizable con"

    Pedido ||--o{ DetallePedido : "contiene"
    Pedido ||--o{ Pago : "pagado con"
    Pedido ||--o{ EstadoPedido : "historial"
    Pedido ||--o{ MensajePedido : "chat"
    Pedido ||--o{ Calificacion : "calificado"
    Pedido }o--|| Direccion : "entrega en"
    Pedido }o--o| Repartidor : "entregado por"
    Pedido }o--o| Promocion : "aplica"

    Pago ||--o{ TransaccionMercadoPago : "procesada por"

    Usuario {
        uuid id PK
        string nombre
        string apellido
        string email UK
        string telefono
        enum rol "admin | cliente | comercio | repartidor | vendedor"
        string google_sub
        int token_version
    }

    Comercio {
        uuid id PK
        string nombre
        string direccion
        float lat
        float lng
        float costo_envio_base
        string mp_access_token
        float calificacion_promedio
        uuid usuario_id FK
    }

    Producto {
        uuid id PK
        string nombre
        string descripcion
        float precio
        int stock
        enum tipo "simple | bundle | unit_selection"
        json extras
        boolean is_delivery_available
        boolean is_takeaway_available
        uuid comercio_id FK
        uuid categoria_id FK
    }

    Pedido {
        uuid id PK
        string numero_pedido UK
        enum estado "esperando_pago | pendiente | confirmado | preparando | listo | en_camino | entregado | cancelado"
        enum order_type "DELIVERY | TAKE_AWAY"
        float subtotal
        float costo_envio
        float descuento
        float total
        float platform_fee
        string codigo_entrega
        uuid usuario_id FK
        uuid comercio_id FK
        uuid repartidor_id FK
    }

    Repartidor {
        uuid id PK
        string documento_identidad
        string tipo_vehiculo
        string patente
        boolean disponible
        boolean en_servicio
        float calificacion_promedio
        int total_entregas
        uuid usuario_id FK
    }

    Pago {
        uuid id PK
        enum metodo_pago "MERCADOPAGO | EFECTIVO | TRANSFERENCIA_ALIAS | TARJETA"
        enum estado "pendiente | aprobado | rechazado"
        float monto
        string payment_id_externo
        uuid pedido_id FK
    }

    Direccion {
        uuid id PK
        string calle
        string numero
        float lat
        float lng
        string alias
        boolean es_principal
        uuid usuario_id FK
    }

    Promocion {
        uuid id PK
        enum tipo "producto | codigo_comercio | codigo_categoria | codigo_global | combo"
        float descuento_porcentaje
        string codigo UK
        json productos_combo
        uuid comercio_id FK
    }
```

---

## 🛠️ Stack Tecnológico

**App Móvil (x3)**: React Native 0.81, Expo SDK 54, TypeScript, NativeWind, React Navigation 7.  
**Panel Web**: React 19, Vite 7, TypeScript, TailwindCSS, Recharts, Leaflet.  
**Backend**: Python 3.11, FastAPI, SQLAlchemy 2.0, Pydantic v2, Socket.IO, Redis.  
**Base de Datos**: PostgreSQL 15+.  
**Integraciones**: Mercado Pago (OAuth + split payments), Google Maps API, Firebase (FCM + Auth), Google Cloud Vision.  
**Infraestructura**: VPS Ubuntu, Gunicorn + Nginx, systemd.

---

## 📸 Capturas de Pantalla

### 1. Exploración de Comercios

<img src="./images/pantalla_principal_cliente.jpeg" alt="Comercios" width="300">

_Pantalla principal de exploración de comercios disponibles por localidad._

### 2. Detalle de Pedido y Tracking

<img src="./images/seguimiento_pedido.jpeg" alt="Tracking" width="300">
<img src="./images/chat_comercio.jpeg" alt="Chat" width="300">

_Seguimiento en tiempo real del pedido con estados y chat integrado._

### 3. Panel de Comercio

<img src="./images/pedidos_kanban.png" alt="Panel Comercio" width="600">
<img src="./images/dashboard_comercio_web.png" alt="Panel Comercio" width="600">
<img src="./images/productos_web.png" alt="Panel Comercio" width="600">

_Dashboard del comerciante con pedidos entrantes, métricas y gestión de productos._

### 4. Panel de Administración

<img src="./images/dashboard_admin_web.png" alt="Panel Comercio" width="600">

_Dashboard administrativo con analíticas de pedidos, ingresos y usuarios activos._
