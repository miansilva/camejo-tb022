# Módulo 07: Arquitectura de desarrollo backend y APIs RESTful

Guía de estudio completa para dominar la arquitectura **cliente-servidor**, el protocolo **HTTP/HTTPS**, los principios de diseño de **APIs RESTful** y la arquitectura en capas.

---

## 1. Arquitectura web cliente-servidor

El modelo cliente-servidor es un patrón de arquitectura distribuida donde las tareas se dividen entre los proveedores de recursos (*Servidores*) y los demandantes de servicios (*Clientes*).

```
┌─────────────────────────┐               Petición HTTP (Request)              ┌─────────────────────────┐
│         CLIENTE         ├───────────────────────────────────────────────────►│        SERVIDOR         │
│ (Browser / React / App) │                                                    │    (Node.js / Python)   │
│                         │◄───────────────────────────────────────────────────┤                         │
└─────────────────────────┘               Respuesta HTTP (Response)            └────────────┬────────────┘
                                                                                            │ Consulta SQL
                                                                                            ▼
                                                                               ┌─────────────────────────┐
                                                                               │      BASE DE DATOS      │
                                                                               │       (PostgreSQL)      │
                                                                               └─────────────────────────┘
```

---

## 2. Protocolo HTTP (Hypertext Transfer Protocol)

HTTP es el protocolo de capa de aplicación sobre TCP/IP que rige el intercambio de datos en la Web.

### Estructura interna de una petición HTTP (Request)

```http
GET /api/v1/albumes/12 HTTP/1.1
Host: api.camejo.edu.ar
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1Ni...

[Cuerpo de la petición / Body (Vacío en GET)]
```

### Estructura interna de una respuesta HTTP (Response)

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 142
Access-Control-Allow-Origin: *

