# Módulo 01: Introducción a la materia e ingeniería de software

Guía de estudio conceptual y técnica completa sobre los fundamentos de la **ingeniería de software, el ciclo de vida del software (SDLC), los modelos de desarrollo y la calidad de software**.

---

## 1. La ingeniería de software y la crisis del software

### La crisis del software (1968)
A finales de la década de 1960, el rápido avance del hardware y la capacidad de cómputo sobrepasó las capacidades operativas para desarrollar programas de manera controlada. Los proyectos informáticos sufrían de:
- **Costos desbordados**: Superaban severamente los presupuestos originales.
- **Entregas fuera de plazo**: Demoras de meses o años sobre lo planificado.
- **Falta de confiabilidad e inestabilidad**: Sistemas plagados de fallos y difíciles de mantener.
- **Insatisfacción del cliente**: Productos que no resolvían las necesidades reales del usuario.

En la célebre conferencia de la **NATO en Garmisch (1968)** se acuñó el término **"La crisis del software"** y se formalizó la necesidad de tratar la producción de software como una disciplina rigurosa de la ingeniería.

### Definiciones formales
- **Definición de IEEE (IEEE 729)**: *"La aplicación de un enfoque sistemático, disciplinado y cuantificable al desarrollo, operación y mantenimiento del software; es decir, la aplicación de la ingeniería al software."*
- **Ian Sommerville (Autor de referencia)**: *"Una disciplina de la ingeniería que se ocupa de todos los aspectos de la producción de software, desde las etapas iniciales de la especificación del sistema hasta el mantenimiento del sistema después de que se ha puesto en uso."*

### Tipos de software
- **De sistema**: Brinda servicios básicos a otros programas (ej. sistemas operativos, compiladores, controladores).
- **De aplicación**: Resuelve necesidades de negocio o usuarios finales (ej. sistemas de gestión, navegadores web, procesadores de texto).
- **Científico / Ingeniería**: Algoritmos de simulación y cálculo complejo.
- **Embebido / Enbebido**: Integrado directamente en hardware de dispositivos (ej. automotriz, electrodomésticos, dispositivos IoT).

### Enfoque disciplinado y calidad
```
┌───────────────────────────────────────────────────────────────────────────────┐
│                            INGENIERÍA DE SOFTWARE                             │
├───────────────────────────────────────────────────────────────────────────────┤
│  Procesos disciplinados + Métodos técnicos + Herramientas de calidad          │
│  ───────> Entregar software confiable, mantenible, seguro y dentro del tiempo │
└───────────────────────────────────────────────────────────────────────────────┘
```

> **Verificación vs. validación (Diferencia conceptual esencial)**:
> - **Verificación**: ¿Estamos construyendo el producto *correctamente*? (Cumplimiento riguroso de especificaciones técnicas y ausencia de errores).
> - **Validación**: ¿Estamos construyendo el producto *correcto*? (Satisfacción real de las necesidades del cliente y usuarios finales).

---

## 2. El ciclo de vida del software (SDLC)

El **Software Development Life Cycle (SDLC)** representa el proceso completo que abarca desde la concepción del sistema hasta su puesta en producción y eventual retiro.

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Requerimientos  ├────>│  Diseño / Arq.   ├────>│  Implementación  │
└──────────────────┘     └──────────────────┘     └─────────┬────────┘
                                                            │
┌──────────────────┐     ┌──────────────────┐               │
│   Mantenimiento  │<────┤    Despliegue    │<──────────────┘
└────────┬─────────┘     └──────────────────┘
         │ Iteración / Evolución
         └────────────────────────────────────────┐
