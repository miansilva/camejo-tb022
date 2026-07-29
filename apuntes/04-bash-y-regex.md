# Módulo 04: Automatización con Bash scripting y expresiones regulares (Regex)

Guía de estudio completa para la automatización de tareas en servidores Linux mediante **Bash scripting** y la búsqueda/procesamiento de texto avanzado con **Expresiones regulares (Regex)**.

---

## 1. Fundamentos de scripting en Bash

Un **Bash script** es un archivo de texto ejecutable que contiene una secuencia de comandos interpretados línea por línea por el shell de Linux.

### Estructura canónica de un script de producción

```bash
#!/bin/bash
# Shebang: Indica al sistema operativo la ruta del intérprete a ejecutar

# Habilitar Modo estricto de manejo de errores (Buenas prácticas de la industria)
set -euo pipefail

# Definición de variables (REGLA: Sin espacios alrededor del signo =)
NOMBRE_APP="CamejoAPI"
PUERTO=8080
LOG_FILE="/var/log/app.log"

echo "=== Iniciando servicio $NOMBRE_APP en puerto $PUERTO ==="
```

#### Explicación de las banderas del modo estricto (`set -euo pipefail`):
- **`-e`** (*Exit on error*): El script finaliza inmediatamente si cualquier comando devuelve un código de salida distinto de cero (`error`).
- **`-u`** (*Unset variables*): Trata el uso de variables no definidas como un error explícito.
- **`-o pipefail`**: Si un comando falla dentro de una tubería (`pipe`), toda la tubería devuelve un código de falla.

### Variables especiales y argumentos de entrada

Cuando ejecutás un script desde la terminal (ej. `./deploy.sh producción v1.2`), Bash captura los parámetros en variables automáticas:

| Variable | Significado | Ejemplo de uso |
| :---: | :--- | :--- |
| **`$0`** | Nombre o ruta del script en ejecución. | `echo "Ejecutando $0"` |
| **`$1`, `$2`** | Primer y segundo argumento pasado al script. | `ENTORNO=$1` |
| **`$#`** | Cantidad total de argumentos pasados. | `if [ $# -lt 1 ]; then ...` |
| **`$@`** | Lista con todos los argumentos pasados como palabras separadas. | `for ARG in "$@"; do ...` |
| **`$?`** | Código de salida del último comando ejecutado (`0` = éxito, `>0` = error). | `if [ $? -ne 0 ]; then ...` |
| **`$$`** | PID (Identificador de Proceso) del script actual. | `echo "PID: $$"` |

---

## 2. Aritmética, funciones y estructuras de datos en Bash

### Operaciones matemáticas y evaluación `(( ))`
Bash permite realizar operaciones aritméticas nativas con enteros utilizando la doble sintaxis de paréntesis `(( ))`:

```bash
# Evaluación de expresiones y modificaciones de variables
(( contador++ ))
(( total = cantidad * precio ))

# Comparaciones numéricas nativas dentro de (( ))
if (( edad >= 18 )); then
    echo "Mayor de edad"
fi

# Bucles numéricos estilo C (Ideal cuando el límite es una variable)
LIMIT=10
for (( i=1; i<=LIMIT; i++ )); do
    echo "Iteración $i"
done
```

### Funciones y ámbito local (`local`)

Las funciones en Bash deben declarar sus variables internas utilizando la palabra clave **`local`** para evitar contaminar el entorno global del script.

```bash
calcular_total() {
    local precio=$1
    local cantidad=$2
    local total=$(( precio * cantidad ))
    
    # Las funciones retornan datos a través de la salida estándar (stdout)
    echo "$total"
}

# Captura del resultado retornado mediante sustitución de comandos $()
RESULTADO=$(calcular_total 150 3)
echo "El total calculado es: $RESULTADO"
```

> ⚠️ **Aclaración**: El comando `return` en Bash únicamente devuelve un código de salida numérico de estado (`0` a `255`), no datos. Para devolver datos se utiliza `echo` y se captura con `$(...)`.

### Manejo de arreglos (Arrays)

