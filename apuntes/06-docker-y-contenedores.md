# Módulo 06: Virtualización, Docker y orquestación de contenedores

Guía de estudio conceptual y técnica completa sobre **contenedores, arquitectura Docker Engine, construcción optima con Dockerfile, multi-stage builds, persistencia de datos, redes y orquestación con Docker Compose**.

---

## 1. Virtualización tradicional vs. contenedores

```
┌─────────────────────────────────────────┐    ┌─────────────────────────────────────────┐
│       MÁQUINAS VIRTUALES (VMs)          │    │         CONTENEDORES (DOCKER)           │
├─────────────┬─────────────┬─────────────┤    ├─────────────┬─────────────┬─────────────┤
│  Aplicación │  Aplicación │  Aplicación │    │  Aplicación │  Aplicación │  Aplicación │
├─────────────┼─────────────┼─────────────┤    ├─────────────┼─────────────┼─────────────┤
│ Dep / Libs  │ Dep / Libs  │ Dep / Libs  │    │ Dep / Libs  │ Dep / Libs  │ Dep / Libs  │
├─────────────┼─────────────┼─────────────┤    ├─────────────┴─────────────┴─────────────┤
│  Guest OS   │  Guest OS   │  Guest OS   │    │             DOCKER ENGINE               │
├─────────────┴─────────────┴─────────────┤    ├─────────────────────────────────────────┤
│            HYPERVISOR                   │    │         KERNEL DEL SO HOST              │
├─────────────────────────────────────────┤    ├─────────────────────────────────────────┤
│         HARDWARE FÍSICO                 │    │         HARDWARE FÍSICO                 │
└─────────────────────────────────────────┘    └─────────────────────────────────────────┘
```

### Cuadro comparativo técnico

| Dimensión | Máquinas virtuales (Hypervisor) | Contenedores (Docker) |
| :--- | :--- | :--- |
| **Aislamiento** | Nivel de hardware (Hypervisor emula componentes). | Nivel de proceso (Namespaces y cgroups del Kernel). |
| **Sistema operativo** | Cada VM ejecuta su propio *Guest OS* completo. | Comparten el mismo Kernel del SO Host. |
| **Consumo de memoria**| Alto (Gigabytes por cada Guest OS). | Mínimo (Megabytes, solo el footprint del proceso). |
| **Tiempo de arranque** | Minutos (Bootstrapping del sistema operativo completo).| Milisegundos (Inicio directo del proceso empaquetado).|
| **Portabilidad** | Depende del formato del Hypervisor (OVA, VMDK). | Ultra portátil (Cualquier host con Docker Engine). |

### Mecanismos del Kernel Linux detrás de Docker
1. **Linux Namespaces (Aislamiento de visión)**:
   - `PID Namespace`: Aísla el árbol de procesos (el proceso del contenedor se ve como PID 1).
   - `NET Namespace`: Aísla las interfaces de red, tablas de ruteo y puertos.
   - `MNT Namespace`: Aísla los puntos de montaje del sistema de archivos.
   - `IPC`, `UTS`, `USER`: Aíslan memoria compartida, hostname y IDs de usuario.
2. **Control Groups - cgroups (Limitación de recursos)**:
   - Limitan y monitorean la cuota máxima de memoria RAM, uso de CPU, I/O de disco y sockets de red que puede consumir un contenedor.

---

## 2. Arquitectura de Docker Engine

- **Docker Client (`docker`)**: Herramienta de comandos CLI que emite instrucciones a la API REST de Docker.
- **Docker Daemon (`dockerd`)**: Proceso persistente en segundo plano en el Host que construye, ejecuta y gestiona imágenes, contenedores, volúmenes y redes.
- **Docker Registry (ej. Docker Hub)**: Repositorio remoto centralizado para almacenar y compartir imágenes de contenedores públicas o privadas.

---

## 3. Imágenes vs. contenedores y sistema de capas

- **Imagen Docker**: Plantilla de solo lectura (*Read-Only*) formada por una pila ordenada de capas inmutables de archivos (*Union File System* / Overlay2).
- **Contenedor Docker**: Instancia ejecutable en tiempo real de una imagen. Agrega una delgada **Capa de lectura y escritura (Read-Write Container Layer)** en la parte superior.

### Optimización de la caché de capas
Cada instrucción en un `Dockerfile` genera una nueva capa. Reordenar las instrucciones de lo **menos frecuente** a lo **más frecuente** acelera drásticamente el tiempo de compilación (*build*):

```dockerfile
# ❌ INEFICIENTE: Copia todo el código antes de instalar dependencias (Invalida la caché)
COPY . .
RUN npm install

# ✅ EFICIENTE: Instala dependencias reutilizando la caché si package.json no cambió
COPY package*.json ./
RUN npm ci
COPY . .
```

