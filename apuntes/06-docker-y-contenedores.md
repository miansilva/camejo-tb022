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

## 3.1. Flujo de trabajo práctico y evolución didáctica: De 0 a Docker Compose

Para comprender la verdadera razón de ser de cada herramienta dentro de Docker sin asumir conceptos previa o arbitrariamente, resulta fundamental analizar la evolución progresiva del flujo de trabajo:

```
┌──────────────────────────────────────────────────────────────┐
│  1. CONTENEDOR MANUAL (1 APP)                                │
│     docker run + apt/npm a mano dentro del contenedor        │
└──────────────────────────────────────────────────────────────┘
                               │
                ¿Cómo automatizo la imagen?
                               ▼
┌──────────────────────────────────────────────────────────────┐
│  2. IMAGEN CON DOCKERFILE (1 APP)                            │
│     Receta inmutable, portable y reusable                    │
└──────────────────────────────────────────────────────────────┘
                               │
               ¿Cómo conecto la BD y Red a mano?
                               ▼
┌──────────────────────────────────────────────────────────────┐
│  3. MULTICONTENEDOR A MANO (APP + BD)                        │
│     Crear red, volumen y conectar contenedores a mano        │
└──────────────────────────────────────────────────────────────┘
                               │
             ¿Cómo evito toda la config manual?
                               ▼
┌──────────────────────────────────────────────────────────────┐
│  4. ORQUESTACIÓN CON DOCKER COMPOSE                          │
│     Todo declarado y automatizado en 1 manifiesto YAML       │
└──────────────────────────────────────────────────────────────┘
```

---

### Paso 1: Configurar 1 contenedor a mano (Comandos puros)

* **Concepto:** Arrancamos un contenedor básico limpio y entramos a la consola para instalar todo manualmente.
* **Comandos:**
  ```bash
  # 1. Crear y entrar a un contenedor de Ubuntu en modo interactivo
  docker run -it -p 3000:3000 --name app_manual ubuntu bash

  # 2. Adentro del contenedor (root@id:/#), instalamos Python y creamos la app a mano:
  apt update && apt install -y python3
  echo "print('Hola mundo desde contenedor manual')" > app.py
  python3 app.py
  ```
* **El problema:** Los contenedores son **efímeros**. Si borrables el contenedor (`docker rm -f app_manual`), **se pierde todo** lo instalado adentro.

---

### Paso 2: Automatizar la imagen con `Dockerfile` (La Receta)

* **Concepto:** En lugar de instalar a mano en la consola, guardamos las instrucciones en un archivo `Dockerfile`.
* **Archivo `Dockerfile`:**
  ```dockerfile
  # 1. Partimos de una imagen limpia con Python listo
  FROM python:3.10-slim

  # 2. Definimos la carpeta de trabajo
  WORKDIR /app

  # 3. Copiamos nuestro archivo al contenedor
  COPY app.py .

  # 4. Ejecutamos la app
  CMD ["python", "app.py"]
  ```
* **Comandos:**
  ```bash
  # 1. Crear la imagen reusable
  docker build -t mi-app-python .

  # 2. Correr el contenedor desde la imagen
  docker run mi-app-python
  ```
* **La mejora:** La imagen es **reusable**. Cualquier persona del equipo puede construir el mismo entorno.
* **El nuevo problema:** Nuestra aplicación ahora necesita conectarse a una **Base de Datos (Redis)**. ¿Cómo las conectamos a mano usando solo comandos de Docker?

---

### Paso 3: Conectar múltiples contenedores A MANO (El dolor de Redes y Volúmenes)

Si queremos conectar nuestra App con una Base de Datos **sin usar Docker Compose**, estamos obligados a hacer **TODO manualmente desde la terminal**:

1. **Crear la red virtual a mano:**
   ```bash
   # Docker crea una red bridge aislada
   docker network create mi-red
   ```
2. **Crear el volumen de datos a mano (para que la BD no pierda información):**
   ```bash
   # Docker reserva un espacio en disco administrado
   docker volume create datos-db
   ```
3. **Levantar la Base de Datos en esa red y con ese volumen:**
   ```bash
   docker run -d --name mi-redis --network mi-red -v datos-db:/data redis:alpine
   ```
4. **Levantar nuestra App en la MISMA red para que pueda encontrar a Redis por su nombre (`mi-redis`):**
   ```bash
   docker run -d --name mi-app --network mi-red -p 3000:3000 mi-app-python
   ```

* **El gran problema:**
  * Hay que recordar y ejecutar **4 comandos complejos y largos en orden estricto**.
  * Si te olvidás de pasar el parámetro `--network mi-red`, los contenedores **no se ven entre sí**.
  * Es muy difícil de compartir con otros desarrolladores.

---

### Paso 4: Automatizar TODO con `Docker Compose`

* **Concepto:** Reemplazamos todos los comandos manuales de creación de redes, volúmenes y contenedores por un único archivo estructurado llamado `docker-compose.yml`.

* **Archivo `docker-compose.yml`:**
  ```yaml
  version: '3.8'

  services:
    # Servicio 1: Nuestra App
    app:
      build: .
      ports:
        - "3000:3000"

    # Servicio 2: Base de Datos Redis
    redis:
      image: redis:alpine
      volumes:
        - datos-redis:/data   # Guarda los datos en el volumen

  volumes:
    datos-redis:              # Compose crea este volumen automáticamente
  ```