```bash
# Declaración de un arreglo
servidores=("web01" "web02" "db01")

# Acceso a elementos
echo "${servidores[0]}"       # Primer elemento ("web01")
echo "${servidores[@]}"       # Todos los elementos
echo "${#servidores[@]}"      # Cantidad total de elementos (longitud)

# Agregar un nuevo elemento
servidores+=("cache01")

# Eliminar un elemento
unset servidores[1]
```

---

## 3. Descriptores de archivo, redirecciones y pipes

En UNIX existen 3 descriptores de archivo estándar abiertos por defecto para cada proceso:

```
                  ┌─────────────────────────────────┐
  Teclado ───────>│ 0 - stdin  (Entrada estándar)   │
                  │                                 │
  Pantalla <──────┤ 1 - stdout (Salida estándar)    │
                  │                                 │
  Pantalla <──────┤ 2 - stderr (Salida de errores)  │
                  └─────────────────────────────────┘
```

### Operadores de redirección

```bash
# Redirección de stdout sobreescribiendo el archivo de destino
echo "Log de inicio" > salida.log

# Redirección de stdout anexando al final del archivo (append)
echo "Nueva entrada" >> salida.log

# Redirección exclusiva de los errores (stderr) a un archivo
ls /carpeta_inexistente 2> errores.log

# Redirección combinada de stdout y stderr al mismo archivo
./script.sh &> todo.log

# Descartar salidas o errores enviándolos al hoyo negro del sistema (/dev/null)
comando_ruidoso > /dev/null 2>&1
```

### Tuberías (Pipes `|`)

El operador pipe (`|`) conecta la salida estándar (`stdout`) del comando de la izquierda directamente con la entrada estándar (`stdin`) del comando de la derecha.

```bash
# Filtrar errores 500 en un log de accesos y contar cuántos ocurrieron
cat access.log | grep "HTTP/1.1 500" | wc -l
```

---

## 4. Estructuras de control de flujo

### Condicionales (`if / elif / else`)

```bash
#!/bin/bash

RUTA_CONFIG="/etc/nginx/nginx.conf"

# Evaluar si el archivo existe (-f) y es legible (-r)
if [ -f "$RUTA_CONFIG" ] && [ -r "$RUTA_CONFIG" ]; then
    echo "El archivo de configuración existe y es accesible."
elif [ -d "/etc/nginx" ]; then
    echo "La carpeta existe pero falta el archivo nginx.conf."
else
    echo "Nginx no está instalado."
    exit 1
fi
```

#### Operadores de test frecuentes `[ ... ]`
- **En archivos**: `-f` (es archivo regular), `-d` (es directorio), `-e` (existe), `-s` (no está vacío).
- **En cadenas**: `-z` (cadena vacía), `-n` (cadena no vacía), `=` (iguales), `!=` (distintas).
- **En números**: `-eq` (igual $=), `-ne` (distinto $\neq$), `-gt` (mayor $>$), `-ge` (mayor o igual $\ge$), `-lt` (menor $<$), `-le` (menor o igual $\le$).

### Selección múltiple (`case ... in`)

```bash
case $ACCION in
    iniciar)
        echo "Iniciando servicio..."
        ;;
    detener)
        echo "Deteniendo servicio..."
        ;;
    *)
        echo "Opción no válida."
        exit 1
        ;;
esac
```

### Bucles de validación (`while`)

```bash
# Patrón de validación interactiva de entrada
while true; do
    read -p "Ingrese un número entre 1 y 5: " OPCION
    case $OPCION in
        [1-5])
            echo "Opción válida: $OPCION"
            break
            ;;
        *)
            echo "Entrada inválida. Intente de nuevo."
            continue
            ;;
    esac
done
```

---

## 5. Expresiones regulares (Regex)

Una **Expresión regular** es una secuencia de caracteres que forma un patrón de búsqueda para coincidir, evaluar o reemplazar texto.

### Metacaracteres y clases básicas

