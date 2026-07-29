# Módulo 06: Contenedores y arquitectura Docker

Guía de estudio completa para dominar la virtualización basada en **contenedores con Docker**, la optimización de **`Dockerfile`**, la persistencia de datos y la orquestación con **Docker Compose**.

---

## 1. Virtualización tradicional vs contenedores

### Máquinas virtuales (VMs)
Una máquina virtual emula un sistema informático completo sobre un **Hypervisor** (Hypervisor Tipo 1 como ESXi/Proxmox o Tipo 2 como VirtualBox).
- **Desventajas**: Cada VM incluye un sistema operativo completo (*Guest OS*), consumiendo gigabytes de memoria RAM, espacio en disco duro y minutos de tiempo de arranque.

### Contenedores (Docker)
Los contenedores **comparten el mismo Kernel del sistema operativo Host**. No ejecutan un SO propio, lo que les permite arrancar en milisegundos y consumir recursos mínimos de RAM y CPU.

```
┌─────────────────────────────────────────┐    ┌─────────────────────────────────────────┐
│         VIRTUALIZACIÓN (VMs)            │    │        CONTENEDORES (DOCKER)            │
├─────────────┬─────────────┬─────────────┤    ├─────────────┬─────────────┬─────────────┤
│    App A    │    App B    │    App C    │    │    App A    │    App B    │    App C    │
├─────────────┼─────────────┼─────────────┤    ├─────────────┼─────────────┼─────────────┤
│  Libs/Bin   │  Libs/Bin   │  Libs/Bin   │    │  Libs/Bin   │  Libs/Bin   │  Libs/Bin   │
├─────────────┼─────────────┼─────────────┤    ├─────────────┴─────────────┴─────────────┤
│  Guest OS   │  Guest OS   │  Guest OS   │    │             DOCKER ENGINE               │
├─────────────┴─────────────┴─────────────┤    ├─────────────────────────────────────────┤
│               HYPERVISOR                │    │           SISTEMA OPERATIVO HOST        │
├─────────────────────────────────────────┤    ├─────────────────────────────────────────┤
│            HARDWARE FÍSICO              │    │            HARDWARE FÍSICO              │
└─────────────────────────────────────────┘    └─────────────────────────────────────────┘
```

### Mecanismos internos del Kernel Linux que posibilitan Docker

1. **Linux Namespaces (Aislamiento)**: Aíslan lo que el proceso dentro del contenedor puede *ver*:
   - `pid`: Aísla el árbol de procesos (el proceso principal del contenedor se ve a sí mismo como PID 1).
   - `net`: Aísla las interfaces de red y puertos.
   - `mnt`: Aísla los puntos de montaje del sistema de archivos.
   - `ipc`, `uts`, `user`: Aísla memoria compartida, hostname y usuarios.
2. **Control Groups - cgroups (Limitación)**: Limitan y miden los recursos que el contenedor puede *consumir* (máximo de memoria RAM, cuota de CPU, I/O de disco).

---

## 2. Componentes de la arquitectura Docker

- **Docker Client (`docker`)**: Herramienta CLI de terminal con la que interactúa el desarrollador.
- **Docker Daemon (`dockerd`)**: Servicio persistente en segundo plano que gestiona las imágenes, contenedores, redes y volúmenes a través de una API REST.
- **Docker Registry (ej. Docker Hub)**: Repositorio centralizado público o privado para almacenar y distribuir imágenes de Docker.

---

## 3. Imágenes vs contenedores

- **Imagen Docker**: Plantilla de solo lectura (*Read-Only*) compuesta por múltiples capas superpuestas mediante un sistema de archivos de unión (*Union File System* / Overlay2). Contiene el código, dependencias, librerías y configuración base.
- **Contenedor Docker**: Instancia ejecutable en tiempo de real de una imagen. Agrega una capa delgada de lectura y escritura (*Read-Write Container Layer*) en la parte superior.

---

## 4. Guía detallada de `Dockerfile`

Un `Dockerfile` es un script de texto que contiene las instrucciones secuenciales para construir una imagen Docker.

### Instrucciones principales

