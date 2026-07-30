# Módulo 02: Entorno de desarrollo, Linux, filesystem y editores

Guía de estudio conceptual y técnica completa sobre el sistema operativo **Linux**, la estructura jerárquica del sistema de archivos (FHS), la administración de permisos POSIX, gestión de procesos, comandos CLI y editores de texto de consola.

---

## 1. Arquitectura del sistema operativo UNIX / Linux

Linux es un sistema operativo multitarea, multiusuario y de código abierto basado en la filosofía de diseño UNIX. Su arquitectura se divide en capas bien delimitadas:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     ESPACIO DE USUARIO (User Space)                      │
│    Aplicaciones (VS Code, Navegadores, Servidores) + Shell (Bash/Zsh)    │
└────────────────────────────────────┬─────────────────────────────────────┘
                                     │ Llamadas al sistema (System Calls)
                                     ▼
┌────────────────────────────────────┴─────────────────────────────────────┐
│                     ESPACIO DEL KERNEL (Kernel Space)                    │
│  - Gestión de procesos (Scheduler)     - Administración de memoria (RAM) │
│  - Controladores de hardware (Drivers) - Sistema de archivos (VFS)       │
│  - Subsistema de red (TCP/IP, Sockets) - Seguridad y permisos POSIX      │
└────────────────────────────────────┬─────────────────────────────────────┘
                                     │
                                     ▼
