# Módulo 04: Terminal, Bash scripting y expresiones regulares (Regex)

Guía de estudio conceptual y técnica completa sobre la **Shell Unix, scripting en Bash, descriptores de entrada/salida, construcciones avanzadas y procesamiento de texto con Regex y herramientas CLI**.

---

## 1. Fundamentos de scripting en Bash y modo estricto

Un **script en Bash** es un archivo de texto ejecutable interpretado secuencialmente por la Shell de Linux.

```bash
#!/bin/bash
# Shebang (#!): Especifica la ruta absoluta del intérprete de comandos que ejecutará el script.

# Modo Estricto de Manejo de Errores (Best Practice de Producción)
set -euo pipefail

# Definición de variables (REGLA: Sin espacios al lado del signo =)
PUERTO=8080
ENTORNO="producción"
```

### El modo estricto (`set -euo pipefail`)
- **`-e` (`errexit`)**: Cancela la ejecución del script inmediatamente si cualquier comando devuelve un código de retorno distinto de cero (`error`).
- **`-u` (`nounset`)**: Trata las variables no declaradas o vacías como un error explícito en lugar de sustituirlas silenciosamente por cadenas vacías.
- **`-o pipefail`**: Si una serie de comandos conectados por tuberías (`|`) falla en cualquier punto, toda la tubería devuelve el código de error del comando fallido.

### Comillas dobles vs. comillas simples vs. comillas invertidas
- **Comillas dobles (`"..."`)**: Permite la expansión de variables (`$VAR`) y la evaluación de expresiones de sustitución (`$(cmd)`).
- **Comillas simples (`'...'`)**: Trata todo el texto de forma literal. Anula la expansión de cualquier variable o metacarácter.
- **Comillas invertidas (`` `cmd` ``)**: Sintaxis heredada (*legacy*) para sustitución de comandos. Se recomienda usar la sintaxis moderna `$(cmd)`.

---

## 2. Variables especiales y argumentos de entrada

Al invocar un script desde la terminal (ej. `./deploy.sh api v2.0`), Bash asigna los argumentos recibidos en variables reservadas:

| Variable | Significado técnico | Ejemplo de uso |
| :---: | :--- | :--- |
| **`$0`** | Nombre o ruta del script en ejecución. | `echo "Ejecutando $0"` |
| **`$1`, `$2`** | Primer y segundo argumento pasados al script. | `APLICACION=$1` |
| **`$#`** | Cantidad total de argumentos pasados al script. | `if (( $# < 2 )); then exit 1; fi` |
| **`$@`** | Colección de todos los argumentos pasados como elementos separados. | `for ARG in "$@"; do ... done` |
| **`$*`** | Todos los argumentos combinados en una única cadena separada por el primer carácter de IFS. | `echo "$*"` |
| **`$?`** | Código de salida (*Exit status*) del último comando (`0` = éxito, `1-255` = error). | `if [[ $? -ne 0 ]]; then ...` |
| **`$$`** | PID (*Process ID*) del proceso del script actual. | `echo "PID: $$"` |

---

## 3. Expansión de parámetros y sustitución en variables

Bash provee mecanismos avanzados para manipular el valor de variables sin necesidad de invocar herramientas externas como `sed` o `cut`:

```bash
ARCHIVO="/var/log/nginx/access.log.gz"

# 1. Valor por defecto si la variable está desconfigurada o vacía (${VAR:-defecto})
PUERTO="${PUERTO_ENV:-8080}"

# 2. Obtener la longitud de la cadena (${#VAR})
echo "${#ARCHIVO}"      # Imprime la cantidad de caracteres

# 3. Eliminar el sufijo más corto que coincida con un patrón (${VAR%patron})
echo "${ARCHIVO%.gz}"   # Salida: /var/log/nginx/access.log

# 4. Eliminar el sufijo más largo que coincida con un patrón (${VAR%%patron})
echo "${ARCHIVO%%.*}"   # Salida: /var/log/nginx/access

# 5. Eliminar el prefijo más corto que coincida con un patrón (${VAR#patron})
RUTA="lib/src/main.js"
echo "${RUTA#*/}"       # Salida: src/main.js

# 6. Eliminar el prefijo más largo que coincida con un patrón (${VAR##patron})
echo "${ARCHIVO##*/}"   # Salida: access.log.gz (Extrae solo el nombre del archivo)

# 7. Reemplazo de cadenas (${VAR/buscado/reemplazo})
CORREO="usuario@hotmail.com"
echo "${CORREO/hotmail/gmail}"  # Salida: usuario@gmail.com
```

