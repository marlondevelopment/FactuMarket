# Colección de Postman - FactuMarket

Este directorio contiene la colección de Postman con todos los endpoints de la API de FactuMarket.

## 📦 Archivo Incluido

- **FactuMarket.postman_collection.json**: Colección completa con todos los endpoints de los tres servicios

## 🚀 Cómo Importar la Colección

### Método 1: Importar desde archivo

1. Abre Postman
2. Haz clic en el botón **Import** (esquina superior izquierda)
3. Selecciona la pestaña **File**
4. Arrastra el archivo `FactuMarket.postman_collection.json` o haz clic en **Choose Files**
5. Haz clic en **Import**

### Método 2: Importar desde la carpeta

1. Abre Postman
2. Haz clic en **Import**
3. Arrastra toda la carpeta `postman/` directamente a Postman
4. Postman detectará automáticamente el archivo de colección

## 📋 Contenido de la Colección

La colección está organizada en tres carpetas principales:

### 1. **Invoices** (Facturas)
- `create` - Crear una nueva factura
  - Incluye ejemplo de respuesta exitosa (201)
- `show` - Obtener una factura específica
  - Incluye ejemplo de respuesta exitosa (200)
- `list` - Listar todas las facturas con paginación y filtros
  - Incluye ejemplo con filtros de fecha y paginación

### 2. **AuditEvents** (Eventos de Auditoría)
- `show` - Obtener eventos de auditoría por ID de entidad

### 3. **customers** (Clientes)
- `list` - Listar todos los clientes
  - Incluye ejemplo de respuesta exitosa (200)
- `create` - Crear un nuevo cliente
  - Incluye ejemplo de respuesta exitosa (201)
- `show` - Obtener un cliente específico
  - Incluye ejemplo de respuesta exitosa (200)

## 🌐 Configuración de URLs

Todos los requests están configurados para usar:

- **API Gateway**: `http://localhost:8080` (recomendado)
- **Puertos directos**: Algunos requests usan puertos directos (3001, 3002, 3003)

> **Nota**: Se recomienda usar el API Gateway en el puerto 8080 para centralizar todas las peticiones.

## 💡 Ejemplos de Uso

### Crear un Cliente
```http
POST http://localhost:8080/customers
Content-Type: application/json

{
  "customer": {
    "name": "pepito perez",
    "email": "pepito@gmail.com",
    "identification": 353125312,
    "address": "calle 1 # 1 - 5"
  }
}
```

### Crear una Factura
```http
POST http://localhost:8080/invoices
Content-Type: application/json

{
  "invoice": {
    "customer_id": 1,
    "amount": 2500,
    "issued_at": "2025-01-01",
    "status": "created"
  }
}
```

### Listar Facturas con Filtros
```http
GET http://localhost:8080/invoices?start_date=2025-01-01&end_date=2025-02-02&page=1&per_page=2
```

## 📝 Respuestas Guardadas

La colección incluye **ejemplos de respuestas guardadas** para varios endpoints, lo que te permite:

- Ver ejemplos de respuestas exitosas sin ejecutar los requests
- Entender la estructura de datos esperada
- Usar como referencia durante el desarrollo

## ⚙️ Variables de Entorno (Opcional)

Para facilitar el uso, puedes crear un Environment en Postman con las siguientes variables:

```json
{
  "gateway_url": "http://localhost:8080",
  "customer_service_url": "http://localhost:3001",
  "invoice_service_url": "http://localhost:3002",
  "audit_service_url": "http://localhost:3003"
}
```

Luego, puedes reemplazar las URLs en los requests por `{{gateway_url}}`, `{{customer_service_url}}`, etc.

## 🔄 Actualizar la Colección

Si realizas cambios en los requests y deseas actualizar el archivo JSON:

1. En Postman, haz clic derecho en la colección **FactuMarket**
2. Selecciona **Export**
3. Elige **Collection v2.1** (recomendado)
4. Guarda el archivo reemplazando `FactuMarket.postman_collection.json`

## 🆘 Solución de Problemas

### Error de conexión
- Verifica que Docker y todos los servicios estén corriendo: `docker-compose ps`
- Asegúrate de que los puertos no estén ocupados

### Error 404 Not Found
- Verifica que estés usando la URL correcta (Gateway: 8080 o puertos directos)
- Revisa que las rutas en los requests sean correctas

### Error 422 Unprocessable Entity
- Verifica que el formato del JSON sea correcto
- Asegúrate de que todos los campos requeridos estén presentes
- Para facturas, verifica que el `customer_id` exista en la base de datos

---

**Nota**: Esta colección fue creada y probada con la configuración del proyecto FactuMarket. Asegúrate de tener los servicios corriendo antes de ejecutar los requests.
