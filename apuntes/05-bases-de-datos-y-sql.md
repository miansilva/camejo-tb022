# Módulo 05: Bases de datos relacionales y SQL (Teoría y práctica)

Guía de estudio completa para comprender el diseño de **Bases de datos relacionales (RDBMS)**, las propiedades del modelo entidad-relación y el dominio del lenguaje **SQL (Structured Query Language)**.

---

## 1. Fundamentos del modelo relacional

El modelo relacional, propuesto por **Edgar F. Codd** en 1970, organiza los datos en tablas matemáticas denominadas **relaciones**.

```
Tabla / Relación: "bandas"
┌────┬──────────────┬──────────────┬────────────────┬─────────────────┐
│ id │ nombre       │ pais_origen  │ fecha_creacion │ cant_integrantes│ ── Atributo / Columna
├────┼──────────────┼──────────────┼────────────────┼─────────────────┤
│ 1  │ Queen        │ Inglaterra   │ 1970           │ 4               │ ── Tupla / Fila
│ 2  │ Soda Stereo  │ Argentina    │ 1982           │ 3               │
└────┴──────────────┴──────────────┴────────────────┴─────────────────┘
  ▲
Primary Key (PK)
```

### Conceptos clave

- **Primary Key (PK - Clave primaria)**: Atributo o conjunto de atributos que identifica de manera única e inequívoca a cada fila dentro de una tabla. No puede contener valores nulos (`NULL`) ni duplicados.
  - *Clave subrogada*: Un identificador numérico artificial autoincremental (`SERIAL` / `AUTO_INCREMENT`) o `UUID`.
  - *Clave natural*: Un atributo propio del dominio (ej. DNI, CUIT, ISBN).
- **Foreign Key (FK - Clave foránea)**: Atributo en una tabla que hace referencia a la Primary Key de otra tabla, creando una relación entre ambas.
- **Integridad referencial**: Regla del RDBMS que garantiza que nunca existan claves foráneas apuntando a registros inexistentes.
  - Acciones ante borrado (`ON DELETE`): `CASCADE` (borra en cascada), `SET NULL` (coloca nulo), `RESTRICT` / `NO ACTION` (impide el borrado si hay hijos).

### Tipos de relaciones (Cardinalidad)

1. **Relación $1:1$ (Uno a uno)**: Un registro de la Tabla A se relaciona con como máximo un registro de la Tabla B.
2. **Relación $1:N$ (Uno a muchos)**: Un registro de la Tabla A se relaciona con múltiples registros de la Tabla B (ej. Una `banda` tiene muchos `albumes`). La FK se ubica en el lado $N$ (`albumes`).
3. **Relación $N:M$ (Muchos a muchos)**: Múltiples registros de la Tabla A se relacionan con múltiples registros de la Tabla B (ej. Un `concierto` tiene muchas `bandas` y una `banda` participa en muchos `conciertos`).
   - **Solución**: Requiere crear una **Tabla intermedia / Tabla de unión** (`conciertos_musicos`) con las claves foráneas de ambas tablas.

---

## 2. Clasificación del lenguaje SQL

SQL se divide en sublenguajes según el tipo de operación:

```
┌─────────────────────────────────────────────────────────────┐
│                       LENGUAJE SQL                          │
├──────────────┬──────────────┬──────────────┬────────────────┤
│     DDL      │     DML      │     DQL      │      TCL       │
│ Data Def.    │ Data Manip.  │ Data Query   │ Trans. Control │
├──────────────┼──────────────┼──────────────┼────────────────┤
│ CREATE       │ INSERT       │ SELECT       │ COMMIT         │
│ ALTER        │ UPDATE       │              │ ROLLBACK       │
│ DROP         │ DELETE       │              │ SAVEPOINT      │
│ TRUNCATE     │              │              │                │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

---

## 3. Orden de escritura vs orden de ejecución lógica en DQL

Al escribir una consulta SQL, la sintaxis exige un orden determinado, pero el motor de la base de datos ejecuta las cláusulas en un orden interno diferente:

### Orden de escritura (Sintaxis)
```sql
SELECT columnas
FROM tabla_principal
JOIN tabla_secundaria ON condicion_join
WHERE condicion_filtro_filas
GROUP BY columnas_agrupamiento
HAVING condicion_filtro_grupos
ORDER BY columna_ordenamiento ASC|DESC
LIMIT cantidad;
```

### Orden de ejecución lógica interna de la BD
1. **`FROM` & `JOIN`**: Se ubican las tablas y se realizan las uniones de datos.
2. **`WHERE`**: Se filtran las filas individuales que no cumplen la condición.
3. **`GROUP BY`**: Se agrupan las filas restantes según los atributos indicados.
4. **`HAVING`**: Se filtran los grupos creados (no las filas individuales).
5. **`SELECT`**: Se evalúan las expresiones y se proyectan las columnas finales.
6. **`DISTINCT`**: Se eliminan filas duplicadas del resultado.
7. **`ORDER BY`**: Se ordena el conjunto de resultados final.
8. **`LIMIT` / `OFFSET`**: Se recorta la cantidad de filas a retornar.

> ⚠️ **Diferencia crítica en exámenes**: `WHERE` filtra filas **antes** de agrupar. `HAVING` filtra grupos **después** de aplicar funciones de agregación (`COUNT`, `SUM`, `AVG`).

---

## 4. Tipos de uniones (`JOIN`s) entre tablas

```
INNER JOIN              LEFT JOIN               RIGHT JOIN              FULL OUTER JOIN
  ┌───┬───┐               ┌───┬───┐               ┌───┬───┐               ┌───┬───┐
  │ A │ B │               │ A │ B │               │ A │ B │               │ A │ B │
  │  ███  │               │ █████ │               │  ████ │               │ █████ │
  └───┴───┘               └───┴───┘               └───┴───┘               └───┴───┘
Coincidencias            Todo A +                Todo B +                Todo A y
en ambas tablas.         coincidencias B.        coincidencias A.        todo B.
```

### Ejemplos sintácticos

#### `INNER JOIN` (Intersección)
Retorna únicamente los registros que tienen coincidencias exactas en ambas tablas.
```sql
SELECT a.nombre AS album, b.nombre AS banda
FROM albumes a
INNER JOIN bandas b ON a.banda_id = b.id;
```

#### `LEFT JOIN` (Inclusión izquierda)
Retorna todos los registros de la tabla izquierda (`bandas`), y si no hay coincidencia en la derecha (`conciertos`), llena esos campos con `NULL`.
```sql
SELECT b.nombre AS banda, c.nombre AS concierto
FROM bandas b
LEFT JOIN conciertos_musicos cm ON cm.banda_id = b.id
LEFT JOIN conciertos c ON cm.concierto_id = c.id;
```

---

## 5. Subconsultas y operadores de conjuntos

### 1. Subconsulta con `IN` / `NOT IN`
```sql
-- Obtener álbumes de bandas que sean originarias de Argentina
SELECT nombre 
FROM albumes 
WHERE banda_id IN (
    SELECT id 
    FROM bandas 
    WHERE pais_origen = 'Argentina'
);
```

### 2. Subconsulta correlacionada con `EXISTS` / `NOT EXISTS`
Una subconsulta correlacionada se ejecuta una vez por cada fila procesada por la consulta externa.

```sql
-- Obtener las bandas que NO tienen ningún álbum registrado
SELECT b.nombre 
FROM bandas b
WHERE NOT EXISTS (
    SELECT 1 
    FROM albumes a 
    WHERE a.banda_id = b.id
);
```
