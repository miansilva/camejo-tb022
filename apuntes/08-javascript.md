# Módulo 08: JavaScript - Fundamentos, asincronía y desarrollo web

Guía de estudio completa para dominar **JavaScript (JS / ECMAScript)**, el lenguaje de programación fundamental para el desarrollo web (Front-end en navegador y Back-end con Node.js), abarcando sintaxis, estructuras de datos, programación funcional, orientado a objetos, asincronía y manipulación del DOM.

---

## 1. Arquitectura del motor de JavaScript (V8) y ejecución

JavaScript es un lenguaje interpretado y compilado en tiempo de ejecución (*Just-In-Time Compilation - JIT*).

```
┌─────────────────────────────────────────────────────────────┐
│                    MOTOR JAVASCRIPT (V8)                    │
├──────────────────────────────┬──────────────────────────────┤
│        MEMORY HEAP           │          CALL STACK          │
│   (Memoria no estructurada   │    (Pila de ejecución LIFO   │
│    donde se almacenan        │    donde se gestionan los    │
│    objetos y funciones)      │    marcos de contexto)       │
└──────────────────────────────┴──────────────────────────────┘
```

### 1. Memory Heap (Memoria principal)
Espacio de memoria no estructurado donde el motor asigna espacio para objetos, arreglos y funciones.

### 2. Call Stack (Pila de ejecución)
Estructura de datos LIFO (*Last In, First Out*) que rastrea la llamada a funciones. Cuando se invoca una función, se agrega un marco de pila (*Stack Frame*); al retornar, se destruye.

### 3. Recolección de basura (Garbage collection)
El motor aplica el algoritmo **Mark-and-Sweep**:
1. Parte de las raíces globales (`window` o `global`).
2. Recorre las referencias a objetos en memoria marcando los que son alcanzables.
3. Libera la memoria de los objetos no alcanzables (*unreachable*).

---

## 2. Alcance (Scope), entorno léxico y clausuras (Closures)

### Tipos de alcance (Scope)

- **Alcance global (*Global Scope*)**: Variables declaradas fuera de cualquier función o bloque. Accesibles desde todo el programa.
- **Alcance de función (*Function Scope*)**: Variables declaradas con `var` dentro de una función.
- **Alcance de bloque (*Block Scope*)**: Variables declaradas con `let` y `const` dentro de llaves `{ ... }`.

### Elevación (Hoisting)
Mecanismo por el cual el motor mueve las declaraciones de variables y funciones al inicio de su alcance durante la fase de compilación.
- `var`: Se eleva e inicializa como `undefined`.
- `let` y `const`: Se elevan pero permanecen en la **Zona muerta temporal (*Temporal Dead Zone - TDZ*)** hasta que se ejecuta su línea de inicialización.

### Clausuras (Closures)
Una **clausura** es la combinación de una función y el entorno léxico en el cual fue declarada. Permite a una función interna acceder a las variables de su función padre incluso después de que la función padre haya finalizado su ejecución.

```javascript
// Ejemplo de encapsulamiento de datos mediante Closure
function crearContador() {
    let contador = 0; // Variable privada encerrada en el closure
    
    return {
        incrementar: function() {
            contador++;
            return contador;
        },
        obtenerValor: function() {
            return contador;
        }
    };
}

const miContador = crearContador();
console.log(miContador.incrementar()); // 1
console.log(miContador.incrementar()); // 2
console.log(miContador.obtenerValor()); // 2
// miContador.contador -> undefined (Protegido contra modificación externa)
```

---

## 3. Variables, tipos de datos y operadores

### Variables: `var` vs `let` vs `const`

| Criterio | `var` | `let` | `const` |
| :--- | :---: | :---: | :---: |
| **Alcance** | Función | Bloque | Bloque |
| **Reasignación** | Sí | Sí | **No** |
| **Redeclaración** | Sí | No | No |
| **Hoisting** | `undefined` | TDZ (Error) | TDZ (Error) |

### Tipos de datos en JavaScript

#### Primitivos (Inmutables, comparación por valor)
- **`string`**: Cadenas de texto.
- **`number`**: Números de punto flotante de 64 bits (IEEE 754).
- **`boolean`**: `true` o `false`.
- **`null`**: Ausencia explícita de valor.
- **`undefined`**: Variable declarada sin valor asignado.
- **`symbol`**: Identificador único global.
- **`bigint`**: Enteros de precisión arbitraria (`100n`).