---

## 4. Guía completa de `Dockerfile` y construcción multi-stage

### Instrucciones principales
- **`FROM`**: Especifica la imagen base (ej. `node:20-alpine`, `python:3.11-slim`).
- **`WORKDIR`**: Define el directorio de trabajo interno.
- **`COPY` / `ADD`**: Copia archivos desde el Host hacia el sistema de archivos del contenedor.
- **`RUN`**: Ejecuta comandos en fase de construcción de la imagen.
- **`ENV`**: Define variables de entorno permanentes.
- **`EXPOSE`**: Documenta el puerto en el que escucha el contenedor.
- **`USER`**: Cambia el usuario de ejecución evitando ejecutar como `root` por seguridad.
- **`CMD` vs `ENTRYPOINT`**:
  - `CMD ["node", "app.js"]`: Comando y parámetros por defecto (Sobrescribible fácilmente al ejecutar `docker run`).
  - `ENTRYPOINT ["node"]`: Define el binario ejecutable fijo.

### Multi-stage builds (Imágenes de producción livianas)
Permite utilizar imágenes completas con compiladores para construir la aplicación y luego copiar **únicamente los artefactos resultantes** a una imagen base de producción ultra liviana:

```dockerfile
# ── ETAPA 1: Compilación (Build Stage) ──
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build  # Genera la carpeta /dist

# ── ETAPA 2: Producción (Production Stage) ──
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Copiar solo las dependencias de producción y los binarios compilados
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist

USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

---

## 5. Persistencia de datos: volumes vs. bind mounts

Los datos dentro de la capa R/W de un contenedor son **efímeros**. Si el contenedor se elimina, los datos desaparecen. Para persistir datos se utilizan dos estrategias:

```
┌──────────────────────────────────────────────────────────────┐
│                   SISTEMA DE ARCHIVOS HOST                   │
├──────────────────────────────┬───────────────────────────────┤
│ 1. Named Volumes (Docker)    │ 2. Bind Mounts (Directorio)   │
│    /var/lib/docker/volumes/  │    /home/usuario/proyecto/src │
│    - Gestionados por Docker. │    - Mapeo directo a carpetas │
│    - Rendimiento superior.   │      locales del Host.        │
│    - Recomendado Producción. │    - Recomendado Desarrollo.  │
└──────────────────────────────┴───────────────────────────────┘
```

```bash
# Crear un volumen administrado por Docker
docker volume create db_data

# Montar volumen en contenedor PostgreSQL
docker run -d -v db_data:/var/lib/postgresql/data -p 5432:5432 postgres:16
```

---

## 6. Redes en Docker

Docker administra el tráfico entre contenedores y el Host a través de varios controladores de red (*Drivers*):

- **Bridge (Por defecto)**: Crea una red privada virtual interna dentro del Host. Los contenedores pueden comunicarse por IP o por nombre de servicio si pertenecen a una red personalizada.
- **Host**: Elimina el aislamiento de red; el contenedor comparte directamente la pila de red e interfaces del Host.
- **None**: Desconecta completamente al contenedor de cualquier interfaz de red (Máximo aislamiento).

```bash
# Crear una red bridge personalizada
docker network create red_aplicacion

# Conectar contenedores a la misma red para permitir resolución DNS interna por nombre
docker run -d --name mi_bd --network red_aplicacion postgres:16
docker run -d --name mi_api --network red_aplicacion -p 3000:3000 mi-api
```

---

## 7. Orquestación local con Docker Compose

**Docker Compose** permite declarar y coordinar múltiples contenedores interconectados mediante un archivo YAML denominado `docker-compose.yml`.

### Ejemplo completo multi-servicio (Backend + BD PostgreSQL)

```yaml
version: '3.8'

services:
  backend_api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
      - DB_HOST=postgres_db
      - DB_USER=camejo_user
      - DB_PASS=secretpass
      - DB_NAME=camejo_db
    depends_on:
      - postgres_db
    networks:
      - app_network

  postgres_db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: camejo_user
      POSTGRES_PASSWORD: secretpass
      POSTGRES_DB: camejo_db
    ports:
      - "5433:5432"  # Mapea el puerto 5433 del host al 5432 del contenedor
    volumes:
      - pg_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Ejecuta DDL inicial la primera vez
    networks:
      - app_network

volumes:
  pg_data:

networks:
  app_network:
    driver: bridge
```

### Comandos de operación con Docker Compose
```bash
# Iniciar todos los servicios en segundo plano compilando imágenes si es necesario
docker compose up -d --build

# Ver logs unificados de todos los servicios en tiempo real
docker compose logs -f

# Detener todos los servicios preservando volúmenes
docker compose down

# Detener servicios y ELIMINAR volúmenes para forzar un reinicio desde cero
docker compose down -v
```
