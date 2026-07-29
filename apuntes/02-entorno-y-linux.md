# Módulo 02: Entorno de desarrollo, Linux y filesystem

Guía de estudio completa para dominar el sistema operativo **Linux**, la estructura del sistema de archivos (FHS), la gestión de permisos POSIX y la línea de comandos (CLI).

---

## 1. Arquitectura de sistemas operativos UNIX / Linux

Linux es un sistema operativo multitarea y multiusuario derivado de la arquitectura de diseño UNIX.

```
┌─────────────────────────────────────────────────────────────┐
│               USUARIO / APLICACIONES                        │
│          (Navegador, VS Code, Scripts, CLI)                 │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                       SHELL (CLI)                           │
│               (Bash, Zsh, Sh - Interprete)                  │
└──────────────────────────────┬──────────────────────────────┘
                               │ Llamadas al sistema (Syscalls)
┌──────────────────────────────▼──────────────────────────────┐
│                      KERNEL LINUX                           │
│  - Gestión de procesos       - Gestión de memoria           │
│  - Controladores (Drivers)   - Sistema de archivos (VFS)    │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                        HARDWARE                             │
│               (CPU, RAM, Disco, Red, GPU)                   │
└─────────────────────────────────────────────────────────────┘
```

### Conceptos clave
- **Kernel (Núcleo)**: Es el componente central del sistema operativo. Se ejecuta en modo privilegiado (*Ring 0*) y administra directamente el hardware del equipo.
- **System Calls (Syscalls)**: Interfaz que permite a las aplicaciones en espacio de usuario solicitar servicios al Kernel (ej. abrir un archivo `open()`, crear un proceso `fork()`, enviar datos por red `send()`).
- **Shell**: Programa que actúa como interfaz entre el usuario y el sistema operativo. Interpreta los comandos tecleados en la terminal.

---

## 2. Jerarquía del sistema de archivos (FHS)

En UNIX / Linux rige la máxima: **"Todo es un archivo"** (*Everything is a file*). Los discos, dispositivos, procesos y conectores de red se representan como entradas dentro de la jerarquía de archivos.

La estructura responde al estándar **Filesystem Hierarchy Standard (FHS)** y parte de un único directorio raíz denominado `/`:

```
/ (Raíz)
├── bin    ── Ejecutables binarios esenciales para todos los usuarios (ls, cp, cat, bash)
├── boot   ── Archivos necesarios para el arranque (Kernel Linux, GRUB, initramfs)
├── dev    ── Archivos de dispositivos físicos y virtuales (/dev/sda, /dev/null, /dev/urandom)
├── etc    ── Archivos de configuración global del sistema y servicios instalados
├── home   ── Carpetas personales de los usuarios del sistema (/home/usuario)
├── lib    ── Librerías compartidas necesarias para los binarios de /bin y /sbin
├── media  ── Puntos de montaje automáticos para discos extraíbles (USB, CD-ROM)
├── mnt    ── Punto de montaje manual temporal para sistemas de archivos externos
├── opt    ── Paquetes de software opcionales de terceros (ej. Chrome, Zoom)
├── proc   ── Sistema de archivos virtual que expone información del Kernel y procesos
├── root   ── Directorio personal del usuario administrador (root)
├── sbin   ── Binarios de administración del sistema exclusivos para root (fdisk, reboot)
├── tmp    ── Archivos temporales (el sistema suele vaciarlo en cada reinicio)
├── usr    ── Programas y archivos de lectura de usuario (/usr/bin, /usr/lib, /usr/local)
└── var    ── Archivos de datos variables que cambian constantemente (logs en /var/log, mail)
```

---

## 3. Sistema de permisos POSIX y propiedad

Cada archivo y directorio en Linux pertenece a un **Usuario propietario** y a un **Grupo propietario**.

### Representación de permisos

Al ejecutar `ls -l`, cada entrada muestra un bloque de 10 caracteres como el siguiente:

```
- r w x r - x r - -
│ └──┬──┘ └──┬──┘ └──┬──┘
│    │      │      └── Otros usuarios (o - Others)
│    │      └───────── Grupo propietario (g - Group)
│    └──────────────── Usuario propietario (u - User)
└───────────────────── Tipo de recurso (- = archivo, d = directorio, l = enlace)
```

### Tipos de permisos

| Permiso | Valor Octal | En Archivos | En Directorios |
| :---: | :---: | :--- | :--- |
| **`r` (Read)** | `4` | Leer el contenido del archivo. | Listar los archivos dentro de la carpeta (`ls`). |
| **`w` (Write)** | `2` | Modificar/guardar el archivo. | Crear, renombrar o borrar archivos dentro del directorio. |
| **`x` (Execute)** | `1` | Ejecutar el archivo (script/binario). | Acceder o entrar a la carpeta (`cd`) y consultar sus metadatos. |

