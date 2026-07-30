# Módulo 08: Programación web con JavaScript (ES6+), DOM y asincronía

Guía de estudio conceptual y técnica completa sobre **JavaScript (ES6+)**, arquitectura interna del motor V8, el **Event loop**, asincronía (`Promises` y `async/await`), manipulación del DOM y consumo de APIs mediante `fetch()`.

---

## 1. Arquitectura interna del motor de JavaScript (V8) y ejecución

JavaScript es un lenguaje de programación dinámico, débilmente tipado y multiparadigma. Se ejecuta mediante un motor de análisis y compilación en tiempo de ejecución (*Just-In-Time Compilation - JIT*), como el V8 de Google.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MOTOR JAVASCRIPT (V8)                               │
├──────────────────────────────────────┬──────────────────────────────────────┤
│            MEMORY HEAP               │              CALL STACK              │
│   (Memoria no estructurada donde     │     (Pila de ejecución LIFO donde    │
│    se asignan objetos y variables)   │      se apilan las funciones)        │
└──────────────────────────────────────┴──────────────────────────────────────┘
```

### Componentes de memoria
1. **Memory Heap (Heap de memoria)**: Región de memoria no estructurada donde el motor reserva espacio para objetos, arrays, cierres (*closures*) y funciones.
2. **Call Stack (Pila de llamadas)**: Estructura LIFO (*Last In, First Out*) que rastrea el contexto de ejecución. Cada llamada a una función agrega un marco de pila (*Stack Frame*), el cual se destruye al retornar.
3. **Recolección de basura (Garbage Collector)**: Implementa el algoritmo **Mark-and-Sweep**. Parte de la raíz global (`window` o `global`), marca todos los objetos alcanzables y libera de la memoria los objetos huérfanos que carecen de referencias.

---

## 2. Alcance (scope), hoisting y clausuras (closures)

### Tipos de alcance (*scope*)
- **Global Scope**: Variables declaradas fuera de cualquier bloque o función. Accesibles desde cualquier punto del programa.
- **Function Scope**: Variables declaradas con la palabra clave `var`. Se restringen al cuerpo de la función contenedora.
- **Block Scope**: Variables declaradas con `let` y `const`. Tienen alcance estricto limitado al bloque envuelto por llaves `{ ... }`.

### Elevación (*hoisting*)
Proceso por el cual el motor mueve mentalmente las declaraciones al inicio de su alcance en la fase de compilación:
- `var`: Se eleva e inicializa implícitamente como `undefined`.
- `let` y `const`: Se elevan pero permanecen inaccesibles dentro de la **Zona muerta temporal (Temporal Dead Zone - TDZ)** hasta que se alcanza su declaración explícita.

### Clausuras (*closures*)
Un **Closure** es la combinación de una función y el entorno léxico (*Lexical Environment*) en el que fue creada. Permite que una función interna conserve acceso a las variables scope de su función contenedora padre, incluso después de que la función padre haya retornado.

```javascript
// Patrón de encapsulamiento de estado privado mediante un Closure
function crearContador() {
    let contador = 0; // Variable privada no accesible desde fuera
    
    return {
        incrementar: () => ++contador,
        obtenerValor: () => contador
    };
}

const miContador = crearContador();
console.log(miContador.incrementar()); // Salida: 1
console.log(miContador.incrementar()); // Salida: 2
console.log(miContador.obtenerValor()); // Salida: 2
// miContador.contador -> undefined (Protegido contra mutaciones no autorizadas)
```

---

## 3. Variables, tipos de datos y coerción de tipos

### `var` vs. `let` vs. `const`

| Característica | `var` | `let` | `const` |
| :--- | :---: | :---: | :---: |
| **Alcance** | Función | Bloque | Bloque |
| **Reasignación** | Permitida | Permitida | **Inadmisible** (Constante) |
| **Redeclaración** | Permitida | Inadmisible | Inadmisible |
| **Hoisting** | `undefined` | TDZ (Error) | TDZ (Error) |

### Tipos primitivos vs. referencia
- **Primitivos (Inmutables)**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`. Se comparan y copian por **valor**.
- **Referencia (Mutables)**: `Object`, `Array`, `Function`, `Map`, `Set`. Se comparan y copian por **dirección de memoria**.

```javascript
// Comparación estricta (===) vs Coerción débil (==)
5 == "5";     // true (Coerción implícita de tipos - EVITAR)
5 === "5";    // false (Comparación estricta de tipo y valor - RECOMENDADO)
```

---

## 4. Sintaxis moderna en ES6+ y métodos de arrays

