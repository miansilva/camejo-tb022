# Módulo 07: Desarrollo frontend: HTML5, CSS3 y maquetado moderno

Guía de estudio conceptual y técnica completa sobre la estructura semántica con **HTML5**, diseño estético con **CSS3**, el **modelo de caja**, sistemas de diagramado **Flexbox** y **CSS Grid**, posicionamiento y **diseño responsive**.

---

## 1. Fundamentos de HTML5 y estructura semántica

**HTML (HyperText Markup Language)** es el lenguaje de marcado estándar que define la estructura y el esqueleto de las aplicaciones y páginas web.

### Estructura canónica de un documento HTML5

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Título de la aplicación</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <!-- Contenido visible de la aplicación -->
</body>
</html>
```

### Componentes de la cabecera (`<head>`)
- `<!DOCTYPE html>`: Declara al navegador que el documento sigue la especificación HTML5.
- `<meta charset="UTF-8">`: Define la codificación de caracteres para interpretar adecuadamente acentos y caracteres especiales.
- `<meta name="viewport" content="width=device-width, initial-scale=1.0">`: **Indispensable para Responsive Design**. Adapta el ancho del área de visión al ancho del dispositivo emulador o móvil.

### Etiquetas semánticas de estructura
El uso de etiquetas semánticas reemplaza el uso indiscriminado de `<div>`, otorgando significado accesible a los lectores de pantalla y motores de búsqueda (SEO):

```
┌─────────────────────────────────────────────────────────────┐
│                         <header>                            │
├─────────────────────────────────────────────────────────────┤
│                          <nav>                              │
├──────────────────────────────┬──────────────────────────────┤
│           <main>             │           <aside>            │
│  ┌────────────────────────┐  │  (Barra lateral, enlaces     │
│  │       <article>        │  │   relacionados, publicidad)  │
│  └────────────────────────┘  │                              │
│  ┌────────────────────────┐  │                              │
│  │       <section>        │  │                              │
│  └────────────────────────┘  │                              │
├──────────────────────────────┴──────────────────────────────┤
│                         <footer>                            │
└─────────────────────────────────────────────────────────────┘
```

- **`<header>`**: Cabecera de la página o de una sección (títulos, logos, búsquedas).
- **`<nav>`**: Bloque que contiene los enlaces principales de navegación del sitio.
- **`<main>`**: Contenido principal y único de la página (solo debe existir uno por documento).
- **`<article>`**: Bloque de contenido independiente y autocontenido (ej. post de blog, tarjeta de producto).
- **`<section>`**: Agrupación temática de contenido relacionado.
- **`<aside>`**: Contenido complementario o indirectamente relacionado con el contenido principal.
- **`<footer>`**: Pie de página (derechos de autor, políticas de privacidad, datos de contacto).

---

## 2. Formularios e interacción con el usuario

Los formularios (`<form>`) son el mecanismo mediante el cual el usuario envía datos hacia el servidor.

```html
<form action="/api/v1/usuarios" method="POST">
    <!-- Asociación semántica entre label e input usando 'for' e 'id' -->
    <label for="nombre">Nombre Completo:</label>
    <input type="text" id="nombre" name="nombre" placeholder="Ej. Gonzalo Martínez" required maxlength="50">

    <label for="email">Correo Electrónico:</label>
    <input type="email" id="email" name="email" placeholder="usuario@dominio.com" required>

    <label for="password">Contraseña (Mínimo 8 caracteres):</label>
    <input type="password" id="password" name="password" required minlength="8">

    <label for="telefono">Teléfono (Formato 123-456-7890):</label>
    <input type="tel" id="telefono" name="telefono" pattern="\d{3}-\d{3}-\d{4}">

    <button type="submit">Registrar Usuario</button>