┌────────────────────────────────────┴─────────────────────────────────────┐
│                                HARDWARE                                  │
│                   CPU, Memoria RAM, Discos, Red, GPU                     │
└──────────────────────────────────────────────────────────────────────────┘
```

### Componentes fundamentales
- **Kernel (Núcleo)**: Componente principal que se ejecuta en el nivel más privileged (*Ring 0* del procesador). Gestiona directamente los recursos físicos del hardware y asigna tiempos de procesador y memoria a las aplicaciones.
- **System Calls (Syscalls)**: Interfaz programática que expone el Kernel para que las aplicaciones del espacio de usuario soliciten servicios críticos (ej. `open()`, `read()`, `write()`, `fork()`, `execve()`, `socket()`).
- **Shell (Consola / Intérprete)**: Interfaz en línea de comandos (CLI) que lee, interpreta y ejecuta las instrucciones tecleadas por el usuario.

---

## 2. Jerarquía del sistema de archivos (estándar FHS)

En los sistemas UNIX rige el principio fundamental: **"Todo es un archivo"** (*Everything is a file*). Dispositivos de hardware, sockets de red, tuberías de datos y procesos en ejecución se exponen como nodos o archivos dentro del árbol de directorios.

El estándar **Filesystem Hierarchy Standard (FHS)** organiza la estructura a partir del directorio raíz (`/`):

```
/ (Directorio raíz)
├── bin    ── Binarios y comandos esenciales del sistema requeridos por todos los usuarios (ls, cp, cat, bash).
├── boot   ── Archivos necesarios para el arranque del sistema (Kernel Linux, GRUB, initramfs).
├── dev    ── Archivos de dispositivos físicos y virtuales (/dev/sda, /dev/null, /dev/urandom).
├── etc    ── Archivos de configuración global del sistema operativo y aplicaciones instaladas.
├── home   ── Carpetas personales de los usuarios no administradores (/home/usuario).
├── lib    ── Librerías compartidas necesarias para ejecutar los binarios esenciales de /bin y /sbin.
├── media  ── Puntos de montaje automáticos para medios extraíbles (discos USB, unidades ópticas).
├── mnt    ── Punto de montaje temporal para sistemas de archivos o unidades montadas manualmente.
├── opt    ── Paquetes de software opcionales instalados por aplicaciones de terceros.
├── proc   ── Sistema de archivos virtual (en RAM) que expone el estado del Kernel y procesos en ejecución.
├── root   ── Directorio personal exclusivo del usuario administrador del sistema (root).
├── sbin   ── Binarios de administración del sistema reservados para el superusuario (fdisk, iptables, reboot).
├── sys    ── Sistema de archivos virtual que expone parámetros del hardware y controladores del Kernel.
├── tmp    ── Archivos temporales del sistema y usuarios (suele borrarse en cada reinicio).
├── usr    ── Jerarquía secundaria para programas y datos de lectura compartidos (/usr/bin, /usr/lib, /usr/local).
└── var    ── Archivos con datos variables y dinámicos (archivos de log en /var/log, colas de correo, bases de datos).
```

### Directorios virtuales clave (`/proc` y `/sys`)
- **/proc**: No consume espacio en disco físico. Al consultar `/proc/cpuinfo` o `/proc/meminfo`, el Kernel genera en tiempo real los datos del procesador y uso de memoria. Cada proceso activo posee una carpeta con su número de identificación PID (ej. `/proc/1234/`).
- **/dev/null**: El "agujero negro" de Linux. Todo dato enviado a `/dev/null` se descarta inmediatamente.
- **/dev/urandom**: Generador de números aleatorios criptográficamente seguros mantenido por el Kernel.

---

## 3. Modelo de permisos POSIX, propiedad y modos octales

Cada recurso en el sistema de archivos posee un **Usuario propietario (User/Owner)**, un **Grupo propietario (Group)** y un conjunto de **Permisos de acceso**.

### Estructura de visualización de permisos
Al ejecutar `ls -la`, el primer bloque de 10 caracteres define el tipo de archivo y los permisos:

```
- r w x r - x r - -   1   usuario   devs   4096   Jul 30 10:00   script.sh
│ └──┬──┘ └──┬──┘ └──┬──┘
│    │       │       └── Otros usuarios del sistema (o - Others)
│    │       └───────── Grupo propietario (g - Group)
│    └──────────────── Usuario propietario (u - User)
└───────────────────── Tipo de recurso (- = archivo normal, d = directorio, l = enlace simbólico)
```

### Tabla de permisos POSIX

| Permiso | Simbólico | Octal | Significado en archivos | Significado en directorios |
| :--- | :---: | :---: | :--- | :--- |
| **Lectura** | `r` | `4` | Permite leer o copiar el contenido del archivo. | Permite listar los archivos dentro del directorio (`ls`). |
| **Escritura** | `w` | `2` | Permite modificar o sobreescribir el contenido. | Permite crear, renombrar o borrar archivos en el directorio. |
| **Ejecución**| `x` | `1` | Permite ejecutar el archivo (scripts bash, binarios). | Permite acceder/entrar al directorio (`cd`) y acceder a metadatos. |

### Cálculo del modo octal
Los permisos se calculan sumando los valores numéricos para cada uno de los tres grupos (User, Group, Others):

- `rwx` $= 4 + 2 + 1 = 7$ (Acceso total)
- `rw-` $= 4 + 2 + 0 = 6$ (Lectura y escritura)
- `r-x` $= 4 + 0 + 1 = 5$ (Lectura y ejecución)
- `r--` $= 4 + 0 + 0 = 4$ (Solo lectura)

```bash
# Ejemplo: Asignar rwx al dueño (7), r-x al grupo (5) y r-- a otros (4)
chmod 754 script.sh
```

### Máscara por defecto (`umask`)
La orden `umask` define los permisos por defecto que se restringen al crear nuevos archivos o carpetas:
- Permisos máximos por defecto para carpetas: `777`.
- Permisos máximos por defecto para archivos: `666`.
- Si el `umask` es `022`, una nueva carpeta tendrá permisos `777 - 022 = 755` (`rwxr-xr-x`) y un nuevo archivo tendrá `666 - 022 = 644` (`rw-r--r--`).

### Permisos especiales (SUID, SGID y Sticky Bit)
1. **SUID (Set User ID)**: Si se aplica a un ejecutable, cualquier usuario que lo ejecute lo hará con los privilegios del *dueño* del archivo (ej. `/usr/bin/passwd` permite modificar contraseñas del sistema).
2. **SGID (Set Group ID)**: Si se aplica a un directorio, los archivos creados dentro heredarán automáticamente el grupo propietario del directorio padre.
3. **Sticky Bit**: Aplicado a carpetas compartidas (como `/tmp`), garantiza que únicamente el dueño de un archivo pueda borrarlo o renombrarlo, previniendo que usuarios impidan el trabajo de otros.

---

## 4. Referencia de comandos CLI y administración de procesos

### Gestión de archivos y navegación
```bash
# Consultar ruta absoluta actual
pwd

# Listar archivos detallados incluyendo ocultos (.)
ls -la

# Creación de estructura de directorios padre e hijos
mkdir -p proyecto/src/controllers

# Copia recursiva de carpetas
cp -r carpeta_origen/ carpeta_destino/

# Mover o renombrar archivos
mv archivo_viejo.txt /tmp/archivo_nuevo.txt

# Eliminación recursiva y forzada
rm -rf carpeta_obsoleta/
```

### Filtros, inspección de texto y procesamiento
```bash
# Visualizar archivo completo
cat archivo.txt

# Paginador interactivo para archivos extensos
less /var/log/syslog

# Inspección de encabezado y pie de página
head -n 15 archivo.txt
tail -n 20 archivo.txt

# Monitorear logs en tiempo real
tail -f /var/log/nginx/access.log

# Búsqueda de patrones en archivos (Búsqueda recursiva con número de línea)
grep -rn "EXCEPTION" ./src/

# Buscar archivos en el sistema por nombre o tipo
find . -type f -name "*.py"

# Conteo de líneas, palabras y caracteres
wc -l archivo.txt
```

### Gestión de procesos y señales del sistema

Un **proceso** es una instancia de un programa en ejecución con un espacio de memoria asignado por el Kernel.

```bash
# Listar todos los procesos activos con detalle
ps aux