#### De referencia (Mutables, comparación por dirección de memoria)
- **`Object`**, **`Array`**, **`Function`**, **`Date`**, **`Map`**, **`Set`**.

### Comparación estricta vs conversión implícita

```javascript
// Conversión implícita de tipos (Coerción débil) - EVITAR
5 == "5";     // true
0 == false;   // true
null == undefined; // true

// Comparación estricta (Mismo tipo y mismo valor) - USAR SIEMPRE
5 === "5";    // false
5 === 5;      // true
```

---

## 4. Funciones y sintaxis moderna (ES6+)

### 1. Declaración de funciones
```javascript
function calcularArea(ancho, alto) {
    return ancho * alto;
}
```

### 2. Funciones flecha (*Arrow Functions*)
Sintaxis concisa que no posee su propio binding de `this`, `arguments` ni `super`.

```javascript
// Sintaxis completa
const calcularArea = (ancho, alto) => {
    return ancho * alto;
};

// Retorno implícito de una sola línea
const alCuadrado = x => x * x;
```

### 3. Parámetros por defecto, Rest y Spread operator

```javascript
// Parámetros por defecto
const conectarDB = (puerto = 5432, host = "localhost") => {
    console.log(`Conectando a ${host}:${puerto}`);
};

// Rest Parameters (...args): Convierte argumentos sueltos en un array
const sumarArgumentos = (...numeros) => {
    return numeros.reduce((acum, n) => acum + n, 0);
};

// Spread Operator (...arr): Desestructura un iterable en elementos individuales
const base = [1, 2];
const combinado = [...base, 3, 4]; // [1, 2, 3, 4]
```

---

## 5. Estructuras de datos: Objetos, Arrays y Colecciones

### Desestructuración de objetos y arrays
```javascript
const estudiante = {
    nombre: "Silva",
    curso: "75.18",
    calificaciones: [9, 10, 8]
};

// Desestructuración de objeto y alias
const { nombre, curso: materia } = estudiante;

// Desestructuración de array
const [primeraNota, segundaNota] = estudiante.calificaciones;
```

### Métodos iterativos de Arrays
```javascript
const productos = [
    { id: 1, nombre: "Laptop", precio: 1200, activo: true },
    { id: 2, nombre: "Mouse", precio: 25, activo: false },
    { id: 3, nombre: "Teclado", precio: 80, activo: true }
];

// map: Transforma cada elemento creando un nuevo array
const nombres = productos.map(p => p.nombre);

// filter: Filtra los elementos que cumplen la condición
const activos = productos.filter(p => p.activo);

// find: Retorna el primer elemento coincidente
const laptop = productos.find(p => p.id === 1);

// reduce: Acumula el resultado de procesar el array
const totalPrecio = productos.reduce((total, p) => total + p.precio, 0);

// some / every: Verificaciones booleanas
const hayCaros = productos.some(p => p.precio > 1000); // true
const todosActivos = productos.every(p => p.activo);    // false
```

### Colecciones de ES6 (`Map` y `Set`)
- **`Set`**: Colección de valores únicos sin duplicados.
- **`Map`**: Colección de pares clave-valor donde la clave puede ser de cualquier tipo (objetos, funciones, etc.).

```bash
// Eliminar duplicados de un array con Set
const duplicados = [1, 2, 2, 3, 4, 4, 5];
const unicos = [...new Set(duplicados)]; // [1, 2, 3, 4, 5]
```

---

## 6. Prototipos y clases (Programación orientada a objetos)

JavaScript utiliza **herencia basada en prototipos**. Cada objeto tiene una propiedad interna que enlaza a otro objeto denominado su prototipo (`__proto__`).

### Clases en ES6 (Azúcar sintáctico sobre prototipos)

```javascript
class Persona {
    constructor(nombre, edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    presentarse() {
        return `Hola, soy ${this.nombre} y tengo ${this.edad} años.`;
    }
}

// Herencia de clases
class Estudiante extends Persona {
    constructor(nombre, edad, padron) {
        super(nombre, edad); // Invocación al constructor de la clase padre
        this.padron = padron;
    }

    estudiar() {
        return `${this.nombre} está estudiando para el examen de la materia.`;
    }
}

const estudiante1 = new Estudiante("Silva", 22, "102345");
console.log(estudiante1.presentarse());
console.log(estudiante1.estudiar());
```

---

