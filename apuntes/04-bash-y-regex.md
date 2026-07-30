# Módulo 04: Terminal, scripting en Bash y expresiones regulares (Regex)

Guía de estudio conceptual y técnica completa sobre la **shell Unix, interpretación de comandos, navegación del sistema de archivos, permisos, scripting en Bash, descriptores I/O, estructuras de control, expresiones regulares (Regex) y procesamiento de texto con `grep`, `sed` y `awk`**.

---

## 1. Introducción a Bash y a la terminal Unix

### 1.1. Historia y evolución: ¿por qué "Bourne Again"?

Todo comenzó en **1971** con el primer sistema operativo Unix desarrollado en los Laboratorios Bell (AT&T). Unix requería una interfaz de línea de comandos para interactuar con el Kernel, dando origen al concepto de **shell** (intérprete de comandos).

* **1977 - Bourne Shell (`sh`)**: Creado por Stephen Bourne en Bell Labs. Fue el estándar revolucionario que introdujo el scripting, variables y estructuras de control.
* **1978 - C Shell (`csh`)**: Creado por Bill Joy (BSD). Introdujo sintaxis similar al lenguaje C y el historial de comandos.
* **1983 - Korn Shell (`ksh`)**: Creado por David Korn. Combinó las mejores características de `sh` y `csh`.
* **1989 - Bash (Bourne Again Shell)**: Creado por Brian Fox para el Proyecto GNU de Richard Stallman. 

> 💡 **El juego de palabras en el nombre:**
> **"Bourne Again Shell"** es un doble sentido en inglés:
> 1. **Literal:** Es la shell Bourne "otra vez" (*Bourne Again*), compatible de forma retroactiva con `sh`.
> 2. **Juego de palabras:** Suena exactamente como *"Born Again Shell"* (Shell renacida).

```
Línea de tiempo histórica:

1971       1977         1978        1983       1989       1996       2019
  │          │            │           │          │          │          │
Unix ───► sh (Bourne) ─► csh (C) ───► ksh ────► Bash ────► Bash 2 ──► Bash 5
          (AT&T)       (Berkeley)   (Korn)     (GNU)
```

Actualmente, **Bash 5.x** es el estándar por defecto en la mayoría de las distribuciones Linux y servidores de producción. En macOS se adoptó Zsh en 2019, aunque Bash sigue estando totalmente disponible. En Windows se utiliza principalmente mediante WSL (*Windows Subsystem for Linux*) o Git Bash.

---

### 1.2. Bash como lenguaje de programación

Bash es un lenguaje **procedural (imperativo)** e interpretado secuencialmente. 

* **Características principales:**
  - Ejecuta comandos línea por línea desde arriba hacia abajo.
  - No posee tipos de datos estrictos (todas las variables se almacenan internamente como cadenas de texto, interpretándose numéricamente al realizar operaciones aritméticas).
  - Ámbito de variables global por defecto (debe usarse la palabra clave `local` dentro de funciones para limitar su alcance).
  - Manejo de errores mediante **códigos de salida** (*Exit status* de 0 a 255).

#### Criterio técnico: ¿cuándo usar Bash vs. Python?

| Criterio / Necesidad | Usar Bash scripting | Usar Python u otro lenguaje |
| :--- | :--- | :--- |
| **Automatización del sistema** | ✅ Ideal para conectar comandos del SO. | ❌ Innecesariamente verboso. |
| **Glue code (unir programas)** | ✅ Redirección nativa de pipes y archivos. | ❌ Requiere módulos `subprocess`. |
| **Lógica de negocio y estructuras** | ❌ Difícil de mantener si crece mucho. | ✅ Excelente soporte para POO y estructuras. |
| **Manipulación masiva de datos** | ❌ Limitado en rendimiento y tipos. | ✅ Librerías potentes (Pandas, JSON, APIs). |

> 💡 **Regla de oro de mantenimiento:** Si un script de Bash supera las **100 - 200 líneas de código** o requiere estructuras de datos complejas (objetos, arrays anidados), es momento de migrar la solución a Python.

---

### 1.3. Anatomía del prompt