```

### Las 6 etapas fundamentales

#### 1. Análisis y elicitación de requerimientos
Es la fase inicial donde se identifica **qué** debe resolver el sistema.
- **Elicitación (Indagación)**: Entrevistas, observación directa y talleres con stakeholders.
- **Negociación y priorización**: Alineación de expectativas según restricciones de presupuesto y tiempo.
- **Especificación**: Elaboración de la Documentación de Requerimientos (SRS / Historias de Usuario).
- **Validación**: Confirmación formal con el cliente de los requerimientos redactados.

#### 2. Diseño y arquitectura de software
Se define **cómo** funcionará el sistema internamente:
- Selección de patrones arquitectónicos (Monolito, Microservicios, MVC, Cliente-Servidor).
- Diseño de la base de datos (Modelo E-R, esquemas relacionales o no relacionales).
- Definición de interfaces y APIs de integración.

#### 3. Implementación y codificación
Traducción del diseño en código fuente ejecutable, aplicando:
- Estándares de estilo y convenciones de nombres.
- Principios de código limpio (**DRY**, **KISS**, **YAGNI**).
- Control de versiones distribuido (Git).

#### 4. Aseguramiento de calidad (QA) y niveles de pruebas
Evaluación sistemática para detectar y corregir defectos:
- **Pruebas unitarias**: Verifican funciones, métodos o clases aisladas.
- **Pruebas de integración**: Comprueban la comunicación y flujo entre distintos módulos o servicios.
- **Pruebas de sistema**: Evalúan el comportamiento global del sistema end-to-end.
- **Pruebas de aceptación (UAT)**: Validadas directamente por el usuario o cliente de negocio antes de salir a producción.

#### 5. Despliegue (deployment) y pipeline de entornos
Promoción progresiva del código a través de entornos aislados:

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ Development ├─────>│     QA      ├─────>│   Staging   ├─────>│ Production  │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
  Desarrollo           Pruebas de          Réplica exacta       Entorno vivo
  e integración        calidad y           de producción       con usuarios
  de funciones.        automatización.     para validación.    reales.
```

#### 6. Mantenimiento y evolución
- **Correctivo**: Corrección de bugs detectados en producción.
- **Adaptativo**: Modificaciones para operar sobre nuevas plataformas o dependencias.
- **Perfectivo/Evolutivo**: Incorporación de nuevas características para agregar valor al negocio.

---

## 3. Clasificación de requerimientos (estándar ISO/IEC 25010)

Los requerimientos definen el contrato funcional y técnico del software. Se dividen en dos categorías principales:

### Requerimientos funcionales (RF)
Describen las **funciones, servicios o comportamientos específicos** que el sistema debe realizar.
- *Ejemplo*: "El sistema debe permitir a los usuarios autenticados exportar sus reportes en formato PDF."
- *Ejemplo*: "Se debe enviar un correo electrónico de confirmación al registrar una compra."

### Requerimientos no funcionales (RNF)
Especifican las **propiedades, cualidades y restricciones globales** bajo las cuales debe operar el sistema. Se estructuran según el estándar **ISO/IEC 25010**:

| Categoría RNF | Descripción y métricas |
| :--- | :--- |
| **Eficiencia de desempeño** | Tiempos de respuesta (ej. `< 200ms`), rendimiento (throughput) y uso óptimo de recursos (CPU/RAM). |
| **Seguridad** | Confidencialidad, integridad, cifrado de datos en tránsito/reposo (TLS 1.3, AES-256) y autenticación/autorización. |
| **Usabilidad** | Facilidad de aprendizaje, accesibilidad (normas WCAG) y prevención de errores del usuario. |
| **Confiabilidad** | Tolerancia a fallos, disponibilidad (ej. `99.9% uptime`) y capacidad de recuperación (disaster recovery). |
| **Mantenibilidad** | Modularidad, facilidad de lectura de código, testabilidad y reusabilidad de componentes. |
| **Portabilidad** | Capacidad del sistema para ejecutarse en diferentes entornos, navegadores u sistemas operativos. |

---

## 4. Modelos de desarrollo de software

### 1. Modelo en cascada (waterfall)
Modelo secuencial e inflexible donde cada fase debe completarse y aprobarse rigurosamente antes de iniciar la siguiente.

```
┌──────────────────┐
│  Requerimientos  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      Diseño      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Implementación  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│     Pruebas      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│    Despliegue    │
└──────────────────┘
```
- **Ventajas**: Documentación exhaustiva y planificación clara para proyectos pequeños con requerimientos inmutables.
- **Desventajas**: Inflexible al cambio, entrega valor recién al final del proceso y alto riesgo de detectar errores de análisis en etapas tardías.

### 2. Modelo en V (V-model)
Variante de la cascada que asocia directamente cada etapa de desarrollo con su correspondiente fase de pruebas.