## 7. Asincronía y el Event loop

 JavaScript es mono-hilo. La asincronía se gestiona mediante el **Event loop**, delegando operaciones pesadas a las APIs del navegador o del sistema operativo.

### Modelo de ejecución asíncrona

```
1. Call Stack ejecuta el código síncrono.
2. Las tareas asíncronas (fetch, setTimeout) se envían a Web APIs.
3. Al finalizar, las callbacks de Promesas van a la Microtask Queue.
4. Las callbacks de timers van a la Macrotask / Callback Queue.
5. El Event Loop pasa primero las Microtasks y luego las Macrotasks al Call Stack cuando está vacío.
```

### Promesas (`Promise`)

Objeto que representa un valor que estará disponible ahora, en el futuro o nunca.

```javascript
const consultarAPI = () => {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const exito = true;
            if (exito) {
                resolve({ status: 200, payload: "Datos recibidos" });
            } else {
                reject(new Error("Error de servidor"));
            }
        }, 1000);
    });
};

consultarAPI()
    .then(res => console.log(res.payload))
    .catch(err => console.error(err.message))
    .finally(() => console.log("Finalizado"));
```

### Múltiples promesas simultáneas
- **`Promise.all([p1, p2])`**: Espera a que todas se resuelvan. Si una falla, rechaza inmediatamente.
- **`Promise.allSettled([p1, p2])`**: Espera a que todas finalicen (resueltas o rechazadas).
- **`Promise.race([p1, p2])`**: Retorna el resultado de la primera promesa en finalizar.

### Async / Await

Sintaxis síncrona sobre promesas para mejorar la legibilidad.

```javascript
async function obtenerDatosProcesados() {
    try {
        const respuesta = await consultarAPI();
        console.log("Respuesta recibida:", respuesta);
    } catch (error) {
        console.error("Manejo de excepción:", error.message);
    }
}
```

---

## 8. Módulos en JavaScript: CommonJS vs ES Modules

### 1. CommonJS (`require` / `module.exports`)
Estándar tradicional de Node.js (carga síncrona).

```javascript
// exportar.js
module.exports = { sumar, restar };

// importar.js
const { sumar, restar } = require('./exportar');
```

### 2. ES Modules (`import` / `export`)
Estándar oficial de JavaScript moderno (ES6, soporta carga asíncrona nativa).

```javascript
// math.js
export const sumar = (a, b) => a + b;
export default class Calculadora {}

// app.js
import Calculadora, { sumar } from './math.js';
```

---

## 9. Control de errores y excepciones

El manejo adecuado de errores evita que la aplicación se caiga de forma inesperada.

```javascript
class ValidacionError extends Error {
    constructor(mensaje) {
        super(mensaje);
        this.name = "ValidacionError";
    }
}

function procesarUsuario(usuario) {
    if (!usuario.nombre) {
        throw new ValidacionError("El nombre de usuario es obligatorio.");
    }
    return true;
}

try {
    procesarUsuario({});
} catch (error) {
    if (error instanceof ValidacionError) {
        console.warn("Error de validación:", error.message);
    } else {
        console.error("Error inesperado:", error);
    }
} finally {
    console.log("Limpieza de recursos finalizada.");
}
```

---

## 10. Manipulación del DOM y Fetch API (Entorno navegador)

### Selección y modificación de elementos
```javascript
// Selección de elementos
const elementoTitulo = document.querySelector('#tituloPrincipal');
const listaItems = document.querySelectorAll('.item-lista');

// Modificación de propiedades y clases CSS
elementoTitulo.textContent = "Apuntes de desarrollo de software";
elementoTitulo.classList.add('activo');
```

### Manejo de eventos
```javascript
const formulario = document.querySelector('#formLogin');

formulario.addEventListener('submit', async (event) => {
    event.preventDefault(); // Evita la recarga de página por defecto
    
    const email = document.querySelector('#emailInput').value;
    console.log("Enviando formulario para:", email);
});
```

### Peticiones HTTP con Fetch API
```javascript
async function cargarDatosRemotos() {
    try {
        const respuesta = await fetch('https://api.example.com/v1/datos', {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer TOKEN_AQUI'
            }
        });

        if (!respuesta.ok) {
            throw new Error(`Error HTTP Status: ${respuesta.status}`);
        }

        const datos = await respuesta.json();
        console.log("Datos obtenidos:", datos);
    } catch (error) {
        console.error("Fallo la comunicación de red:", error);
    }
}
```