</form>
```

### Atributos de validación nativa en formularios
- **`required`**: Obliga a completar el campo antes de enviar el formulario.
- **`placeholder`**: Texto de sugerencia temporal que desaparece al escribir.
- **`pattern`**: Define una **Expresión Regular (Regex)** que el valor ingresado debe cumplir para ser válido.
- **`maxlength` / `minlength`**: Limita el número de caracteres permitidos.
- **`min` / `max`**: Restringe valores numéricos o de fechas.

---

## 3. Fundamentos de CSS3, selectores y especificidad

**CSS (Cascading Style Sheets)** controla la presentación visual, colores, fuentes y maquetado de la interfaz.

### Selectores y especificidad
Cuando dos o más reglas CSS aplican estilos contradictorios sobre un mismo elemento, el navegador resuelve el conflicto evaluando el puntaje de **Especificidad**:

```
CÁLCULO DE ESPECIFICIDAD DE SELECTORES
 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 │ Inline Style │     IDs      │  Clases /    │ Elementos /  │
 │ (style="")   │   (#header)  │ Pseudoclases │ Pseudoelem.  │
 ├──────────────┼──────────────┼──────────────┼──────────────┤
 │  1, 0, 0, 0  │  0, 1, 0, 0  │  0, 0, 1, 0  │  0, 0, 0, 1  │
 └──────────────┴──────────────┴──────────────┴──────────────┘
```

```css
/* Especificidad 0, 0, 0, 1 (Elemento) */
p { color: black; }

/* Especificidad 0, 0, 1, 0 (Clase) */
.texto-destacado { color: blue; }

/* Especificidad 0, 1, 0, 0 (ID - Gana a las anteriores) */
#primer-parrafo { color: red; }
```

> ⚠️ **La regla `!important`**: Invalida la cascada normal de especificidad. Su uso excesivo se considera un antipatrón ya que dificulta el mantenimiento del código.

---

## 4. El modelo de caja (box model)

Todo elemento renderizado en la página web es conceptualmente una caja rectangular compuesta por 4 capas concéntricas:

```
┌─────────────────────────────────────────────────────────────┐
│                          MARGIN                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                       BORDER                          │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │                    PADDING                      │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │                 CONTENT                   │  │  │  │
│  │  │  │           (Ancho x Alto)                  │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

- **Content**: El área del contenido real (texto, imágenes).
- **Padding**: Espacio de relleno interno entre el contenido y el borde.
- **Border**: Línea de borde que rodea el padding.
- **Margin**: Espacio transparente externo que separa la caja de otros elementos vecinos.

### Control del ancho: `content-box` vs. `border-box`
Por defecto (`box-sizing: content-box`), el ancho total de una caja se calcula sumando `width + padding + border`. Esto causa desarreglos al maquetar.

**Buenas prácticas modernas en CSS**:
```css
/* Forzar que width incluya padding y border de forma intuitiva */
*, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}
```

---

## 5. Maquetado con Flexbox (1D - Unidimensional)

Flexbox está diseñado para distribuir y alinear elementos a lo largo de un solo eje (horizontal o vertical).

```
Eje principal (Main Axis) ───────────────────────────►
 ┌─────────────────────────────────────────────────────────┐
 │ ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
 │ │ Flex Item 1 │   │ Flex Item 2 │   │ Flex Item 3 │     │  Eje cruzado
 │ └─────────────┘   └─────────────┘   └─────────────┘     │  (Cross Axis)
 └─────────────────────────────────────────────────────────┘  │
                                                              ▼
```

### Propiedades del contenedor flex (`display: flex`)
```css
.contenedor-flex {
    display: flex;
    flex-direction: row;            /* row | row-reverse | column | column-reverse */
    justify-content: space-between; /* flex-start | flex-end | center | space-between | space-around */
    align-items: center;            /* flex-start | flex-end | center | stretch */
    flex-wrap: wrap;                /* nowrap | wrap */
    gap: 1.5rem;                    /* Espaciado entre ítems */
}
```

### Propiedades de los ítems flex
```css
.item-flex {
    flex-grow: 1;    /* Proporción para crecer si hay espacio disponible */
    flex-shrink: 0;  /* Capacidad de encogerse si falta espacio */
    flex-basis: 250px; /* Tamaño base inicial antes de distribuir espacio */
}
```

---

## 6. Maquetado con CSS Grid (2D - Bidimensional)

CSS Grid permite diseñar esquemas complejos controlando columnas y filas simultáneamente.

```css
.grid-container {
    display: grid;
    /* Crear 3 columnas: la 1ra fija de 200px, las otras 2 distribuidas equitativamente con fr */
    grid-template-columns: 200px repeat(2, 1fr);
    grid-template-rows: auto 1fr auto;
    gap: 20px;
    
    /* Maquetado declarativo por áreas */
    grid-template-areas: 
        "header header header"
        "sidebar main main"
        "footer footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

---

## 7. Posicionamiento en CSS

La propiedad `position` especifica cómo se ubica un elemento en el flujo del documento:

| Valor | Comportamiento en el flujo |
| :--- | :--- |
| **`static`** | Comportamiento por defecto. Sigue el flujo normal de la página. Ignores `top`, `left`, `z-index`. |
| **`relative`** | Conserva su espacio original en el flujo, pero se desplaza relativo a su posición inicial. |
| **`absolute`** | Se remueve del flujo normal. Se posiciona relativo al ancestro más cercano que tenga `position != static`. |
| **`fixed`** | Se remueve del flujo y se posiciona relativo a la ventana (*viewport*). Permanece fijo al hacer scroll. |
| **`sticky`** | Alterna entre `relative` y `fixed` según la posición del scroll del usuario. |

---

## 8. Diseños adaptables (responsive web design) y media queries

El **Responsive Web Design** asegura que la interfaz se adapte visualmente al tamaño de pantalla del dispositivo (móviles, tablets, laptops).

```css
/* Estilos generales Mobile-First (Para celulares) */
.contenedor {
    display: flex;
    flex-direction: column;
}

/* Media Query para pantallas medianas o superiores (Tablets / Laptops) */
@media (min-width: 768px) {
    .contenedor {
        flex-direction: row;
    }
}
```
