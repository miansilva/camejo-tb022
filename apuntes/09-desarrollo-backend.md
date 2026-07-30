# Módulo 09: Desarrollo backend, arquitectura HTTP, APIs RESTful y Node.js/Express

Guía de estudio conceptual y técnica completa sobre la arquitectura **cliente-servidor, el protocolo HTTP/HTTPS, diseño de APIs RESTful, arquitectura en capas, Express.js y seguridad con JWT**.

---

## 1. Arquitectura web cliente-servidor

El patrón **Cliente-Servidor** es la piedra angular del desarrollo web distribuido. Divide las responsabilidades entre quienes consumen datos (**Clientes**) y quienes procesan la lógica de negocio y almacenan información (**Servidores**).

```
┌─────────────────────────┐            Petición HTTP (Request)           ┌─────────────────────────┐
│         CLIENTE         ├─────────────────────────────────────────────►│        SERVIDOR         │
│ (Navegador / React App) │                                              │   (Node.js + Express)   │
│                         │◄─────────────────────────────────────────────┤                         │
└─────────────────────────┘            Respuesta HTTP (Response)         └────────────┬────────────┘
                                                                                      │ Consulta SQL (pg)
                                                                                      ▼
                                                                         ┌─────────────────────────┐
                                                                         │      BASE DE DATOS      │
                                                                         │       (PostgreSQL)      │
                                                                         └─────────────────────────┘
```

---

## 2. Protocolo HTTP (HyperText Transfer Protocol)

HTTP es un protocolo stateless de capa de aplicación que rige el intercambio de mensajes entre cliente y servidor.

### Estructura interna de mensajes HTTP

#### Petición HTTP (*Request*)
```http
POST /api/v1/usuarios HTTP/1.1
Host: api.camejo.edu.ar
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1Ni...

{
  "nombre": "Gonzalo Martinez",
  "email": "gmartinez@gmail.com"
}
```

#### Respuesta HTTP (*Response*)
```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: *

{
  "id": 42,
  "nombre": "Gonzalo Martinez",
  "email": "gmartinez@gmail.com",
  "creadoEn": "2026-07-30T10:00:00Z"
}
```

### Métodos HTTP (Verbos) e idempotencia

| Método | Operación CRUD | ¿Seguro? | ¿Idempotente? | Descripción |
| :---: | :---: | :---: | :---: | :--- |
| **`GET`** | Read | **Sí** | **Sí** | Recupera datos. No modifica el estado del servidor. |
| **`POST`** | Create | No | No | Crea un nuevo recurso. Ejecutarlo $N$ veces crea $N$ recursos distintos. |
| **`PUT`** | Update (Total) | No | **Sí** | Reemplaza por completo el recurso. $N$ ejecuciones dejan exactamente el mismo resultado final. |
| **`PATCH`**| Update (Parcial)| No | No | Modifica parcialmente uno o más campos del recurso. |
| **`DELETE`**| Delete | No | **Sí** | Elimina el recurso indicado. |

- **Seguridad (*Safety*)**: Un método es seguro si su invocación es puramente de lectura y no altera el estado del servidor.
- **Idempotencia**: Un método es idempotente si ejecutar la misma solicitud una o mil veces consecutivas produce **exactamente el mismo estado final en el servidor**.

### Clasificación de códigos de estado HTTP

#### `2xx` Éxito
- **`200 OK`**: Solicitud procesada correctamente.
- **`201 Created`**: Recurso creado exitosamente (Respuesta habitual a `POST`).
- **`204 No Content`**: Procesamiento exitoso sin cuerpo en la respuesta (Habitual en `DELETE`).

#### `3xx` Redirección
- **`301 Moved Permanently`**: El recurso cambió de ubicación de forma definitiva.
- **`304 Not Modified`**: El cliente puede usar su versión cacheada.

#### `4xx` Errores del cliente
- **`400 Bad Request`**: Datos de entrada o sintaxis JSON inválida.
- **`401 Unauthorized`**: Falta token de autenticación o es inválido.
- **`403 Forbidden`**: Cliente autenticado pero sin permisos suficientes para el recurso.
- **`404 Not Found`**: El recurso o endpoint solicitado no existe.
- **`409 Conflict`**: Conflicto de estado (ej. email duplicado).

#### `5xx` Errores del servidor
- **`500 Internal Server Error`**: Excepción no capturada en el servidor.
- **`503 Service Unavailable`**: Servidor en mantenimiento o sobrecargado.

---

## 3. Principios de APIs RESTful

**REST (Representational State Transfer)** es un estilo arquitectónico formulado por Roy Fielding basado en 6 restricciones:

1. **Diseño orientado a recursos**: Las URIs identifican **recursos (sustantivos en plural)**, no acciones (verbos).
   - ✅ **Correcto**: `GET /api/v1/productos`, `DELETE /api/v1/productos/5`
   - ❌ **Incorrecto**: `GET /api/v1/obtenerProductos`, `POST /api/v1/eliminarProducto?id=5`