```
  Análisis de Requerimientos ───────> Pruebas de Aceptación (UAT)
            │                                    ▲
            ▼                                    │
    Diseño del Sistema ──────────────> Pruebas de Sistema
            │                                    ▲
            ▼                                    │
    Diseño Arquitectura ─────────────> Pruebas de Integración
            │                                    ▲
            ▼                                    │
     Diseño Detallado ───────────────> Pruebas Unitarias
            │                                    ▲
            └───────────> CODIFICACIÓN <──────────┘
```

### 3. Metodologías ágiles (agile)
Enfoque **iterativo e incremental** orientado a entregar software funcional frecuentemente, aceptando cambios en los requerimientos incluso en etapas avanzadas.

#### Los 4 valores del manifiesto ágil (2001)
1. **Individuos e interacciones** por encima de procesos y herramientas.
2. **Software funcionando** por encima de documentación defensiva/exhaustiva.
3. **Colaboración con el cliente** por encima de negociación contractual.
4. **Respuesta ante el cambio** por encima de seguimiento estricto de un plan.

---

## 5. Frameworks ágiles: Scrum y Kanban

### Framework Scrum
Scrum organiza el trabajo en ciclos de duración fija (normalmente de 2 a 4 semanas) denominados **Sprints**.

#### Roles principales
- **Product Owner (PO)**: Representa la voz del cliente/negocio, prioriza el Product Backlog y maximiza el valor del producto.
- **Scrum Master (SM)**: Facilitador del proceso, elimina bloqueos y asegura que el equipo comprenda los principios de Scrum.
- **Developers (Equipo de Desarrollo)**: Equipo multidisciplinario autoorganizado responsable de entregar un incremento funcional al finalizar cada Sprint.

#### Artefactos de Scrum
- **Product Backlog**: Lista priorizada de todas las características, historias de usuario y mejoras del producto.
- **Sprint Backlog**: Conjunto de ítems del Product Backlog seleccionados para el Sprint actual, con su correspondiente plan de ejecución.
- **Incremento**: La suma de todos los ítems completados durante un Sprint que cumplen con la **Definition of Done (DoD)** y están listos para ser desplegados.

#### Ceremonias (eventos) de Scrum
```
 ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
 │ Sprint Planning │ ────> │  Daily Standup  │ ────> │  Sprint Review  │ ────> │  Retrospective  │
 └─────────────────┘       └─────────────────┘       └─────────────────┘       └─────────────────┘
  Planificación de          Reunión diaria de         Demostración del          Reflexión e
  objetivos e historias.    15m para sincronizar.     incremento al cliente.    mejora continua.
```

### Framework Kanban
Metodología ágil basada en la visualización del flujo de trabajo continuo.

#### Principios clave
- **Tablero visual**: Representación de columnas (ej. *Por Hacer*, *En Progreso*, *En Revisión*, *Terminado*).
- **Límites de trabajo en progreso (WIP Limits)**: Restricción explícita de la cantidad máxima de tareas permitidas simultáneamente en cada estado para evitar cuellos de botella y reducir el multitareas.
- **Gestión del flujo**: Optimización continua del tiempo de entrega desde la solicitud hasta la puesta en producción (Lead Time y Cycle Time).

---

## 6. Principios de código limpio y deuda técnica

### Principios fundamentales
- **DRY (Don't Repeat Yourself)**: Evitar la duplicación de código extrayendo lógica repetida a funciones o módulos reutilizables.
- **KISS (Keep It Simple, Stupid)**: Mantener la solución más simple posible; evitar abstracciones prematuras y sobreingeniería.
- **YAGNI (You Aren't Gonna Need It)**: No escribir código ni funcionalidades especulativas basándose en "posibles necesidades futuras".

### Deuda técnica (technical debt)
Concepto introducido por **Ward Cunningham** que compara los accesos directos o decisiones de desarrollo apresuradas con una deuda financiera.
- **Principal**: El ahorro inmediato de tiempo al aplicar un hack o solución sucia.
- **Interés**: El esfuerzo adicional que se paga en cada cambio futuro debido a la complejidad y rigidez introducidas.
- **Pagando la deuda**: Se paga mediante la **Refactorización (Refactoring)**, que consiste en reestructurar el código existente para mejorar su mantenibilidad sin alterar su comportamiento externo.