---

## 4. Evaluadores condicionales: `[ ... ]` vs. `[[ ... ]]`

En Bash existen dos formas de realizar pruebas condicionales. El operador extendido `[[ ... ]]` es ampliamente superior al comando tradicional `[ ... ]` (*test*):

```
┌─────────────────────────────────────────────────────────────────┐
│                 COMPARATIVA [ ... ] vs [[ ... ]]                │
├──────────────────────────────┬──────────────────────────────────┤
│      Comando Test [ ... ]    │ Extended Test [[ ... ]]          │
├──────────────────────────────┼──────────────────────────────────┤
│ Requiere comillas dobles     │ Maneja espacios y nulos          │
│ para evitar Word Splitting.  │ de forma segura automáticamente. │
├──────────────────────────────┼──────────────────────────────────┤
│ Usa operadores legacy        │ Soporta operadores nativos       │
│ `-a` (AND) y `-o` (OR).      │ `&&`, `||` y paréntesis.         │
├──────────────────────────────┼──────────────────────────────────┤
│ Sin soporte de Regex.        │ Soporta Regex con `=~`           │
│                              │ y comodines de texto (glob).     │
└──────────────────────────────┴──────────────────────────────────┘
```

### Ejemplo con evaluación de Regex (`=~`)
```bash
EMAIL="alumno@fi.uba.ar"

# Verificar si el correo pertenece al dominio fi.uba.ar usando [[ ... ]]
if [[ "$EMAIL" =~ ^[a-zA-Z0-9._%+-]+@fi\.uba\.ar$ ]]; then
    echo "Correo institucional válido."
fi
```

---

## 5. Descriptores de archivo, redirecciones y pipes

Cada proceso iniciado en la Shell dispone de 3 descriptores de archivo estándar:

```
                  ┌─────────────────────────────────────┐
  Teclado ───────>│ 0 - stdin  (Entrada estándar)       │
                  │                                     │
  Pantalla <──────┤ 1 - stdout (Salida estándar)        │
                  │                                     │
  Pantalla <──────┤ 2 - stderr (Salida estándar errores)│
                  └─────────────────────────────────────┘
```

### Redirecciones de salida y comando `tee`
```bash
# Sobreescribir stdout a un archivo
echo "Inicio" > app.log

# Anexar stdout al final de un archivo (append)
echo "Nueva entrada" >> app.log

# Redirigir únicamente los errores (stderr)
ls /directorio_inexistente 2> errores.log

# Redirigir stdout y stderr al mismo archivo
./script.sh &> salida_completa.log

# Descartar salidas enviándolas al agujero negro (/dev/null)
comando_ruidoso > /dev/null 2>&1

# Comando tee: Muestra la salida por pantalla Y al mismo tiempo la guarda en archivo
echo "Mensaje de auditoría" | tee -a auditoria.log
```

### Tuberías (`|`)
Una tubería conecta la salida estándar (`stdout`) del comando emisor directamente con la entrada estándar (`stdin`) del comando receptor:
```bash
cat /var/log/auth.log | grep "Failed password" | awk '{print $11}' | sort | uniq -c
```

---

## 6. Funciones, captura de señales (`trap`) y subshells

### Funciones y ámbito local (`local`)
```bash
crear_usuario() {
    # Usar 'local' para evitar contaminar las variables globales del script
    local usuario=$1
    local rol=$2
    
    echo "Creando usuario $usuario con rol $rol..."
    # Retornar datos imprimibles a través de stdout
    echo "SUCCESS:$usuario"
}

# Capturar la salida de la función usando sustitución de comandos $()
RESPUESTA=$(crear_usuario "usuario" "admin")
```