# Monitor interactivo de consumo de CPU, RAM y procesos en tiempo real
top

# Consultar el espacio consumido en disco en formato legible (Human Readable)
df -h

# Consultar el tamaño consumido por una carpeta específica
du -sh /var/log/
```

#### Señales del Kernel a procesos (`kill`)
- **`SIGINT` (Señal 2)**: Interrupción enviada desde teclado (`Ctrl + C`).
- **`SIGTERM` (Señal 15)**: Solicitud de terminación ordenada (permite al proceso cerrar archivos y conexiones).
- **`SIGKILL` (Señal 9)**: Terminación forzada e inmediata ejecutada directamente por el Kernel.

```bash
# Enviar señal de terminación limpia
kill 1234

# Forzar la terminación inmediata e incondicional del proceso PID 1234
kill -9 1234
```

#### Procesos en segundo plano (*Background*)
- Para ejecutar un comando en segundo plano, agregar `&` al final: `python3 server.py &`
- `jobs`: Muestra los trabajos en segundo plano gestionados por la sesión actual de la terminal.
- `fg %1`: Mueve el trabajo 1 al primer plano (*foreground*).
- `bg %1`: Reanuda la ejecución de un trabajo pausado en segundo plano (*background*).

### Atajos de teclado esenciales de la terminal
- **`Ctrl + C`**: Interrumpe y cancela el proceso actual (`SIGINT`).
- **`Ctrl + Z`**: Pausa el proceso actual y lo envía a segundo plano (`SIGTSTP`).
- **`Ctrl + L`**: Limpia la pantalla de la terminal (Equivalente al comando `clear`).
- **`Ctrl + A` / `Ctrl + E`**: Mueve el cursor al inicio (`A`) o al final (`E`) de la línea de comandos.
- **`Ctrl + R`**: Búsqueda interactiva en el historial de comandos ejecutados previamente.

---

## 5. Editores de texto en consola: Nano y Vim

### Editor Nano (Sencillez operativa)
Nano es un editor directo sin modos, adecuado para modificaciones rápidas:
- **`Ctrl + O`**: Guardar (*WriteOut*) los cambios en disco.
- **`Ctrl + X`**: Salir del editor Nano.
- **`Ctrl + K`**: Cortar la línea de texto actual.
- **`Ctrl + U`**: Pegar la línea cortada.
- **`Ctrl + W`**: Buscar un texto en el documento.

### Editor Vim (Editor modal avanzado)

```
┌─────────────────────────────────────────────────────────────┐
│                         MODO NORMAL                         │
│          (Navegar, borrar, copiar, pegar comandos)          │
└──────────────┬──────────────────────────────▲───────────────┘
               │ Presionar 'i', 'a' o 'o'     │ Presionar 'Esc'
               ▼                              │
┌──────────────┴──────────────┐   ┌───────────┴────────────────┐
│        MODO INSERCIÓN       │   │        MODO COMANDO        │
│   (Escribir texto directo)  │   │   (Ejecutar :w, :q, :wq)   │
└─────────────────────────────┘   └────────────────────────────┘
```

#### Cheatsheet de operación en Vim

| Categoría | Comando | Descripción |
| :--- | :---: | :--- |
| **Navegación** | `h`, `j`, `k`, `l` | Mover cursor a la Izquierda (`h`), Abajo (`j`), Arriba (`k`), Derecha (`l`). |
| **Navegación** | `w` / `b` | Avanzar al inicio de la siguiente palabra (`w`) o retroceder (`b`). |
| **Navegación** | `0` / `$` | Ir al inicio de la línea (`0`) o al final de la línea (`$`). |
| **Navegación** | `gg` / `G` | Ir al inicio del archivo (`gg`) o a la última línea (`G`). |
| **Edición** | `i` / `a` | Entrar a Modo Inserción antes (`i`) o después (`a`) del cursor. |
| **Edición** | `o` | Abrir una nueva línea debajo y entrar a Modo Inserción. |
| **Edición** | `dd` | Cortar / Eliminar la línea actual. |
| **Edición** | `yy` | Copiar (*yank*) la línea actual. |
| **Edición** | `p` | Pegar (*paste*) el texto copiado debajo del cursor. |
| **Edición** | `u` / `Ctrl + r` | Deshacer última acción (`u`) o rehacer (`Ctrl + r`). |
| **Comandos `:`** | `:w` | Guardar cambios en disco (*write*). |
| **Comandos `:`** | `:q` | Salir de Vim (*quit*). |
| **Comandos `:`** | `:wq` / `ZZ` | Guardar cambios y salir. |
| **Comandos `:`** | `:q!` | Salir forzadamente sin guardar ningún cambio realizado. |
| **Búsqueda** | `/patron` | Buscar `patron` en el texto (presionar `n` para siguiente coincidencia). |