{
  "id": 12,
  "nombre": "Artaud",
  "lanzamiento": 1973,
  "ranking": 1
}
```

### Métodos HTTP (Verbos)

| Método | Propósito | ¿Seguro? | ¿Idempotente? |
| :---: | :--- | :---: | :---: |
| **`GET`** | Recuperar la representación de un recurso existente. | **Sí** | **Sí** |
| **`POST`** | Crear un nuevo recurso en el servidor. | No | No |
| **`PUT`** | Reemplazar por completo un recurso existente con la carga recibida.| No | **Sí** |
| **`PATCH`** | Modificar parcialmente campos de un recurso existente. | No | No |
| **`DELETE`** | Eliminar un recurso del servidor. | No | **Sí** |

- **Método seguro (*Safe Method*)**: No altera el estado interno del servidor (operación de solo lectura).
- **Método idempotente**: Realizar la misma petición $N$ veces consecutivas produce exactamente el mismo estado final en el servidor que realizarla una sola vez.

### Clasificación de códigos de estado HTTP

#### `2xx` Éxito
- **`200 OK`**: Petición procesada exitosamente.
- **`201 Created`**: Recurso creado exitosamente (respuesta habitual a un `POST`).
- **`204 No Content`**: Éxito sin cuerpo en la respuesta (respuesta habitual a un `DELETE`).

#### `3xx` Redirección
- **`301 Moved Permanently`**: El recurso cambió de dirección URL definitivamente.
- **`304 Not Modified`**: El recurso no cambió desde la última captura en caché.

#### `4xx` Errores del cliente (Petición inválida o sin permisos)
- **`400 Bad Request`**: Sintaxis inválida o fallas de validación en los datos enviados.
- **`401 Unauthorized`**: Autenticación requerida. Falta el token de acceso o es inválido.
- **`403 Forbidden`**: El cliente está autenticado pero NO posee permisos para acceder al recurso.
- **`404 Not Found`**: El recurso solicitado no existe en la dirección URL especificada.
- **`409 Conflict`**: Conflicto de estado (ej. intentar registrar un email que ya existe).

#### `5xx` Errores del servidor (Fallas no controladas)
- **`500 Internal Server Error`**: Excepción no capturada en el código del servidor.
- **`502 Bad Gateway`**: El servidor actuando de proxy recibió una respuesta inválida del servicio interno.
- **`503 Service Unavailable`**: El servidor está sobrecargado o en mantenimiento.

---

## 3. Estilo arquitectónico REST (Representational State Transfer)

REST es un conjunto de restricciones arquitectónicas propuesto por **Roy Fielding** en el año 2000.

### Principios fundamentales de REST

1. **Diseño centrado en recursos (URIs)**: Los endpoints identifican *recursos* en plural (sustantivos), NO acciones (verbos).
   - ✅ **Correcto**: `GET /api/v1/bandas`, `POST /api/v1/bandas`, `DELETE /api/v1/bandas/4`
   - ❌ **Incorrecto**: `GET /api/v1/obtenerBandas`, `POST /api/v1/borrarBanda?id=4`
2. **Representaciones estándar**: El cliente y el servidor intercambian representaciones de los recursos en formato estándar estructurado (habitualmente `JSON`).
3. **Sin estado (Statelessness)**: El servidor no guarda sesión en memoria entre peticiones. Cada petición enviada por el cliente debe contener toda la información de autenticación necesaria (ej. Token JWT).
4. **CORS (Cross-Origin Resource Sharing)**: Mecanismo de seguridad ejecutado por los navegadores que bloquea peticiones HTTP realizadas desde un origen (dominio/puerto) distinto al de la API, a menos que el servidor envíe las cabeceras `Access-Control-Allow-Origin`.

---

## 4. Arquitectura backend en capas (Layered Architecture)

Para asegurar la mantenibilidad, legibilidad y facilidad de prueba del código backend, se aplica la separación de responsabilidades en 3 capas principales:

```
┌─────────────────────────────────────────────────────────────┐
│                    CAPA CONTROLLER (HTTP)                   │
│  - Recibe HTTP Request y extrae parámetros/body.            │
│  - Valida datos de entrada (DTOs).                          │
│  - Retorna HTTP Response y código de estado.                │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                    CAPA SERVICE (NEGOCIO)                   │
│  - Contiene la lógica de negocio pura.                      │
│  - Reglas de validación comercial y cálculos.               │
│  - Orquesta múltiples llamadas a repositorios o servicios.  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                   CAPA REPOSITORY (DATOS)                   │
│  - Ejecuta consultas SQL puras a la base de datos o usa ORM.│
│  - Aísla la tecnología de persistencia del resto de la app. │
└─────────────────────────────────────────────────────────────┘
```

### Ejemplo de código en capas (Node.js + Express)

```javascript
// 1. Controller Layer: Manejo HTTP
app.get('/api/v1/bandas/:id', async (req, res, next) => {
    try {
        const { id } = req.params;
        const banda = await bandaService.obtenerPorId(id);
        
        if (!banda) {
            return res.status(404).json({ error: 'Banda no encontrada' });
        }
        
        return res.status(200).json(banda);
    } catch (error) {
        next(error); // Middleware global de manejo de excepciones
    }
});
```

---

## 5. Autenticación con JWT (JSON Web Tokens)

Un **JWT** (RFC 7519) es un estándar compacto y autosuficiente para transmitir información de autenticación entre partes como un objeto JSON firmado digitalmente.

### Estructura de un token JWT

Un token JWT consta de 3 partes separadas por puntos (`Header.Payload.Signature`):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IlNpbHZhIiwicm9sZSI6ImFkbWluIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
└──────────────────┬─────────────────┘└──────────────────────────┬───────────────────────────┘└──────────────────┬──────────────────┘
            1. Header (Algoritmo)                        2. Payload (Datos de usuario)                     3. Signature (Firma secreta)
```

1. **Header**: Indica el algoritmo de firma (ej. `HS256`, `RS256`) y tipo de token.
2. **Payload**: Contiene las declaraciones (*claims*) sobre el usuario (`sub`, `nombre`, `rol`, fecha de expiración `exp`). **Nota**: Está codificado en Base64, NO encriptado (no guardar datos sensibles como contraseñas en el payload).
3. **Signature**: Firma criptográfica generada por el servidor combinando el Header, Payload y una clave secreta privada. Garantiza que el token no ha sido manipulado por el cliente.