El prompt de la terminal provee información clave del contexto de ejecución:

```
usuario@servidor:~/proyecto$
   │       │        │      │
   │       │        │      └── $ = Usuario normal (# = Usuario root)
   │       │        └───────── Directorio actual (~ representa la carpeta Home)
   │       └────────────────── Nombre del equipo (Hostname)
   └────────────────────────── Nombre del usuario activo
```

---

### 1.4. Atajos de teclado indispensables en la terminal

| Atajo | Función y utilidad |
| :--- | :--- |
| **`Tab`** | Autocompleta comandos, rutas y archivos (presionar `Tab` 2 veces lista opciones). |
| **`Ctrl + C`** | Envía la señal `SIGINT` para cancelar el comando en ejecución actual. |
| **`Ctrl + D`** | Envía la señal `EOF` (*End of File*) para cerrar la sesión actual de la terminal. |
| **`Ctrl + L`** | Limpia la pantalla (equivalente al comando `clear`). |
| **`Ctrl + A`** | Mueve el cursor al **inicio** de la línea. |
| **`Ctrl + E`** | Mueve el cursor al **final** de la línea. |
| **`Ctrl + U`** | Borra todo el texto desde la posición del cursor hacia el inicio. |
| **`Ctrl + K`** | Borra todo el texto desde la posición del cursor hacia el final. |
| **`Ctrl + R`** | Abre la búsqueda interactiva inversa en el historial de comandos. |
| **`↑` / `↓`** | Navega verticalmente por el historial de comandos ejecutados. |

---

## 2. Navegación, sistema de archivos y permisos

### 2.1. El árbol de directorios de Linux

Linux organiza los archivos bajo un único árbol jerárquico cuyo origen absoluto es la raíz `/`:

```
/ (Raíz del sistema)
├── home/                ← Directorios personales de los usuarios (/home/usuario)
├── etc/                 ← Archivos de configuración globales del sistema y servicios
├── var/                 ← Archivos de datos variables (logs en /var/log, bases de datos)
├── tmp/                 ← Archivos temporales (se borran al reiniciar)
├── usr/                 ← Binarios y librerías de aplicaciones del usuario
└── bin/                 ← Comandos esenciales del sistema operativo (ls, cd, cp)
```

#### Rutas absolutas vs. relativas

| Tipo de ruta | Carácter inicial | Ejemplo | Descripción |
| :--- | :---: | :--- | :--- |
| **Absoluta** | `/` | `/home/usuario/proyecto/main.sh` | Ruta completa sin importar la posición actual. |
| **Relativa** | `.` o nombre | `./proyecto/main.sh` | Ruta calculada a partir del directorio actual. |
| **Home** | `~` | `~/proyecto/main.sh` | Atajo que apunta al `$HOME` del usuario activo. |

---

### 2.2. Comandos de navegación y gestión de archivos

#### Navegación y consulta
* **`pwd`** (*Print Working Directory*): Muestra la ruta absoluta del directorio actual.
* **`cd`** (*Change Directory*):
  - `cd /var/log`: Va a la ruta especificada.
  - `cd ..`: Sube un nivel en la jerarquía de directorios.
  - `cd ~` o solo `cd`: Regresa al directorio Home del usuario.
  - `cd -`: Regresa al directorio anterior en el que se estuvo parado.
* **`ls`** (*List*): Lista el contenido de un directorio.
  - `ls -l`: Formato largo (detalla permisos, propietario, tamaño y fecha).
  - `ls -a`: Muestra archivos ocultos (archivos cuyo nombre inicia con `.`).
  - `ls -lh`: Muestra tamaños en formato legible por humanos (KB, MB, GB).
  - `ls -lt`: Ordena archivos por fecha de modificación (más recientes primero).

#### Creación y manipulación
* **`mkdir`**: Crea directorios. Usar `mkdir -p carpeta/subcarpeta` para crear carpetas anidadas de forma recursiva.
* **`touch`**: Crea un archivo vacío si no existe o actualiza su marca de tiempo (*timestamp*) si ya existe.
* **`cp`**: Copia archivos o directorios. Usar `cp -r origen/ destino/` para copiar carpetas recursivamente.
* **`mv`**: Mueve o renombra archivos y directorios (`mv viejo.txt nuevo.txt`).
* **`rm`**: Elimina archivos. Usar `rm -r` para carpetas y `rm -rf` para forzar el borrado sin confirmaciones.