### Arrow Functions y operadores Spread/Rest
```javascript
// Arrow function con retorno implícito de una sola línea
const multiplicar = (a, b) => a * b;

// Rest Parameters (...args): Agrupa argumentos variables en un array
const sumarTodo = (...numeros) => numeros.reduce((acc, n) => acc + n, 0);

// Spread Operator (...arr): Desestructura un array u objeto
const base = [1, 2];
const extendido = [...base, 3, 4]; // [1, 2, 3, 4]
```

### Métodos iterativos esenciales de arrays
```javascript
const productos = [
    { id: 1, nombre: "Teclado", precio: 50, activo: true },
    { id: 2, nombre: "Mouse", precio: 25, activo: false },
    { id: 3, nombre: "Monitor", precio: 200, activo: true }
];

# 1. map: Transforma cada elemento retornando un nuevo array de igual tamaño
const nombres = productos.map(p => p.nombre);

# 2. filter: Retorna un nuevo array con los elementos que satisfacen la condición
const activos = productos.filter(p => p.activo);

# 3. find: Devuelve el primer elemento que cumple la condición
const item = productos.find(p => p.id === 2);

# 4. reduce: Acumula los elementos en un único valor final
const totalPrecios = productos.reduce((acc, p) => acc + p.precio, 0);
```

---

## 5. Arquitectura asíncrona y el Event loop

JavaScript es un lenguaje **mono-hilo (Single-Threaded)** con un modelo de I/O no bloqueante. Delegará operaciones lentas (peticiones de red, lectura de archivos, temporizadores) al entorno de ejecución (*Web APIs* en navegador o *Node.js Libuv*).

```
┌────────────────────────┐      ┌────────────────────────┐
│       CALL STACK       │      │   Web APIs / Node Libuv│
│  (Ejecución síncrona)  │      │ (fetch, setTimeout, FS)│
└───────────┬────────────┘      └───────────┬────────────┘
            │                               │ Al finalizar
            ▼                               ▼
┌────────────────────────┐      ┌────────────────────────┐
│    MICROTASK QUEUE     │      │ MACROTASK / CALLBACK Q.│
│ (Callbacks de Promesas)│      │  (setTimeout, I/O)     │
└───────────┬────────────┘      └───────────┬────────────┘
            │                               │
            └──────────► EVENT LOOP ◄───────┘
               (Pasa tareas al Call Stack vacío)
```

1. **Call Stack**: Ejecuta todo el código síncrono.
2. **Web APIs / Node APIs**: Procesan en segundo plano operaciones asíncronas.
3. **Microtask Queue**: Cola prioritaria donde se encolan las llamadas de Promesas (`.then()`, `async/await`).
4. **Macrotask Queue**: Cola para temporizadores (`setTimeout`) e I/O.
5. **Event Loop**: Revisa constantemente el Call Stack. Cuando está **completamente vacío**, vacía primero la **Microtask Queue** y luego toma elementos de la **Macrotask Queue**.

---

## 6. Manejo de asincronía: `Promises` y `async/await`

### Promesas (`Promise`)
Objeto que representa el resultado futuro de una operación asíncrona (Estados: *Pending*, *Fulfilled*, *Rejected*).

```javascript
const obtenerDatos = () => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const exito = true;
            if (exito) resolve({ data: "Payload recibido" });
            else reject(new Error("Fallo de comunicación"));
        }, 1000);
    });
};

obtenerDatos()
    .then(respuesta => console.log(respuesta.data))
    .catch(error => console.error(error.message))
    .finally(() => console.log("Operación terminada"));
```

### `async` / `await` y manejo de errores defensivo
Sintaxis síncrona para trabajar con Promesas de forma limpia mediante `try/catch`:

```javascript
async function procesarSolicitud() {
    try {
        const respuesta = await obtenerDatos();
        console.log("Éxito:", respuesta.data);
    } catch (error) {
        console.error("Excepción capturada:", error.message);
    }
}
```

---

## 7. Manipulación del DOM y peticiones asíncronas (`fetch`)

### Selección de elementos y eventos en el navegador
```javascript
// Selección de nodo en el árbol DOM
const botonEnviar = document.querySelector('#btnEnviar');
const inputEmail = document.querySelector('.input-email');

// Delegación de evento click
botonEnviar.addEventListener('click', async (event) => {
    event.preventDefault(); // Previene la recarga del formulario
    const email = inputEmail.value;
    await enviarAlServidor(email);
});
```

### Consumo de APIs REST mediante `fetch()`
```javascript
async function enviarAlServidor(email) {
    try {
        const respuesta = await fetch('https://api.ejemplo.com/v1/usuarios', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ email })
        });

        if (!respuesta.ok) {
            throw new Error(`Error en la petición: ${respuesta.status}`);
        }

        const datos = await respuesta.json();
        console.log("Usuario registrado con éxito:", datos);
    } catch (error) {
        console.error("Error de red:", error.message);
    }
}
```
