# Módulo 05: Bases de datos relacionales, normalización y SQL

Guía de estudio conceptual y técnica completa sobre el **modelo relacional, normalización de bases de datos (1NF-3NF), lenguaje SQL (DDL, DML, DQL), JOINs avanzados y transacciones ACID**.

---

## 1. El modelo relacional y diseño E-R

El modelo relacional, formulado por **Edgar F. Codd**, organiza la información en colecciones de tablas bidimensionales matemáticamente denominadas **relaciones**.

```
Relación / Tabla: "usuarios"
┌────┬──────────────────┬──────────────────────┬─────────────┐
│ id │ nombre           │ email                │ creado_en   │ ── Atributos (Columnas)
├────┼──────────────────┼──────────────────────┼─────────────┤
│ 1  │ Gonzalo Martínez │ gmartinez@gmail.com  │ 2026-01-15  │ ── Tupla (Fila / Registro)
│ 2  │ Nicolás Riedel   │ nriedel@gmail.com    │ 2026-02-20  │
└────┴──────────────────┴──────────────────────┴─────────────┘
  ▲
Primary Key (PK)
```

### Conceptos clave de arquitectura relacional
- **Primary Key (PK - Clave primaria)**: Atributo o combinación de atributos que identifica unívocamente a cada registro. No admite valores nulos (`NOT NULL`) ni duplicados (`UNIQUE`).
  - *Clave subrogada*: Identificador numérico autoincremental (`SERIAL` / `AUTO_INCREMENT`) o identificador único universal (`UUID`).
  - *Clave natural*: Atributo propio del dominio de negocio (ej. CUIT, DNI, ISBN).
- **Foreign Key (FK - Clave foránea)**: Atributo que almacena el valor de la clave primaria de otra tabla, estableciendo un vínculo relacional entre ambas.
- **Integridad referencial**: Garantía lógica impuesta por el motor RDBMS de que ninguna clave foránea puede hacer referencia a un registro inexistente en la tabla padre.
  - Comportamiento al eliminar (`ON DELETE`): `CASCADE` (elimina registros dependientes), `SET NULL` (asigna nulo a los hijos), `RESTRICT` / `NO ACTION` (bloquea la eliminación si existen dependencias).

### Tipos de relaciones (Cardinalidad)
1. **Relación $1:1$ (Uno a uno)**: Un registro de la Tabla A se relaciona con como máximo un registro de la Tabla B.
2. **Relación $1:N$ (Uno a muchos)**: Un registro de la Tabla A se relaciona con múltiples registros de la Tabla B. La FK se ubica siempre en el lado $N$.
3. **Relación $N:M$ (Muchos a muchos)**: Múltiples registros de la Tabla A se relacionan con múltiples registros de la Tabla B. Requiere una **tabla intermedia / tabla de unión** con las claves foráneas de ambas tablas.

---

## 2. Normalización de bases de datos (1NF a 3NF)

La **Normalización** es el proceso formal de simplificación y estructuración de un diseño relacional para **eliminar redundancias de datos** y **prevenir anomalías de inserción, actualización y borrado**.

### 1ra Forma Normal (1NF): Atomicidad
- **Regla**: Todos los atributos de una tabla deben contener **valores atómicos** (indivisibles). No se permiten listas, arreglos ni grupos repetitivos dentro de una sola celda.
- *Corrección*: Si un usuario tiene múltiples teléfonos, se debe mover el teléfono a una tabla independiente vinculada por FK.

### 2da Forma Normal (2NF): Dependencia funcional completa
- **Regla**: El diseño debe cumplir 1NF y **todos los atributos no clave deben depender totalmente de la clave primaria completa**, no de una parte de ella (aplica a claves primarias compuestas).
- *Corrección*: Separar los datos que dependen solo de una parte de la clave en una tabla nueva.

### 3ra Forma Normal (3NF): Eliminación de dependencias transitivas
- **Regla**: El diseño debe cumplir 2NF y **ningún atributo no clave debe depender funcionalmente de otro atributo no clave**.
- *Ejemplo de violación*: Una tabla `empleados(id, nombre, departamento_id, nombre_departamento)`. El `nombre_departamento` depende de `departamento_id`, no directamente de `id` (empleado).
- *Corrección*: Mover `departamento_id` y `nombre_departamento` a una tabla independiente `departamentos`.

---

## 3. Sublenguajes de SQL: DDL, DML, DQL y TCL

El lenguaje **SQL (Structured Query Language)** se categoriza según el propósito de sus sentencias:

### Data Definition Language (DDL)
Define, modifica y destruye la estructura de esquemas y objetos de la BD:

```sql
-- Crear tabla con restricciones completas
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    edad INT CHECK (edad >= 18),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Modificar estructura agregando columna
ALTER TABLE usuarios ADD COLUMN rol VARCHAR(20) DEFAULT 'USER';

-- Eliminar tabla y su contenido de forma definitiva
DROP TABLE usuarios;
```