> ⚠️ **PRECAUCIÓN DE SEGURIDAD:** El comando `rm` en Linux elimina archivos de forma directa sin pasar por una papelera de reciclaje. NUNCA ejecutar `rm -rf /` o `rm -rf *` sin verificar previamente la ubicación actual con `pwd`.

#### Visualización de archivos
* **`cat`**: Muestra todo el contenido del archivo de forma continua en la terminal.
* **`less`**: Abre un visor paginado interactivo (avanzar con `Espacio`, buscar con `/texto`, salir con `q`).
* **`head`**: Muestra las primeras líneas de un archivo (`head -n 20 archivo.txt`).
* **`tail`**: Muestra las últimas líneas de un archivo (`tail -n 20 archivo.txt`). Usar `tail -f /var/log/syslog` para monitorear un archivo de log en tiempo real.

#### Wildcards / comodines de búsqueda
* **`*`**: Coincide con cero o más caracteres (`ls *.log`).
* **`?`**: Coincide con exactamente un carácter (`ls foto?.jpg`).
* **`[abc]`**: Coincide con cualquiera de los caracteres entre corchetes (`ls doc[123].pdf`).
* **`[a-z]`**: Coincide con un rango de caracteres (`ls [a-z]*.txt`).
* **`[!abc]`**: Negación; coincide con caracteres que NO estén en la lista.

---

### 2.3. Permisos de archivos y modelo de seguridad

Cada archivo o directorio en Unix posee un esquema de permisos asignado a 3 entidades:
1. **Propietario (`u` - User)**: El usuario creador del archivo.
2. **Grupo (`g` - Group)**: El grupo de usuarios asociado al archivo.
3. **Otros (`o` - Others)**: El resto de los usuarios del sistema.

```
Visualización con 'ls -l':

- r w x r - x r - -   1   usuario   grupo   4096   mar 12 10:00   script.sh
│ └──┬──┘ └──┬──┘ └──┬──┘
│    │       │       └── Permisos de Otros (r--) -> Solo lectura (4)
│    │       └────────── Permisos de Grupo (r-x) -> Lectura y ejecución (4+1=5)
│    └────────────────── Permisos del Propietario (rwx) -> Lectura, escritura y ejecución (4+2+1=7)
└─────────────────────── Tipo de archivo (- archivo normal, d directorio, l enlace simbólico)
```

#### Notación octal / numérica
Cada permiso individual tiene una representación numérica decimal:
* **`r` (Read / Lectura)** = `4`
* **`w` (Write / Escritura)** = `2`
* **`x` (Execute / Ejecución)** = `1`

Se suman los valores correspondientes para cada rol:

| Permisos | Operación de suma | Código octal | Descripción |
| :---: | :---: | :---: | :--- |
| `rwx` | $4 + 2 + 1$ | **`7`** | Control total (lectura, escritura y ejecución). |
| `rw-` | $4 + 2 + 0$ | **`6`** | Lectura y escritura (estándar para archivos de datos). |
| `r-x` | $4 + 0 + 1$ | **`5`** | Lectura y ejecución (estándar para scripts y carpetas). |
| `r--` | $4 + 0 + 0$ | **`4`** | Solo lectura. |
| `---` | $0 + 0 + 0$ | **`0`** | Sin ningún permiso de acceso. |

#### Comandos `chmod` y `chown`
```bash
# Otorgar permisos ejecutables al propietario y lectura/ejecución al resto (755)
chmod 755 script.sh

# Otorgar permiso de ejecución exclusivamente al propietario usando notación simbólica
chmod u+x script.sh

# Cambiar el propietario y el grupo de un archivo
sudo chown usuario:grupo archivo.txt

# Aplicar permisos de forma recursiva a todo el contenido de un directorio
chmod -R 644 /var/www/html/
```

---