* **El beneficio definitivo:**
  * **No tenés que crear redes a mano:** Docker Compose crea una red compartida automáticamente para todos los servicios del archivo.
  * **No tenés que crear volúmenes a mano:** Compose crea los volúmenes declarados.
  * **Un solo comando para todo:**
    ```bash
    # Levanta la App, Redis, crea la red interna y los volúmenes automáticos:
    docker compose up -d

    # Apaga y limpia todo el sistema:
    docker compose down
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

---

## 8. Cheatsheet y Referencia de Comandos CLI (Progresivo y Agrupado)

Esta guía rápida organiza los comandos indispensables de Docker en un flujo lógico y progresivo, desde la gestión de imágenes hasta la limpieza del sistema.

### 8.1. Gestión de Imágenes (`docker image / docker ...`)

Las imágenes son las plantillas de solo lectura.

| Comando | Descripción |
| :--- | :--- |
| `docker pull <imagen>:<tag>` | Descarga una imagen desde un registry (ej. Docker Hub). |
| `docker images` *(o `docker image ls`)* | Lista todas las imágenes almacenadas localmente en el Host. |
| `docker build -t <nombre>:<tag> .` | Construye una imagen a partir del `Dockerfile` en el directorio actual. |
| `docker rmi <imagen>` *(o `docker image rm`)* | Elimina una imagen local (si no hay contenedores usándola). |
| `docker image inspect <imagen>` | Muestra metadatos detallados en JSON (capas, variables, entrypoint). |

### 8.2. Gestión de Contenedores (`docker container / docker ...`)

Los contenedores son las instancias en ejecución de las imágenes.

> **Diferencia clave:**  
> - `docker create`: Prepara el contenedor y su capa de escritura pero **no lo inicia**.  
> - `docker start`: Arranca un contenedor ya creado o detenido.  
> - `docker run`: Ejecuta un `create` y un `start` en un único paso.

```bash
# 1. Crear e iniciar un contenedor en segundo plano (Detached) con puerto y nombre
docker run -d --name mi-mongo -p 27017:27017 mongo

# 2. Crear e iniciar un contenedor interactivo con consola abierta
docker run -it --name mi-ubuntu ubuntu bash

# 3. Listar contenedores activos (en ejecución)
docker ps

# 4. Listar TODOS los contenedores (activos, detenidos y con error)
docker ps -a

# 5. Detener un contenedor en ejecución (envía SIGTERM)
docker stop mi-mongo

# 6. Iniciar un contenedor detenido
docker start mi-mongo

# 7. Reiniciar un contenedor
docker restart mi-mongo

# 8. Ver logs de un contenedor
docker logs mi-mongo

# 9. Ver logs en tiempo real (Follow)
docker logs -f mi-mongo

# 10. ENTRAR a la consola de un contenedor en ejecución (Crucial para debugging)
docker exec -it mi-mongo bash

# 11. Eliminar un contenedor detenido
docker rm mi-mongo

# 12. Forzar la detención y eliminación de un contenedor activo
docker rm -f mi-mongo
```

### 8.3. Gestión de Redes (`docker network`)

Permiten la comunicación entre contenedores aislados.

| Comando | Descripción |
| :--- | :--- |
| `docker network ls` | Lista las redes disponibles en el Engine (bridge, host, custom). |
| `docker network create <nombre>` | Crea una red bridge personalizada con DNS automático. |
| `docker network inspect <nombre>` | Muestra los contenedores conectados a esa red y sus IPs. |
| `docker network connect <red> <contenedor>` | Conecta un contenedor en ejecución a una red existente. |
| `docker network rm <nombre>` | Elimina una red personalizada. |

### 8.4. Gestión de Volúmenes (`docker volume`)

Garantizan la persistencia de datos fuera de la vida útil del contenedor.

| Comando | Descripción |
| :--- | :--- |
| `docker volume ls` | Lista los volúmenes administrados por Docker. |
| `docker volume create <nombre>` | Crea un nuevo volumen persistente. |
| `docker volume inspect <nombre>` | Muestra la ruta física del volumen en el disco del Host. |
| `docker volume rm <nombre>` | Elimina un volumen (debe estar desconectado). |

### 8.5. Orquestación con Docker Compose (`docker compose`)

Administración del ciclo de vida multi-contenedor desde `docker-compose.yml`.

```bash
# Levantar todos los servicios en segundo plano (reconstruyendo si hubo cambios)
docker compose up -d --build

# Ver el estado de los servicios definidos en el Compose
docker compose ps

# Ver logs unificados de todos los servicios en tiempo real
docker compose logs -f <nombre_servicio>

# Ejecutar un comando dentro de un servicio del Compose
docker compose exec app sh

# Apagar los servicios y remover la red compartida
docker compose down

# Apagar servicios y ELIMINAR volúmenes asignados (reinicio limpio completo)
docker compose down -v
```

### 8.6. Monitoreo, Diagnóstico y Limpieza (`docker system`)

```bash
# Ver consumo de CPU, Memoria e I/O en tiempo real de los contenedores
docker stats

# Ver espacio en disco utilizado por objetos de Docker
docker system df

# LIMPIEZA TOTAL: Elimina contenedores detenidos, redes no usadas e imágenes huérfanas
docker system prune -f

# Limpieza profunda incluyendo imágenes no utilizadas
docker system prune -a --volumes
```

