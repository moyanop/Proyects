<div align="center">

# 🛢️ Sistema de Gestión — Lubricentro Asensio

**Plataforma integral para la operación diaria de un lubricentro**

Control de inventario · Punto de venta · Facturación electrónica AFIP · Servicios vehiculares · Marketing predictivo

---

[![React](https://img.shields.io/badge/React_19-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactjs.org/)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white)](https://www.prisma.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

</div>

---

## 📋 Tabla de Contenidos

- [Sobre mi proyecto](#-sobre-mi-proyecto)
- [Funcionalidades Principales](#-funcionalidades-principales)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [Diagrama de Base de Datos](#-diagrama-de-base-de-datos)
- [Stack Tecnológico](#-stack-tecnológico)
- [Capturas de Pantalla](#-capturas-de-pantalla)

---

## 💡 Sobre mi proyecto

Sistema diseñado para digitalizar y optimizar la operación diaria de un lubricentro, cubriendo todo el flujo comercial y técnico del negocio: desde la carga de productos e inventario, pasando por el punto de venta con generación de remitos, hasta el historial técnico de cada vehículo con recordatorios automáticos de mantenimiento.

El sistema resuelve problemas reales del rubro:

- **Visibilidad del costo**: Los operarios ven un costo inverso calculado a partir del margen, protegiendo los márgenes reales del negocio.
- **Trazabilidad completa**: Cada movimiento de stock, venta o servicio queda registrado.
- **Marketing predictivo**: El sistema calcula automáticamente cuándo un vehículo necesita su próximo service basándose en el promedio de km diarios recorridos.
- **Facturación legal**: Integración directa con ARCA/AFIP para emitir Facturas C electrónicas con CAE.

---

## ✨ Funcionalidades Principales

- **📦 Inventario Inteligente**: CRUD de productos, control de stock con alertas, código de barras, actualización masiva y costo inverso para operarios.
- **🧾 Punto de Venta (POS)**: Generación de remitos, descuentos, vinculación de clientes y facturación electrónica AFIP directa.
- **🔧 Servicios y Taller**: Historial por vehículo, consumo de productos de inventario, patrón "Snapshot" para congelar precios.
- **🚗 CRM y Marketing**: Cálculo de km promedio diario, estimación de próximo service y recordatorios automatizados por WhatsApp/Email.
- **📅 Reservas Online**: Sistema de turnos vinculados a servicios y vehículos.
- **📈 Analíticas**: Dashboard de ventas y gráficos interactivos del rendimiento del negocio.

---

## 🏗️ Arquitectura del Sistema

```mermaid
graph TB
    subgraph Cliente["🖥️ Frontend"]
        REACT["React 19 + Vite<br/>React Query + Router"]
    end

    subgraph API["⚙️ Backend"]
        EXPRESS["Node.js + Express<br/>Arquitectura de Capas"]
        CRON["node-cron<br/>(Background Jobs)"]
    end

    subgraph Data["🗄️ Base de Datos"]
        PRISMA["Prisma ORM"]
        PG[("PostgreSQL")]
    end

    REACT --> |Axios REST| EXPRESS
    EXPRESS --> PRISMA
    PRISMA --> PG
    CRON --> EXPRESS
```

---

## 🗃️ Diagrama de Base de Datos

Diagrama simplificado con las entidades core del sistema (excluyendo catálogos y entidades secundarias):

```mermaid
erDiagram
    User ||--o{ Vehicle : "posee"
    User ||--o{ Sale : "vende/compra"

    Vehicle ||--o{ ServiceHistory : "historial de servicios"

    Sale ||--o{ SaleItem : "contiene"
    Sale ||--|| Invoice : "factura AFIP"

    ServiceHistory ||--o{ ServiceHistoryItem : "detalle"

    Product ||--o{ SaleItem : "vendido en"
    Product ||--o{ ServiceHistoryItem : "consumido en"
    Product ||--o{ InventoryLog : "movimientos"

    User {
        string name
        string email
        enum role "ADMIN | OPERARIO | MECHANIC | USER"
    }

    Vehicle {
        string license UK
        string brand
        int averageDailyKm
        date nextServiceEstimatedDate
    }

    Product {
        string barcode UK
        string name
        float costPrice
        float sellingPrice
        int stockCurrent
    }

    Sale {
        string saleNumber UK
        float total
        uuid userId FK "Vendedor"
    }

    ServiceHistory {
        datetime fechaServicio
        int kilometraje
        uuid vehicleId FK
    }
```

---

## 🛠️ Stack Tecnológico

**Frontend**: React 19, Vite, React Router v7, React Query (TanStack), React Bootstrap, Recharts.  
**Backend**: Node.js, Express.js, TypeScript, Prisma ORM, Zod, JWT (Autenticación persistida en localStorage).  
**Base de Datos**: PostgreSQL.  
**Integraciones**: AFIP SDK (Factura Electrónica), Resend (Emails).  
**DevOps**: Docker & Docker Compose (Multi-stage builds), Nginx.

---

## 📸 Capturas del Sistema

### 1. Punto de Venta (POS) y Facturación

<!-- ![Punto de Venta](./images/pos.png) -->

_Panel principal de ventas, carrito de compras y generación de remitos/facturas._

### 2. Gestión de Inventario

<!-- ![Inventario](./images/inventario.png) -->

_Control de productos, stock, precios y herramientas de actualización masiva._

### 3. Ficha Técnica y Servicios

<!-- ![Ficha Tecnica](./images/ficha-tecnica.png) -->

_Historial de mantenimiento del vehículo, observaciones del mecánico y productos consumidos._

### 4. Dashboard de Analíticas

<!-- ![Analytics](./images/analytics.png) -->

_Métricas del negocio, evolución de ventas y estadísticas de desempeño._