## 3. Expansiones de la shell y redirección de I/O

### 3.1. Expansiones del intérprete de comandos

Antes de ejecutar un comando, Bash realiza un proceso interno denominado **expansión**, donde reemplaza ciertas expresiones por sus valores reales:

#### 1. Expansión de variables (`$`) y sustitución de comandos (`$()`)
```bash
# Asignar salida de un comando a una variable (sintaxis moderna $(cmd))
USUARIO_ACTIVO=$(whoami)
FECHA_HOY=$(date +%Y-%m-%d)

echo "El usuario $USUARIO_ACTIVO ejecutó la tarea el $FECHA_HOY"
```

#### 2. Operaciones aritméticas (`$(( ... ))`)
Bash permite realizar cálculos numéricos enteros nativamente:
```bash
NUM1=10
NUM2=5
RESULTADO=$(( NUM1 * NUM2 + 3 ))
echo "Resultado: $RESULTADO"  # Salida: 53
```

#### 3. Expansión del historial (`!`)
* **`!!`**: Reemplaza la expresión por el **último comando completo** ejecutado. Muy común para anteponer permisos de superusuario si olvidamos `sudo`:
  ```bash
  apt update          # Falla por falta de permisos
  sudo !!             # Ejecuta: sudo apt update
  ```
* **`!ls`**: Reejecuta el último comando registrado en el historial que haya iniciado con `ls`.