### Limpieza garantizada con `trap`
El comando `trap` captura señales del sistema operativo (como `SIGINT` por Ctrl+C o `SIGTERM`) para ejecutar una función de limpieza antes de finalizar:

```bash
# Definir directorio temporal
TMP_DIR=$(mktemp -d)

# Función de limpieza
limpiar_temporales() {
    echo "Limpiando archivos temporales en $TMP_DIR..."
    rm -rf "$TMP_DIR"
}

# Configurar la trampa para que ejecute la limpieza al salir (EXIT)
trap limpiar_temporales EXIT INT TERM
```

---

## 7. Expresiones regulares (Regex)

Una **Expresión Regular** es una secuencia de patrones que define reglas de búsqueda sobre cadenas de texto.

### Catálogo de metacaracteres y cuantificadores

| Elemento | Tipo | Descripción | Ejemplo |
| :---: | :---: | :--- | :--- |
| **`.`** | Carácter | Coincide con cualquier carácter excepto salto de línea. | `c.sa` -> `casa`, `cosa` |
| **`^` / `$`** | Ancla | Inicio (`^`) o Fin (`$`) de la línea. | `^ERROR` (Empieza con ERROR) |
| **`[abc]`** | Conjunto | Coincide con cualquier carácter dentro del corchete. | `[0-9]` (Cualquier dígito) |
| **`[^abc]`**| Negación | Coincide con cualquier carácter que NO esté en el conjunto.| `[^a-z]` (No letra minúscula)|
| **`*`** | Cuantificador | 0 o más repeticiones del elemento anterior. | `a*` -> `""`, `"a"`, `"aaa"` |
| **`+`** | Cuantificador | 1 o más repeticiones del elemento anterior. | `a+` -> `"a"`, `"aaa"` |
| **`?`** | Cuantificador | 0 o 1 repetición (Elemento opcional). | `https?` -> `http`, `https` |
| **`{n,m}`** | Cuantificador | Entre $n$ y $m$ repeticiones exactas. | `\d{2,4}` -> `12`, `1234` |
| **`\b`** | Ancla | Límite de palabra (*Word boundary*). | `\bcat\b` -> `cat` (no `concat`) |

---

## 8. Herramientas CLI de procesamiento: `grep`, `sed` y `awk`

### `grep` (Global Regular Expression Print)
- **`-E`**: Habilita expresiones regulares extendidas (`+`, `?`, `{n}`, `\b`).
- **`-i`**: Búsqueda insensible a mayúsculas/minúsculas.
- **`-v`**: Invertir coincidencia (Muestra líneas que NO coinciden).
- **`-n`**: Imprime el número de línea.
- **`-c`**: Cuenta la cantidad de líneas coincidentes.
- **`-o`**: Extrae únicamente el texto coincidente exacto (no la línea completa).
- **`-q`**: Modo silencioso (Devuelve exit code `0` si encuentra coincidencia).

```bash
# Extraer todos los correos del archivo
grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" contactos.txt
```

### `sed` (Stream Editor - Edición en flujo)
```bash
# Sustitución simple en pantalla (s/buscado/reemplazo/)
sed 's/localhost/127.0.0.1/' config.txt

# Sustitución global en el mismo archivo (-i: in-place, /g: global)
sed -i 's/PUERTO=8080/PUERTO=9000/g' .env

# Eliminar líneas que coincidan con un patrón (/patrón/d)
sed -i '/^#/d' configuracion.conf
```

### `awk` (Procesamiento de columnas y reportes)
```bash
# Imprimir columnas 1 y 3 de un CSV (-F especifica el delimitador)
awk -F',' '{print $1, $3}' usuarios.csv

# Filtrar y sumar valores de la columna 2 para las líneas que cumplen condición
awk -F',' '$3 == "ACTIVO" {suma += $2} END {print "Total Activos:", suma}' datos.csv
```