| Instrucción | Propósito | Ejemplo |
| :--- | :--- | :--- |
| **`FROM`** | Define la imagen base sobre la cual construir. | `FROM node:20-alpine` |
| **`WORKDIR`** | Establce el directorio de trabajo dentro del contenedor. | `WORKDIR /app` |
| **`COPY`** | Copia archivos/carpetas desde el Host hacia el contenedor. | `COPY package*.json ./` |
| **`RUN`** | Ejecuta comandos durante la fase de *build* de la imagen. | `RUN npm ci --only=production` |
| **`ENV`** | Define variables de entorno dentro de la imagen. | `ENV NODE_ENV=production` |
| **`EXPOSE`** | Documenta el puerto en el que escuchará el contenedor. | `EXPOSE 3000` |
| **`USER`** | Define el usuario no-root que ejecutará la aplicación (Seguridad).| `USER node` |
| **`CMD`** | Comando por defecto al iniciar el contenedor (Sobrescribible). | `CMD ["node", "src/server.js"]` |
| **`ENTRYPOINT`** | Comando fijo ejecutable principal del contenedor. | `ENTRYPOINT ["python3"]` |

> ⚠️ **Diferencia entre `CMD` y `ENTRYPOINT`**: `CMD` provee argumentos por defecto que se pueden reemplazar fácilmente al ejecutar `docker run imagen comando_nuevo`. `ENTRYPOINT` convierte al contenedor en un ejecutable binario estricto.

### Comandos de gestión de imágenes y contenedores

```bash
# Compilar una imagen a partir del Dockerfile actual asignándole un tag
docker build -t mi-api:v1.0 .

# Listar todas las imágenes locales en el equipo
docker images

# Ejecutar un contenedor en segundo plano (-d), mapeando puerto 8080 host -> 3000 contenedor
docker run -d -p 8080:3000 --name api_service mi-api:v1.0

# Listar los contenedores actualmente en ejecución
docker ps

# Listar TODOS los contenedores (incluyendo detenidos)
docker ps -a

# Ver los logs transmitidos por un contenedor
docker logs -f api_service

# Ejecutar una terminal interactiva dentro de un contenedor en ejecución
docker exec -it api_service sh

# Acceso interactivo a psql en un contenedor de PostgreSQL
docker exec -it postgres_db psql -U postgres -d camejo_db

# Detener y eliminar un contenedor
docker stop api_service
docker rm api_service
```

---

## 5. Persistencia de datos y montajes

Los datos creados dentro de la capa *Read-Write* de un contenedor son **efímeros**: si el contenedor se borra, los datos se pierden. Para lograr persistencia existen 3 mecanismos:

```
┌─────────────────────────────────────────────────────────────┐
│                      SISTEMA DEL HOST                       │
├──────────────────────────────┬──────────────────────────────┤
│ 1. Docker Managed Volumes    │ 2. Bind Mounts               │
│    (/var/lib/docker/volumes) │    (/home/usuario/proyecto)  │
│    - Gestionado por Docker.  │    - Mapeo directo a carpeta │
│    - Ideal para Producción y │      del Host.               │
│      Bases de datos.         │    - Ideal para Desarrollo.  │
└──────────────────────────────┴──────────────────────────────┘
```

```bash
# Crear un volumen gestionado por Docker
docker volume create postgres_data

# Ejecutar base de datos usando el volumen creado
docker run -d -p 5432:5432 -v postgres_data:/var/lib/postgresql/data postgres:16
```

---

## 6. Orquestación con Docker Compose e inicialización de bases de datos

**Docker Compose** es una herramienta para definir y coordinar aplicaciones multicontenedor (ej. Backend + Base de datos + Redis) mediante un archivo declarativo `docker-compose.yml`.

### Inicialización automática de bases de datos (`/docker-entrypoint-initdb.d/`)

Las imágenes oficiales de PostgreSQL en Docker ejecutan automáticamente cualquier script SQL montado en la carpeta `/docker-entrypoint-initdb.d/` **únicamente la primera vez que el volumen de datos está vacío**.

```yaml
version: '3.8'

services:
  postgres_db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: camejo_user
      POSTGRES_PASSWORD: secretpass
      POSTGRES_DB: camejo_db
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./schema.sql:/docker-entrypoint-initdb.d/schema.sql  # Mapeo del script DDL
    ports:
      - "5433:5432"  # Se usa el puerto 5433 en el host para evitar conflictos con PostgreSQL local

volumes:
  db_data:
```

### Reglas de comportamiento y reinicio limpio

1. **Conflicto de puertos (`5433:5432`)**: Si tenés un motor PostgreSQL instalado localmente en tu PC escuchando en el puerto 5432, mapeá el puerto del host a `5433` (`"5433:5432"`) para evitar fallos de autenticación o colisiones.
2. **Reinicio completo desde cero**: Si modificás el archivo `schema.sql` y querés que se vuelva a ejecutar automáticamente, debés eliminar los volúmenes existentes:

```bash
# Detener contenedores y eliminar volúmenes para forzar la re-inicialización del DDL
docker compose down -v
docker compose up -d --build
```
