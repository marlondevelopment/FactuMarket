# FactuMarket

FactuMarket es una aplicación basada en microservicios diseñada para gestionar clientes, facturas y auditoría de eventos. El proyecto utiliza Docker para la orquestación de contenedores y consta de tres servicios principales desarrollados en Ruby on Rails.

## Arquitectura

El sistema está compuesto por los siguientes servicios:

*   **Customer Service**: Gestiona la información de los clientes.
    *   Puerto: `3001`
    *   Base de datos: PostgreSQL (`customers_db`)
*   **Invoice Service**: Gestiona la creación y consulta de facturas.
    *   Puerto: `3002`
    *   Base de datos: PostgreSQL (`invoices_db`)
*   **Audit Service**: Registra eventos de auditoría.
    *   Puerto: `3003`
    *   Base de datos: MongoDB (`audit_db`)

## Requisitos Previos

*   [Docker](https://www.docker.com/get-started)
*   [Docker Compose](https://docs.docker.com/compose/install/)

## Configuración y Ejecución

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

## Uso

Una vez ejecutados los scripts, los servicios estarán listos para recibir peticiones HTTP en sus respectivos puertos:

*   **Customer Service**: `http://localhost:3001`
*   **Invoice Service**: `http://localhost:3002`
*   **Audit Service**: `http://localhost:3003`

## Estructura del Proyecto

*   `customer_service/`: Código fuente del servicio de clientes.
    *   `db/scripts/customers.sql`: Script de creación de tabla.
*   `invoice_service/`: Código fuente del servicio de facturas.
    *   `db/scripts/invoices.sql`: Script de creación de tabla.
*   `audit_service/`: Código fuente del servicio de auditoría.
    *   `db/scripts/audit.js`: Script de inicialización de Mongo.
*   `docker-compose.yml`: Definición de servicios y redes Docker.