2. **Sin estado (*Statelessness*)**: El servidor no almacena sesiones del cliente en memoria. Cada solicitud debe incluir toda la información necesaria (ej. Token JWT en cabecera `Authorization`).
3. **Representaciones estandarizadas**: Intercambio de datos utilizando JSON estructurado.
4. **CORS (Cross-Origin Resource Sharing)**: Mecanismo de seguridad de navegadores. Si el frontend (`localhost:5173`) consume el backend (`localhost:3000`), el servidor debe habilitar las cabeceras `Access-Control-Allow-Origin`.

---

## 4. Arquitectura backend en capas (layered architecture)

Para asegurar la mantenibilidad y facilitar las pruebas unitarias, el backend se organiza en 3 capas de responsabilidad única:

```
┌───────────────────────────────────────────────────────────────┐
│                    CAPA CONTROLLER (HTTP)                     │
│  - Recibe HTTP Request, extrae req.params / req.body.         │
│  - Valida esquemas de entrada.                                │
│  - Responde con HTTP Response y Status Code correspondiente.  │
└───────────────────────────────────────────────────────────────┘
                               │
┌──────────────────────────────▼────────────────────────────────┐
│                    CAPA SERVICE (NEGOCIO)                     │
│  - Reglas de negocio puras, cálculos y validaciones.          │
│  - Orquesta múltiples repositorios.                           │
└───────────────────────────────────────────────────────────────┘
                               │
┌──────────────────────────────▼────────────────────────────────┐
│                  CAPA REPOSITORY (DATOS)                      │
│  - Ejecuta consultas SQL directamente a la BD.                │
│  - Aísla la tecnología de persistencia del resto de la app.   │
└───────────────────────────────────────────────────────────────┘
```

---

## 5. Desarrollo con Node.js y Express

### Aplicación completa con middlewares y ruteo

```javascript
import express from 'express';
import cors from 'cors';

const app = express();

// Middlewares globales
app.use(cors());
app.use(express.json()); // Parsing de bodies en formato JSON

// Middleware de logging
app.use((req, res, next) => {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    next();
});

// Definición de rutas (Controller Layer)
app.get('/api/v1/usuarios/:id', async (req, res, next) => {
    try {
        const { id } = req.params;
        const usuario = await obtenerUsuarioService(id);
        
        if (!usuario) {
            return res.status(404).json({ error: 'Usuario no encontrado' });
        }
        
        return res.status(200).json(usuario);
    } catch (error) {
        next(error); // Pasa la excepción al Middleware de manejo de errores
    }
});

// Middleware global de manejo de errores (Debe tener 4 parámetros)
app.use((err, req, res, next) => {
    console.error("Error no capturado:", err.stack);
    res.status(500).json({ error: 'Error interno del servidor' });
});

app.listen(3000, () => console.log("Servidor corriendo en puerto 3000"));
```

---

## 6. Autenticación con JWT (JSON Web Tokens)

Un **JWT** (RFC 7519) es un estándar abierto autosuficiente para transmitir declaraciones de identidad firmadas criptográficamente.

### Estructura de un token JWT

Un token JWT consiste en 3 secciones codificadas en Base64 separadas por puntos (`Header.Payload.Signature`):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NSIsIm5vbWJyZSI6IlNpbHZhIiwicm9sIjoiQURNSU4ifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
└──────────────────┬─────────────────┘└──────────────────────────┬───────────────────────────┘└──────────────────┬──────────────────┘
           1. Header (Algoritmo)                        2. Payload (Claims/Datos)                      3. Signature (Firma secreta)
```

1. **Header**: Especifica el tipo de token (`JWT`) y el algoritmo de firma (`HS256`, `RS256`).
2. **Payload**: Contiene las declaraciones (*claims*) de datos (`sub`, `nombre`, `rol`, expiración `exp`). **Atención**: Está codificado en Base64, NO encriptado (jamás poner claves ni contraseñas en el payload).
3. **Signature**: Firma creada mezclando Header + Payload + Clave Secreta en el servidor. Previene alteraciones o falsificaciones por parte del cliente.

### Middleware de autenticación con JWT

```javascript
import jwt from 'jsonwebtoken';

export const autenticarToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    // El formato esperado es: Bearer TOKEN_AQUI
    const token = authHeader && authHeader.split(' ')[1];

    if (!token) {
        return res.status(401).json({ error: 'Token de acceso requerido' });
    }

    jwt.verify(token, process.env.JWT_SECRET, (err, usuarioDecodificado) => {
        if (err) {
            return res.status(403).json({ error: 'Token inválido o expirado' });
        }
        
        req.usuario = usuarioDecodificado; // Inyecta los datos decodificados en la request
        next();
    });
};
```