| Patrón | Significado | Ejemplo de coincidencia |
| :---: | :--- | :--- |
| **`.`** | Cualquier carácter único (excepto salto de línea). | `c.sa` coincide con `casa`, `cosa`, `c3sa`. |
| **`^`** | Ancla al inicio de la línea. | `^ERROR` coincide solo si la línea empieza con `ERROR`. |
| **`$`** | Ancla al final de la línea. | `final$` coincide solo si la línea termina en `final`. |
| **`[abc]`** | Cualquier carácter dentro del conjunto. | `[aeiou]` coincide con cualquier vocal. |
| **`[^abc]`**| Negación: cualquier carácter que NO esté en el conjunto.| `[^0-9]` coincide con cualquier no-dígito. |
| **`[a-z]`** | Rango de caracteres. | `[A-Z]` coincide con cualquier letra mayúscula. |
| **`\b`** | Límite de palabra (word boundary). | `\bcat\b` coincide solo con la palabra exacta `cat`. |

### Cuantificadores

- **`*`**: 0 o más repeticiones de la entidad anterior.
- **`+`**: 1 o más repeticiones de la entidad anterior.
- **`?`**: 0 o 1 repetición (hace que el elemento sea opcional).
- **`{n,m}`**: Entre $n$ y $m$ repeticiones (ej. `\d{2,4}`).

### Aserciones avanzadas PCRE (Lookaheads y Lookbehinds con `grep -P`)

Las aserciones de Perl no consumen caracteres en el puntero de búsqueda; solo verifican condiciones de contorno:

- **Lookahead positivo `(?=patron)`**: Exige que el `patron` exista a continuación.
- **Lookahead negativo `(?!patron)`**: Exige que el `patron` NO exista a continuación.
- **Lookbehind positivo `(?<=patron)`**: Exige que el `patron` exista justo antes del punto actual.
- **Lookbehind negativo `(?<!patron)`**: Exige que el `patron` NO exista justo antes.

```bash
# Extraer la contraseña en un CSV sin incluir la coma previa usando Lookbehind positivo
grep -oP "(?<=,)[^,]+$" contraseñas.csv
```

---

## 6. Herramientas CLI de procesamiento de texto (`grep`, `sed`, `awk`)

### 1. `grep` (Global Regular Expression Print)

| Banderas | Función |
| :---: | :--- |
| **`-E`** | Habilita expresiones regulares extendidas (`+`, `?`, `{n}`, `\b`, `\w`). |
| **`-P`** | Habilita motor PCRE/Perl (`\d`, Lookaheads `(?=...)`, Lookbehinds `(?<=...)`). |
| **`-o`** | Imprime **únicamente** el texto que coincide con el patrón (no la línea entera). |
| **`-q`** | Modo silencioso (devuelve exit code `0` si encuentra coincidencia; ideal para `if`). |
| **`-i`** | Búsqueda insensible a mayúsculas/minúsculas. |
| **`-v`** | Invertir la coincidencia (muestra las líneas que NO coinciden). |

```bash
# Búsqueda silenciosa en un condicional if
if grep -qE "^Alumno: .*" entregas.txt; then
    echo "Formato de entrega válido."
fi
```

### 2. `sed` (Stream Editor)

```bash
# Reemplazar la primera ocurrencia de "http" por "https" en cada línea
sed 's/http/https/' lista_urls.txt

# Reemplazar TODAS las ocurrencias ('g' = global) e imponer el cambio directamente en el archivo ('-i')
sed -i 's/DB_PORT=5432/DB_PORT=5433/g' .env

# Eliminar líneas que coinciden con un patrón
sed -i '/^DEBUG/d' app.log

# Truco de intercambio de cadenas (Swap con variable auxiliar)
sed -e 's/Gonzalo Martinez/_aux_/g' \
    -e 's/Nicolas Riedel/Gonzalo Martinez/g' \
    -e 's/_aux_/Nicolas Riedel/g' lista.txt
```

### 3. `awk` (Procesamiento de columnas y reportes)

```bash
# Imprimir solo la primera ($1) y tercera ($3) columna de un archivo separado por comas (-F',')
awk -F',' '{print $1, $3}' usuarios.csv

# Filtrar líneas donde la columna 3 sea mayor a 100 y sumar el total de la columna 2
awk -F',' '$3 > 100 {suma += $2} END {print "Suma total:", suma}' ventas.csv
```