#### 4. Reglas de comillas y escape (`\`)
* **Comillas dobles (`"..."`)**: Permite la expansión de variables (`$VAR`) y sustitución de comandos (`$(cmd)`).
* **Comillas simples (`'...'`)**: Trata todo el texto de forma **literal**. Desactiva cualquier tipo de expansión.
* **Backslash (`\`)**: Cancela el significado especial del carácter que le sigue inmediatamente.

```bash
NOMBRE="Carlos"

echo "Hola $NOMBRE"   # Salida: Hola Carlos
echo 'Hola $NOMBRE'   # Salida: Hola $NOMBRE (Literal)
echo "Precio: \$100"  # Salida: Precio: $100 (Escape)
```

---

### 3.2. Descriptores de archivo y redirección de I/O

Cada proceso que se ejecuta en Linux posee 3 canales estándar abiertos por defecto:

```
                   ┌──────────────────────────────────────┐
  Teclado ────────>│ 0 - stdin  (Entrada estándar)        │
                   │                                      │
  Pantalla <───────┤ 1 - stdout (Salida estándar)         │
                   │                                      │
  Pantalla <───────┤ 2 - stderr (Salida estándar errores) │
                   └──────────────────────────────────────┘
```

#### Operadores de redirección

```bash
# 1. Redirigir stdout a un archivo (sobrescribe el archivo si existe)
echo "Fila 1" > salida.txt

# 2. Anexar stdout al final de un archivo (append)
echo "Fila 2" >> salida.txt

# 3. Redirigir únicamente los errores (stderr) a un archivo
ls /carpeta_inexistente 2> errores.log

# 4. Redirigir stdout (1) y stderr (2) al mismo archivo
./script.sh > salida_completa.log 2>&1
# Sintaxis corta equivalente en Bash:
./script.sh &> salida_completa.log

# 5. Descartar la salida enviándola al "agujero negro" (/dev/null)
comando_ruidoso > /dev/null 2>&1

# 6. Redirigir entrada desde un archivo (stdin <)
wc -l < datos.txt
```

#### Here-documents (`<< EOF`) y Here-strings (`<<<`)
Permiten enviar bloques multilínea o cadenas directas a la entrada estándar de un comando:

```bash
# Escribir un archivo de configuración multilínea de forma directa
cat << EOF > configuracion.conf
PUERTO=8080
ENTORNO=produccion
BD_HOST=localhost
EOF

# Pasar una variable directamente a un comando sin usar echo (Here-string)
read USUARIO <<< "admin"
```

#### El comando `tee`
Permite duplicar el flujo de salida: envía el resultado a la pantalla **y simultáneamente** lo guarda en un archivo. Usar `tee -a` para anexar sin sobrescribir.

```bash
echo "Registro de auditoría" | tee -a auditoria.log
```

---

### 3.3. Tuberías (`|`) y operadores de control de flujo

#### Tuberías / pipelines (`|`)
Conectan la salida estándar (`stdout`) de un comando emisor directamente con la entrada estándar (`stdin`) del comando receptor:

```bash
# Filtrar y contar procesos activos de un usuario
ps aux | grep "usuario" | wc -l
```

#### Operadores de control de ejecución (`&&`, `||`, `;`)

| Operador | Nombre | Regla de ejecución | Ejemplo práctico |
| :---: | :---: | :--- | :--- |
| **`&&`** | **AND** | Ejecuta el segundo comando **solo si** el primero finalizó exitosamente (`exit status 0`). | `mkdir app && cd app` |
| **`||`** | **OR** | Ejecuta el segundo comando **solo si** el primero falló (`exit status != 0`). | `cd app || mkdir app` |
| **`;`** | **Secuencia** | Ejecuta los comandos en orden **sin importar** el resultado del anterior. | `clean; build; deploy` |

```bash
# Ejemplo combinado de producción: notificación según resultado
./backup.sh && echo "✓ Backup exitoso" || echo "✗ El backup ha fallado"
```

---

## 4. Scripting en Bash y modo estricto

### 4.1. Anatomía de un script y shebang

Un **script en Bash** es un archivo de texto con permisos de ejecución interpretado de forma secuencial.

```bash
#!/bin/bash
# Shebang (#!): Especifica la ruta absoluta del intérprete que ejecutará el script.

# Modo estricto de manejo de errores (Best practice para producción)
set -euo pipefail

echo "Iniciando despliegue..."
```

#### El modo estricto (`set -euo pipefail`)
* **`-e` (`errexit`)**: Cancela la ejecución del script inmediatamente si cualquier comando devuelve un código de error distinto de cero.
* **`-u` (`nounset`)**: Trata el uso de variables no declaradas o vacías como un error explícito en lugar de sustituirlas silenciosamente por cadenas vacías.
* **`-o pipefail`**: Garantiza que si una serie de comandos en tubería (`|`) falla en cualquier punto, toda la tubería devuelva el código de error del comando que falló.

---

### 4.2. Variables en Bash y ámbito

#### Regla de sintaxis: SIN ESPACIOS
En Bash, la asignación de variables **NO debe llevar espacios** alrededor del signo igual `=`:

```bash
# ❌ INCORRECTO (Produce error 'command not found')
nombre = "Juan"

# ✅ CORRECTO
nombre="Juan"
edad=30
```

#### Variables especiales reservadas
Al ejecutar un script (`./script.sh arg1 arg2`), Bash asigna automáticamente los valores en las siguientes variables:

| Variable | Significado técnico |
| :---: | :--- |
| **`$0`** | Nombre o ruta con la que se invocó el script. |
| **`$1`, `$2`** | Primer y segundo argumento pasados al script. |
| **`$#`** | Cantidad total de argumentos pasados. |
| **`$@`** | Colección de todos los argumentos pasados como elementos separados e independientes. |
| **`$?`** | Código de salida (*Exit status*) del último comando ejecutado (`0` = Éxito, `1-255` = Error). |
| **`$$`** | PID (*Process ID*) del proceso actual del script. |

#### Expansión avanzada de parámetros en variables
Permite manipular cadenas directamente sin necesidad de invocar herramientas externas:

```bash
ARCHIVO="/var/log/nginx/access.log.gz"

# 1. Asignar valor por defecto si la variable está vacía (${VAR:-defecto})
PUERTO="${PUERTO_ENV:-8080}"

# 2. Obtener la longitud en caracteres (${#VAR})
echo "${#ARCHIVO}"      # Imprime la cantidad de caracteres

# 3. Eliminar el sufijo más corto que coincida con el patrón (${VAR%patron})
echo "${ARCHIVO%.gz}"   # Salida: /var/log/nginx/access.log

# 4. Eliminar el prefijo más largo que coincida con el patrón (${VAR##patron})
echo "${ARCHIVO##*/}"   # Salida: access.log.gz (Extrae solo el nombre del archivo)

# 5. Reemplazo de subcadena (${VAR/buscado/reemplazo})
DOMINIO="usuario@hotmail.com"
echo "${DOMINIO/hotmail/gmail}"  # Salida: usuario@gmail.com
```

---

## 5. Estructuras de control y funciones

### 5.1. Evaluadores condicionales `[ ... ]` vs. `[[ ... ]]`

En Bash existen dos sintaxis de prueba condicional. El operador extendido `[[ ... ]]` es técnicamente superior al comando clásico `[ ... ]` (*test*):

```
┌──────────────────────────────────────────────────────────────────┐
│                  COMPARATIVA [ ... ] vs [[ ... ]]                │
├───────────────────────────────┬──────────────────────────────────┤
│      Comando Test [ ... ]     │ Extended Test [[ ... ]]          │
├───────────────────────────────┼──────────────────────────────────┤
│ Requiere comillas dobles      │ Maneja espacios y valores nulos  │
│ para evitar Word Splitting.   │ de forma segura automáticamente. │
├───────────────────────────────┼──────────────────────────────────┤
│ Usa operadores legacy         │ Soporta operadores nativos       │
│ `-a` (AND) y `-o` (OR).       │ `&&`, `||` y paréntesis.         │
├───────────────────────────────┼──────────────────────────────────┤
│ Sin soporte de Regex.         │ Soporta Regex con `=~`           │
│                               │ y comodines de texto (glob).     │
└───────────────────────────────┴──────────────────────────────────┘
```

#### Operadores de comparación

* **Comparaciones numéricas:**
  - `-eq`: Igual (*Equal*)
  - `-ne`: No igual (*Not Equal*)
  - `-gt`: Mayor que (*Greater Than*)
  - `-lt`: Menor que (*Less Than*)
  - `-ge`: Mayor o igual (*Greater or Equal*)
  - `-le`: Menor o igual (*Less or Equal*)

* **Comparaciones de cadenas:**
  - `==` / `!=`: Igualdad o desigualdad de texto.
  - `-z`: Verdadero si la longitud de la cadena es cero (está vacía).
  - `-n`: Verdadero si la cadena NO está vacía.
  - `=~`: Evaluación frente a expresión regular.

---

### 5.2. Estructuras de control (`if`, `for`, `while`, `until`, `case`)

#### 1. Condicional `if`
```bash
if [[ "$EDAD" -ge 18 ]]; then
    echo "Mayor de edad"
elif [[ "$EDAD" -gt 12 ]]; then
    echo "Adolescente"
else
    echo "Menor de edad"
fi
```

#### 2. Bucle `for` (iterativo)
```bash
# Iterar sobre una lista de elementos o wildcard de archivos
for ARCHIVO in *.txt; do
    echo "Procesando $ARCHIVO..."
done

# Iterar sobre una secuencia numérica
for i in {1..5}; do
    echo "Iteración $i"
done
```

#### 3. Bucle `while` (mientras sea verdadero)
```bash
CONTAGIO=1
while [[ $CONTAGIO -le 5 ]]; do
    echo "Contador: $CONTAGIO"
    (( CONTAGIO++ ))
done
```

#### 4. Selector `case` (múltiples patrones)
```bash
case "$OPCION" in
    "start")
        echo "Iniciando servicio..." ;;
    "stop")
        echo "Deteniendo servicio..." ;;
    *)
        echo "Opción no válida" ;;
esac
```

---

### 5.3. Funciones, ámbito local (`local`) y limpieza con `trap`

#### Definición de funciones y ámbito `local`
Las funciones permiten reutilizar bloques de código. Es fundamental declarar las variables internas con la palabra clave `local` para evitar contaminar el entorno global:

```bash
calcular_area() {
    local base=$1
    local altura=$2
    local area=$(( base * altura ))
    
    # Retornar el resultado imprimiéndolo a stdout
    echo "$area"
}

# Capturar el retorno de la función usando sustitución $()
RESULTADO=$(calcular_area 5 10)
echo "El área es: $RESULTADO"
```

#### Limpieza garantizada con `trap`
El comando `trap` captura señales del sistema operativo (como `SIGINT` por Ctrl+C o la salida del script `EXIT`) para ejecutar rutinas de limpieza antes de finalizar:

```bash
# Crear directorio temporal seguro
TMP_DIR=$(mktemp -d)

# Función de desecho
limpiar() {
    echo "Eliminando archivos temporales en $TMP_DIR..."
    rm -rf "$TMP_DIR"
}

# Configurar la trampa para que ejecute la función al salir
trap limpiar EXIT INT TERM
```

---

## 6. Expresiones regulares (Regex) y herramientas CLI

### 6.1. Expresiones regulares (Regex)

Una **expresión regular** es un patrón que define reglas de búsqueda y coincidencia sobre cadenas de texto.

| Elemento | Tipo | Descripción | Ejemplo de coincidencia |
| :---: | :---: | :--- | :--- |
| **`.`** | Carácter | Cualquier carácter excepto salto de línea. | `c.sa` -> `casa`, `cosa` |
| **`^` / `$`** | Ancla | Inicio (`^`) o Fin (`$`) de la línea. | `^ERROR` (Línea empieza con ERROR) |
| **`[abc]`** | Conjunto | Cualquier carácter dentro del corchete. | `[0-9]` (Cualquier dígito) |
| **`[^abc]`**| Negación | Cualquier carácter que NO esté en el conjunto.| `[^0-9]` (Cualquier no-dígito)|
| **`*`** | Cuantificador | 0 o más repeticiones del elemento anterior. | `a*` -> `""`, `"a"`, `"aaa"` |
| **`+`** | Cuantificador | 1 o más repeticiones del elemento anterior. | `a+` -> `"a"`, `"aaa"` |
| **`?`** | Cuantificador | 0 o 1 repetición (Elemento opcional). | `https?` -> `http`, `https` |
| **`{n,m}`** | Cuantificador | Entre $n$ y $m$ repeticiones exactas. | `\d{2,4}` -> `12`, `1234` |
| **`\b`** | Ancla | Límite de palabra (*Word boundary*). | `\bcat\b` -> `cat` (No coincide en `concat`) |

---

### 6.2. Herramientas CLI de procesamiento: `grep`, `sed` y `awk`

#### `grep` (Global Regular Expression Print)
Permite filtrar y buscar líneas que coincidan con un patrón dentro de archivos:
* **`-E`**: Habilita expresiones regulares extendidas (`+`, `?`, `{n}`, `\b`).
* **`-i`**: Búsqueda insensible a mayúsculas y minúsculas.
* **`-v`**: Invertir coincidencia (muestra las líneas que NO coinciden).
* **`-n`**: Imprime el número de línea.
* **`-c`**: Cuenta la cantidad total de coincidencias.
* **`-o`**: Extrae únicamente el texto coincidente exacto (no la línea entera).

```bash
# Extraer todos los correos electrónicos de un archivo de log
grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" usuarios.txt
```

#### `sed` (Stream Editor - Edición en flujo)
Permite transformar y modificar flujos de texto sobre la marcha:

```bash
# Sustituir la primera aparición en pantalla (s/buscado/reemplazo/)
sed 's/localhost/127.0.0.1/' config.txt

# Sustitución global e in situ dentro del mismo archivo (-i, /g)
sed -i 's/PUERTO=8080/PUERTO=9000/g' .env

# Eliminar líneas que comiencen con '#' (comentarios)
sed -i '/^#/d' configuracion.conf
```

#### `awk` (Procesamiento por columnas y reportes)
Lenguaje especializado en el procesamiento estructurado de tablas y archivos delimitados (CSV, TSV, logs):

```bash
# Imprimir las columnas 1 y 3 de un archivo CSV usando la coma como delimitador (-F)
awk -F',' '{print $1, $3}' usuarios.csv

# Filtrar y sumar valores de la columna 2 para líneas donde la columna 3 sea "ACTIVO"
awk -F',' '$3 == "ACTIVO" {suma += $2} END {print "Total activos:", suma}' datos.csv
```