### Cálculo del modo octal

Los permisos se agrupan en tríos sumando los valores individuales ($r=4, w=2, x=1$):

- `rwx` $= 4 + 2 + 1 = 7$ (Todos los permisos)
- `r-x` $= 4 + 0 + 1 = 5$ (Lectura y ejecución)
- `r--` $= 4 + 0 + 0 = 4$ (Solo lectura)

Ejemplo: `chmod 754 script.sh` asigna `rwx` al dueño, `r-x` al grupo y `r--` a otros.

### Comandos de modificación de permisos y propiedad

```bash
# Otorgar permisos de ejecución al dueño de un script
chmod u+x script.sh

# Asignar lectura y escritura al dueño, y lectura al resto (modo octal)
chmod 644 config.json

# Cambiar el propietario de un archivo
chown silva reportes.txt

# Cambiar el propietario y grupo de una carpeta y todo su contenido (recursivo)
chown -R silva:desarrolladores /var/www/proyecto
```

---

## 4. Comandos esenciales de la línea de comandos (CLI)

### Navegación y estructura
```bash
pwd                   # Muestra la ruta absoluta del directorio actual
ls -la                # Lista todos los archivos (incluidos ocultos) en formato detallado
cd /var/log           # Cambia de directorio a /var/log
cd ..                 # Sube un nivel en la jerarquía de directorios
cd ~                  # Navega al directorio personal (/home/usuario)
```

### Manipulación de archivos
```bash
mkdir -p src/utils         # Crea la carpeta src y la subcarpeta utils simultáneamente
touch app.py               # Crea un archivo vacío o actualiza la fecha de modificación
cp archivo.txt copia.txt   # Copia un archivo
cp -r carpeta1/ carpeta2/  # Copia una carpeta y todo su contenido recursivamente
mv archivo.txt /tmp/       # Mueve o renombra un archivo
rm archivo.txt             # Elimina un archivo
rm -rf carpeta/            # Elimina una carpeta y su contenido de forma recursiva y forzada
```

### Lectura y procesamiento de texto
```bash
cat archivo.txt          # Muestra todo el contenido del archivo en pantalla
less archivo.log         # Permite navegar y desplazar el contenido del archivo interactivamente
head -n 10 archivo.txt   # Muestra las primeras 10 líneas
tail -n 20 archivo.txt   # Muestra las últimas 20 líneas
tail -f /var/log/syslog  # Muestra las últimas líneas y sigue transmitiendo cambios en tiempo real
grep -rn "ERROR" ./src   # Busca la palabra "ERROR" recursivamente (-r) con número de línea (-n)
find . -name "*.py"      # Busca archivos con extensión .py en el directorio actual
wc -l archivo.txt        # Cuenta la cantidad de líneas de un archivo
```

### Administración de procesos y recursos
```bash
ps aux                # Muestra todos los procesos en ejecución en el sistema
top / htop            # Monitor interactivo de consumo de CPU, RAM y procesos
kill 1234             # Envía una señal de terminación suave (SIGTERM) al proceso PID 1234
kill -9 1234          # Envía una señal de terminación forzada e inmediata (SIGKILL)
df -h                 # Muestra el espacio libre y usado en los discos en formato legible
du -sh carpeta/       # Muestra el tamaño total consumido por una carpeta
```

---

## 5. Editores de texto en consola: Guía práctica de Vim

Vim es un editor de texto modal de consola rápido e indispensable para la administración de servidores Linux.

### Los modos de Vim

1. **Modo normal** (Por defecto al abrir Vim o al presionar `Esc`): Permite navegar por el texto, copiar, pegar y borrar líneas.
2. **Modo inserción** (Se activa presionando `i`): Permite escribir texto en el documento.
3. **Modo comando** (Se activa presionando `:` desde el Modo normal): Permite ejecutar comandos de guardado, búsqueda y salida.

### Comandos de supervivencia en Vim

| Acción | Comando (desde Modo normal) |
| :--- | :--- |
| **Entrar a escribir** | Presionar `i` (Insert antes del cursor) o `a` (Append después del cursor). |
| **Volver al Modo normal** | Presionar la tecla `Esc`. |
| **Guardar cambios** | Teclear `:w` y presionar `Enter`. |
| **Guardar y salir** | Teclear `:wq` o `ZZ` y presionar `Enter`. |
| **Salir sin guardar (forzado)** | Teclear `:q!` y presionar `Enter`. |
| **Eliminar línea actual** | Teclear `dd`. |
| **Copiar línea actual** | Teclear `yy`. |
| **Pegar debajo** | Teclear `p`. |
| **Buscar texto** | Teclear `/texto_a_buscar` y presionar `Enter` (navegar con `n`). |
