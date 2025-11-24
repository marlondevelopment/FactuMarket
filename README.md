# FactuMarket

FactuMarket es una aplicación basada en microservicios diseñada para gestionar clientes, facturas y auditoría de eventos. El proyecto utiliza Docker para la orquestación de contenedores y consta de tres servicios principales desarrollados en Ruby on Rails con un API Gateway (NGINX) que centraliza las peticiones.

## 📋 Tabla de Contenidos

- [Arquitectura](#arquitectura)
- [Requisitos Previos](#requisitos-previos)
- [Configuración y Ejecución](#configuración-y-ejecución)
- [API Gateway](#api-gateway)
- [Colección de Postman](#-colección-de-postman)
- [Endpoints de la API](#endpoints-de-la-api)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Tecnologías Utilizadas](#tecnologías-utilizadas)

## 🏗️ Arquitectura

El sistema está compuesto por los siguientes servicios:

### **Customer Service** (Servicio de Clientes)
- **Puerto Directo**: `3001`
- **Base de datos**: PostgreSQL (`customers_db`)
- **Función**: Gestiona la información de los clientes (creación, consulta individual y listado)
- **Características**:
  - Validaciones de datos
  - Paginación en listados
  - Auditoría automática de eventos

### **Invoice Service** (Servicio de Facturas)
- **Puerto Directo**: `3002`
- **Base de datos**: PostgreSQL (`invoices_db`)
- **Función**: Gestiona la creación y consulta de facturas
- **Características**:
  - Validación de clientes (verifica existencia en Customer Service)
  - Validación de montos (debe ser > 0)
  - Validación de fechas
  - Filtrado por rango de fechas
  - Paginación en listados
  - Auditoría automática de eventos

### **Audit Service** (Servicio de Auditoría)
- **Puerto Directo**: `3003`
- **Base de datos**: MongoDB (`audit_db`)
- **Función**: Registra todos los eventos de auditoría del sistema
- **Características**:
  - Registro de eventos de creación, consulta, errores
  - Consulta de eventos por entidad
  - Paginación en listados
  - Almacenamiento de metadata contextual

### **API Gateway** (NGINX)
- **Puerto**: `8080`
- **Función**: Centraliza todas las peticiones y las enruta a los servicios correspondientes
- **Beneficios**: 
  - Punto único de acceso
  - Balanceo de carga
  - Simplifica la configuración de clientes

## 📦 Requisitos Previos

- [Docker](https://www.docker.com/get-started) (versión 20.10 o superior)
- [Docker Compose](https://docs.docker.com/compose/install/) (versión 1.29 o superior)

## 🚀 Configuración y Ejecución

Sigue estos pasos para levantar el proyecto y configurar las bases de datos.

### 1. Iniciar los servicios

Ejecuta el siguiente comando en la raíz del proyecto para construir y levantar los contenedores:

```bash
docker-compose up --build
```

Espera a que todos los servicios estén activos. Verás logs indicando que los servidores Rails y las bases de datos están listos.

### 2. Inicialización de Base de Datos (Scripts)

Dado que el proyecto utiliza scripts SQL y JS directos en lugar de migraciones de Rails para la estructura inicial, **es necesario ejecutar manualmente los scripts de inicialización** una vez que los contenedores estén corriendo.

Abre una nueva terminal y ejecuta los siguientes comandos:

#### Para Customer Service (PostgreSQL)
Crea la tabla `customers`:

```bash
docker exec -it postgres_customers psql -U admin -d customers_db -f /scripts/customers.sql
```

#### Para Invoice Service (PostgreSQL)
Crea la tabla `invoices`:

```bash
docker exec -it postgres_invoices psql -U admin -d invoices_db -f /scripts/invoices.sql
```

#### Para Audit Service (MongoDB)
Crea la colección e índices para `audit_events`:

```bash
docker exec -it mongo-db mongosh /scripts/audit.js
```

> **Nota:** Si el comando `mongosh` no está disponible, intenta con `mongo`.

## 🌐 API Gateway

Todas las peticiones se pueden centralizar a través del API Gateway en el puerto **8080**:

```
http://localhost:8080
```

El gateway redirige automáticamente las peticiones según la ruta:
- `/customers/*` → Customer Service (puerto 3001)
- `/invoices/*` → Invoice Service (puerto 3002)
- `/audit_events/*` → Audit Service (puerto 3003)

### Ventajas de usar el API Gateway:
- **Único punto de entrada**: No necesitas recordar múltiples puertos
- **Escalabilidad**: Facilita añadir balanceo de carga
- **Seguridad**: Centraliza autenticación y autorización (futuro)

## 📮 Colección de Postman

El proyecto incluye una **colección completa de Postman** con todos los endpoints documentados y listos para usar.

### 📦 Ubicación

La colección se encuentra en: [`postman/FactuMarket.postman_collection.json`](postman/FactuMarket.postman_collection.json)

### 🚀 Importar en Postman

1. Abre Postman
2. Haz clic en **Import** (esquina superior izquierda)
3. Arrastra el archivo `FactuMarket.postman_collection.json` o selecciona **Choose Files**
4. Haz clic en **Import**

### ✨ Qué incluye

La colección está organizada en **3 carpetas** con todos los endpoints:

- **Customers** (Clientes)
  - Crear cliente
  - Listar clientes
  - Obtener cliente por ID
  
- **Invoices** (Facturas)
  - Crear factura
  - Listar facturas (con paginación y filtros de fecha)
  - Obtener factura por ID

- **AuditEvents** (Auditoría)
  - Obtener eventos de auditoría por entidad

### 💡 Ventajas de usar la colección

- ✅ **Ejemplos de respuestas guardadas** para referencia
- ✅ **Requests preconfigurados** listos para ejecutar
- ✅ **Body de ejemplo** con datos válidos
- ✅ **URLs configuradas** para API Gateway y puertos directos
- ✅ **Sin configuración adicional** - importa y usa

### 📖 Documentación completa

Para más detalles sobre cómo usar la colección, consulta: [postman/README.md](postman/README.md)


## 📡 Endpoints de la API

### Customer Service

#### Crear un cliente
```http
POST http://localhost:8080/customers
Content-Type: application/json

{
  "customer": {
    "name": "Juan Pérez",
    "email": "juan.perez@example.com",
    "identification": "123456789",
    "address": "Calle Principal #123"
  }
}
```

**Respuesta exitosa (201):**
```json
{
  "id": 1,
  "name": "Juan Pérez",
  "email": "juan.perez@example.com",
  "identification": "123456789",
  "address": "Calle Principal #123",
  "created_at": "2025-11-24T22:00:00.000Z",
  "updated_at": "2025-11-24T22:00:00.000Z"
}
```

#### Obtener un cliente
```http
GET http://localhost:8080/customers/1
```

#### Listar clientes (paginado)
```http
GET http://localhost:8080/customers?page=1&per_page=10
```

**Parámetros de paginación:**
- `page`: Número de página (por defecto: 1)
- `per_page`: Registros por página (por defecto: 25)

---

### Invoice Service

#### Crear una factura
```http
POST http://localhost:8080/invoices
Content-Type: application/json

{
  "invoice": {
    "customer_id": 1,
    "amount": 150.50,
    "issued_at": "2025-11-24",
    "status": "pending"
  }
}
```

**Validaciones automáticas:**
- ✅ customer_id debe existir en Customer Service
- ✅ amount debe ser mayor que 0
- ✅ issued_at debe ser una fecha válida

**Respuesta exitosa (201):**
```json
{
  "id": 1,
  "customer_id": 1,
  "amount": "150.50",
  "issued_at": "2025-11-24T00:00:00.000Z",
  "status": "pending",
  "created_at": "2025-11-24T22:00:00.000Z",
  "updated_at": "2025-11-24T22:00:00.000Z"
}
```

#### Obtener una factura
```http
GET http://localhost:8080/invoices/1
```

#### Listar facturas (paginado y filtrado)
```http
GET http://localhost:8080/invoices?page=1&per_page=10
```

**Filtrado por fechas:**
```http
GET http://localhost:8080/invoices?start_date=2025-11-01&end_date=2025-11-30
```

**Parámetros disponibles:**
- `page`: Número de página (por defecto: 1)
- `per_page`: Registros por página (por defecto: 25)
- `start_date`: Fecha de inicio (formato: YYYY-MM-DD)
- `end_date`: Fecha de fin (formato: YYYY-MM-DD)

---

### Audit Service

#### Crear un evento de auditoría
```http
POST http://localhost:8080/audit_events
Content-Type: application/json

{
  "audit_event": {
    "service": "customers",
    "action": "create",
    "entity_type": "Customer",
    "entity_id": "1",
    "message": "Customer created successfully",
    "metadata": {
      "user_ip": "192.168.1.1",
      "additional_info": "Manual creation"
    }
  }
}
```

> **Nota:** Los servicios Customer e Invoice registran automáticamente eventos de auditoría. Este endpoint está disponible para registros manuales si es necesario.

#### Obtener eventos de auditoría por entidad
```http
GET http://localhost:8080/audit_events/1
```

**Respuesta:**
```json
[
  {
    "id": "673f5a2e8f4b2c001234abcd",
    "service": "customers",
    "action": "create",
    "entity_type": "Customer",
    "entity_id": "1",
    "message": "Customer created",
    "metadata": { ... },
    "created_at": "2025-11-24T22:00:00.000Z"
  }
]
```

## 📁 Estructura del Proyecto

```
FactuMarket/
├── customer_service/          # Servicio de clientes
│   ├── app/
│   │   ├── controllers/
│   │   │   └── customers_controller.rb
│   │   ├── models/
│   │   │   └── customer.rb
│   │   └── lib/
│   │       └── audit_logger.rb
│   ├── db/scripts/
│   │   └── customers.sql      # Script de creación de tabla
│   ├── Dockerfile
│   └── Gemfile
│
├── invoice_service/           # Servicio de facturas
│   ├── app/
│   │   ├── controllers/
│   │   │   └── invoices_controller.rb
│   │   ├── models/
│   │   │   └── invoice.rb
│   │   └── lib/
│   │       └── audit_logger.rb
│   ├── db/scripts/
│   │   └── invoices.sql       # Script de creación de tabla
│   ├── Dockerfile
│   └── Gemfile
│
├── audit_service/             # Servicio de auditoría
│   ├── app/
│   │   ├── controllers/
│   │   │   └── audit_events_controller.rb
│   │   └── models/
│   │       └── audit_event.rb
│   ├── db/scripts/
│   │   └── audit.js           # Script de inicialización MongoDB
│   ├── Dockerfile
│   └── Gemfile
│
├── postman/                   # Colección de Postman
│   ├── FactuMarket.postman_collection.json
│   └── README.md              # Documentación de la colección
│
├── nginx.conf                 # Configuración del API Gateway
├── docker-compose.yml         # Orquestación de servicios
└── README.md                  # Este archivo
```

## 🛠️ Tecnologías Utilizadas

- **Ruby on Rails** 7.2.2 - Framework web para los microservicios
- **PostgreSQL** 16 - Base de datos relacional (Customers e Invoices)
- **MongoDB** 7 - Base de datos NoSQL (Audit)
- **Docker** - Containerización
- **Docker Compose** - Orquestación de contenedores
- **NGINX** - API Gateway y reverse proxy
- **Kaminari** - Gema de paginación para Rails

## 📝 Notas Adicionales

### Acceso Directo a los Servicios

Aunque se recomienda usar el API Gateway (puerto 8080), también puedes acceder directamente a cada servicio:

- **Customer Service**: `http://localhost:3001`
- **Invoice Service**: `http://localhost:3002`
- **Audit Service**: `http://localhost:3003`

### Bases de Datos

**PostgreSQL (Customers):**
- Host: `localhost:5433`
- Usuario: `admin`
- Password: `admin123`
- Base de datos: `customers_db`

**PostgreSQL (Invoices):**
- Host: `localhost:5434`
- Usuario: `admin`
- Password: `admin123`
- Base de datos: `invoices_db`

**MongoDB (Audit):**
- Host: `localhost:27017`
- Base de datos: `audit_db` (se crea automáticamente)

### Detener los Servicios

Para detener todos los contenedores:

```bash
docker-compose down
```

Para detener y eliminar los volúmenes (⚠️ esto borrará todos los datos):

```bash
docker-compose down -v
```

### Ver Logs

Para ver los logs de todos los servicios:

```bash
docker-compose logs -f
```

Para ver logs de un servicio específico:

```bash
docker-compose logs -f customer_service
```

## 🔍 Validaciones Implementadas

### Customer Service
- ✅ Nombre requerido
- ✅ Email requerido y formato válido
- ✅ Email único
- ✅ Identificación requerida y única

### Invoice Service
- ✅ Customer ID debe existir en Customer Service
- ✅ Monto debe ser mayor que 0
- ✅ Fecha de emisión debe ser válida
- ✅ Estado requerido

### Audit Service
- ✅ Servicio requerido
- ✅ Acción requerida
- ✅ Tipo de entidad requerido
- ✅ Mensaje requerido

## 🎯 Características Principales

1. **Arquitectura de Microservicios**: Cada servicio es independiente y puede escalarse por separado
2. **API Gateway**: Centralización de peticiones con NGINX
3. **Auditoría Completa**: Todos los eventos se registran automáticamente
4. **Validaciones Robustas**: Validaciones a nivel de modelo y controlador
5. **Comunicación entre Servicios**: Invoice Service valida clientes en Customer Service
6. **Paginación**: Todos los listados soportan paginación
7. **Filtrado Avanzado**: Filtrado de facturas por rango de fechas
8. **Persistencia de Datos**: Volúmenes Docker para mantener datos entre reinicios

---