### Data Manipulation Language (DML)
Permite insertar, actualizar y borrar información almacenada:

```sql
-- Inserción de datos
INSERT INTO usuarios (nombre, email, edad) 
VALUES ('Gonzalo Martinez', 'gmartinez@gmail.com', 25);

-- Actualización condicional
UPDATE usuarios 
SET rol = 'ADMIN' 
WHERE email = 'gmartinez@gmail.com';

-- Borrado condicional
DELETE FROM usuarios 
WHERE edad < 18;
```

---

## 4. Estructura y orden de ejecución lógica en DQL (`SELECT`)

La escritura de consultas difiere significativamente del orden en que el motor evaluador procesa internamente la información:

```
ORDEN DE ESCRITURA (Sintaxis SQL)       ORDEN DE EJECUCIÓN LÓGICA (Motor RDBMS)
 1. SELECT columnas                      1. FROM & JOIN (Carga y unión de tablas)
 2. FROM tabla                           2. WHERE (Filtro de filas individuales)
 3. JOIN tabla_b ON condicion            3. GROUP BY (Agrupamiento de filas)
 4. WHERE condicion_filas                4. HAVING (Filtro de grupos agrupados)
 5. GROUP BY columnas_agrupamiento       5. SELECT (Proyección de expresiones)
 6. HAVING condicion_grupos              6. DISTINCT (Eliminación de duplicados)
 7. ORDER BY columna ASC|DESC            7. ORDER BY (Ordenamiento del resultado)
 8. LIMIT cantidad                       8. LIMIT / OFFSET (Recorte de filas)
```

> ⚠️ **Diferencia fundamental de examen**: `WHERE` filtra filas individuales **antes** de realizar cualquier aglomeración. `HAVING` filtra grupos **después** de haber aplicado funciones de agregación (`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`).

---

## 5. Tipos de uniones (`JOIN`s) entre tablas

```
       INNER JOIN                LEFT JOIN               RIGHT JOIN             FULL OUTER JOIN
    ┌───────┬───────┐        ┌───────┬───────┐       ┌───────┬───────┐       ┌───────┬───────┐
    │   A   │   B   │        │   A   │   B   │       │   A   │   B   │       │   A   │   B   │
    │     █████     │        │ █████████     │       │     █████████ │       │ █████████████ │
    └───────┴───────┘        └───────┴───────┘       └───────┴───────┘       └───────┴───────┘
  Coincidencias en ambas.     Todo A + coinc. B.      Todo B + coinc. A.      Todo A + todo B.
```

### Ejemplos sintácticos

#### `INNER JOIN`
```sql
-- Obtener pedidos con los nombres de sus usuarios correspondientes
SELECT p.id AS pedido_id, u.nombre AS usuario, p.monto
FROM pedidos p
INNER JOIN usuarios u ON p.usuario_id = u.id;
```

#### `LEFT JOIN`
```sql
-- Obtener TODOS los usuarios y sus pedidos (incluyendo usuarios sin pedidos)
SELECT u.nombre, p.id AS pedido_id
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id;
```

---

## 6. Subconsultas y subconsultas correlacionadas

### Subconsulta en cláusula `WHERE` (`IN` / `NOT IN`)
```sql
-- Seleccionar usuarios que realizaron pedidos superiores a $500
SELECT nombre, email 
FROM usuarios 
WHERE id IN (
    SELECT usuario_id 
    FROM pedidos 
    WHERE monto > 500
);
```

### Subconsulta correlacionada (`EXISTS`)
Una subconsulta correlacionada depende de valores expuestos por la consulta externa, evaluándose para cada fila procesada:

```sql
-- Obtener usuarios que NO poseen ningún pedido registrado
SELECT u.nombre 
FROM usuarios u
WHERE NOT EXISTS (
    SELECT 1 
    FROM pedidos p 
    WHERE p.usuario_id = u.id
);
```

---

## 7. Transacciones y propiedades ACID

Una **Transacción** es una unidad lógica de trabajo que agrupa múltiples sentencias SQL que deben ejecutarse como una sola entidad indivisible.

```sql
BEGIN TRANSACTION;

UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

COMMIT; -- Se confirman los cambios si todo tuvo éxito
-- En caso de error se ejecuta: ROLLBACK;
```

### Propiedades ACID
- **Atomicidad (Atomicity)**: Principio de "todo o nada". Si una sola instrucción dentro de la transacción falla, la transacción entera se deshace (`ROLLBACK`).
- **Consistencia (Consistency)**: La base de datos pasa de un estado válido a otro estado válido, preservando todas las restricciones de integridad y reglas.
- **Aislamiento (Isolation)**: Las transacciones concurrentes se ejecutan de manera aislada sin interferir entre sí antes de confirmarse.
- **Durabilidad (Durability)**: Una vez confirmada una transacción (`COMMIT`), los cambios persisten de manera permanente en disco, incluso ante caídas de energía o fallos del servidor.
