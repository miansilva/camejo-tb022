# Módulo 01: Introducción a la materia e ingeniería de software

Guía de estudio completa para comprender los fundamentos conceptuales de la asignatura **Introducción al desarrollo de software (Cátedra Camejo - FIUBA)**.

---

## 1. ¿Qué es la ingeniería de software?

### La crisis del software (1968)
En los años 60, con el aumento del poder de cómputo de las computadoras, los proyectos de software comenzaron a crecer en complejidad de forma desmedida. La mayoría de los proyectos superaban el presupuesto estimado, se entregaban con retrasos de meses/años o eran inestables e ineficientes. A este fenómeno se lo denominó **La crisis del software** en la conferencia de la NATO de 1968 en Garmisch.

Para solucionar este problema nació la **Ingeniería de software**: la aplicación de principios de ingeniería al desarrollo de software.

### Definiciones formales

> **Definición de IEEE (IEEE 729)**:
> *"La aplicación de un enfoque sistemático, disciplinado y cuantificable al desarrollo, operación y mantenimiento del software; es decir, la aplicación de la ingeniería al software."*

> **Ian Sommerville (Autor de referencia)**:
> *"Una disciplina de la ingeniería que se ocupa de todos los aspectos de la producción de software, desde las etapas iniciales de la especificación del sistema hasta el mantenimiento del sistema después de que se ha puesto en uso."*

### Tipos de software y atributos de calidad

#### Clasificación del software
- **De sistema**: Brinda servicios básicos a otros programas (ej. sistemas operativos, compiladores).
- **De aplicación**: Resuelve necesidades de negocio o de usuarios finales (ej. sistemas de gestión, navegadores).
- **Científico / Ingeniería**: Algoritmos de simulación y cálculo complejo.
- **Embebido / Enbebido**: Integrado directamente en hardware de dispositivos (ej. automotriz, electrodomésticos).

#### Criterios de calidad técnica
- ✅ Cumple su objetivo funcional y de negocio.
- ✅ Es usable, accesible e intuitivo.
- ✅ Mantiene una performance y tiempos de respuesta adecuados.
- ✅ Es mantenible, confiable, seguro y escalable en el tiempo.
- ❌ *Aclaración*: El tipo de licencia (paga o libre) **no** es un indicador de calidad de software.

---

## 2. El ciclo de vida del software (SDLC)

El **Software Development Life Cycle (SDLC)** es el proceso estructurado que sigue un sistema de software desde su concepción inicial hasta su retiro definitivo.

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Requerimientos ├────>│ Diseño / Arq.    ├────>│  Implementación  │
└─────────────────┘     └──────────────────┘     └─────────┬────────┘
                                                           │
┌─────────────────┐     ┌──────────────────┐               │
│  Mantenimiento  │<────┤    Despliegue    │<──────────────┘
└────────┬────────┘     └──────────────────┘
         │ (Iteración)
         └────────────────────────────────────────┐
```

### Las 6 etapas detalladas

#### Etapa 1: Análisis de requerimientos
- **Objetivo**: Comprender a fondo el problema y las necesidades reales del cliente antes de comenzar a codificar. Es la etapa que requiere **mayor esmero**: construir bien un producto equivocado es un fallo crítico.
- **Actividades principales**:
  1. *Indagación (Elicitación)*: Entrevistas y recolección de necesidades.
  2. *Negociación*: Definición de alcance y prioridades según presupuesto.
  3. *Especificación*: Documentación de historias de usuario o SRS.
  4. *Validación*: Confirmación con el cliente de los requerimientos redactados.
- **Tipos de requerimientos**:
  - **Funcionales (RF)**: Definen *lo que el sistema debe hacer* (ej. "Permitir autenticación de usuarios").
  - **No funcionales (RNF)**: Definen *las restricciones de comportamiento* (ej. "Tiempos de respuesta < 200ms", "Cifrado TLS 1.3").

#### Etapa 2: Diseño y arquitectura de software
- **Objetivo**: Definir la estructura del sistema y seleccionar la arquitectura técnica adecuada para asegurar que sea escalable y mantenible.

#### Etapa 3: Implementación y codificación
- **Objetivo**: Traducir las especificaciones y diseños en código ejecutable siguiendo estándares de código limpio.

#### Etapa 4: Aseguramiento de calidad, verificación y validación (QA / Testing)

Existe una diferencia conceptual clave entre verificación y validación:

```
┌─────────────────────────────────────────────────────────────┐
│                 VERIFICACIÓN vs VALIDACIÓN                  │
├──────────────────────────────┬──────────────────────────────┤
│         Verificación         │          Validación          │
│ ¿Construimos el producto     │ ¿Construimos el producto     │
│       correctamente?         │          correcto?           │
├──────────────────────────────┼──────────────────────────────┤
│ Comprueba que el software    │ Demuestra que el software    │
│ cumple con las especificaciones│ satisface las expectativas │
│ técnicas y no tiene errores. │ reales del cliente/usuario.  │
└──────────────────────────────┴──────────────────────────────┘
```

- **Niveles de prueba**:
  - *Pruebas unitarias*: Evalúan funciones aisladas.
  - *Pruebas de integración*: Evalúan la comunicación entre módulos.
  - *Pruebas de aceptación (UAT)*: Ejecutadas por los usuarios finales para validar la solución.

#### Etapa 5: Despliegue y entornos (Deployment Environments)

El código se promociona progresivamente a través de 4 entornos aislados:

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ Development ├─────>│     QA      ├─────>│   Staging   ├─────>│ Production  │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
  Desarrollo           Pruebas de          Preproducción        Usuarios finales
  e integración.       calidad.            (Réplica exacta).   (Estable).
```

> ⚠️ **Aclaración**: "Validation" no es un entorno de despliegue, sino el proceso de confirmación de requerimientos. Nunca debe desplegarse en producción sin validación previa.

#### Etapa 6: Mantenimiento y evolución
- **Objetivo**: Mantener el software operativo ante errores (correctivo), cambios de entorno (adaptativo) o nuevas necesidades de negocio (evolutivo).

---

## 3. Modelos de desarrollo de software

### 1. Modelo en cascada (Waterfall)
Flujo lineal y secuencial estricto. Cada fase debe terminarse completamente antes de avanzar. Adecuado únicamente si los requerimientos son fijos e inmutables.

### 2. Metodologías ágiles (Scrum / Kanban)
Desarrollo **iterativo e incremental**.

- **Los 4 valores del manifiesto ágil**:
  1. Individuos e interacciones sobre procesos y herramientas.
  2. Software funcionando sobre documentación exhaustiva.
  3. Colaboración con el cliente sobre negociación contractual.
  4. Respuesta ante el cambio sobre seguir un plan.

- **Scrum**: Organiza el trabajo en *Sprints* de duración fija con roles definidos (Product Owner, Scrum Master, Developers) y ceremonias (Planning, Daily, Review, Retrospective).

---

## 4. Principios de diseño y calidad técnica

### Principios de código limpio
- **DRY (Don't Repeat Yourself)**: Evitar duplicación de lógica.
- **KISS (Keep It Simple, Stupid)**: Buscar la solución más simple y legible.
- **YAGNI (You Aren't Gonna Need It)**: No implementar funcionalidad no requerida en el presente.

### Deuda técnica (Technical Debt)
Metáfora de **Ward Cunningham** que representa el costo futuro causado por elegir soluciones sucias o provisorias en el presente. Se paga mediante la **refactorización (refactoring)** de código.
