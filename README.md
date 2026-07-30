# Introducción al desarrollo de software (Cátedra Camejo - FIUBA)

Repositorio centralizado con guías de estudio completas por unidad y bibliografía para la asignatura **Introducción al desarrollo de software (75.18 / Cátedra Camejo)** de la Facultad de Ingeniería de la Universidad de Buenos Aires (FIUBA).

> [!NOTE]
> **Notas sobre el material, organización y autoría:**
> - **Autoría y herramientas**: Este repositorio y los apuntes de la carpeta `apuntes/` fueron redactados y enriquecidos sobre la base de los contenidos, diapositivas y programa de la materia.
> - **Organización**: La división y numeración en 9 módulos en `apuntes/` responde a una estructuración lógica para optimizar el estudio y repaso personal.
> - **Bibliografía**: El material en `bibliografia/` es complementario. Las guías y la bibliografía oficial completa están disponibles en la [web oficial del curso](https://www.intro-camejo.com.ar/).

---

## 📑 Guías de estudio en apuntes

Los contenidos teóricos completos para estudiar cada unidad se encuentran organizados en los 9 módulos de la carpeta [`apuntes/`](./apuntes):

1. [`01-introduccion-e-ingenieria.md`](./apuntes/01-introduccion-e-ingenieria.md) - Introducción a la materia, Ingeniería de Software, crisis del software (1968), SDLC, ISO/IEC 25010 (RNF), Waterfall, V-Model, metodologías ágiles (Scrum/Kanban), principios DRY/KISS/YAGNI, deuda técnica y pirámide de testing.
2. [`02-entorno-y-linux.md`](./apuntes/02-entorno-y-linux.md) - Arquitectura UNIX/Linux, espacio del Kernel vs espacio de usuario, jerarquía FHS (`/proc`, `/sys`), permisos POSIX (notación octal, `umask`, SUID/SGID/Sticky bit), administración de procesos (`kill`, background `&`) y editores Nano/Vim.
3. [`03-git-y-ssh.md`](./apuntes/03-git-y-ssh.md) - VCS centralizado vs distribuido, internals de Git (blobs, trees, commits, tags), llaves SSH (`ed25519`), comandos `reset`/`restore`/`revert`, rebase interactivo (`git rebase -i`), resolución de conflictos y workflows (Git Flow, GitHub Flow, Trunk-Based).
4. [`04-bash-y-regex.md`](./apuntes/04-bash-y-regex.md) - Shell Unix, Bash scripting, modo estricto (`set -euo pipefail`), expansiones de parámetros (`${var#}`/`${var%}`), `[[ ... ]]` vs `[ ... ]`, redirecciones, `tee`, trampas `trap` y expresiones regulares con `grep`, `sed` y `awk`.
5. [`05-bases-de-datos-y-sql.md`](./apuntes/05-bases-de-datos-y-sql.md) - Modelo relacional, diseño E-R, normalización (1NF a 3NF), sublenguajes (DDL, DML, DQL, TCL), orden de ejecución en `SELECT`, JOINs (Inner, Left, Right, Full), subconsultas correlacionadas y transacciones ACID.
6. [`06-docker-y-contenedores.md`](./apuntes/06-docker-y-contenedores.md) - Virtualización vs Contenedores (namespaces, cgroups), buenas prácticas en Dockerfile, optimización de caché, Multi-Stage Builds, persistencia (Volumes vs Bind Mounts), redes Docker y orquestación con Docker Compose.
7. [`07-html-y-css.md`](./apuntes/07-html-y-css.md) - HTML5 semántico, validación nativa de formularios, especificidad en CSS3, modelo de caja (`content-box` vs `border-box`), Flexbox (1D), CSS Grid (2D), posicionamiento y Responsive Web Design.
8. [`08-javascript.md`](./apuntes/08-javascript.md) - Motor V8 (Heap vs Call Stack), Event Loop (Microtasks vs Macrotasks), cierres (*closures*), hoisting, sintaxis ES6+, Promesas, `async/await`, manipulación del DOM y consumo de APIs con `fetch()`.
9. [`09-desarrollo-backend.md`](./apuntes/09-desarrollo-backend.md) - Modelo Cliente-Servidor, protocolo HTTP/HTTPS, idempotencia y seguridad de métodos, status codes, principios RESTful, arquitectura en 3 capas (Controller, Service, Repository), Express.js y autenticación con JWT.

---

## 📚 Bibliografía local

Documentos y presentaciones locales disponibles en [`bibliografia/`](./bibliografia):

- **Módulo 01**: `01_introduccion_materia.pdf`, `01_presentacion_catedra.pdf`, `01_ingenieria_de_software.pdf`, `01_etapas_ingenieria_de_software.pdf`
- **Módulo 02**: `02_introduccion_a_linux.pdf`, `02_introduccion_terminal.pdf`, `02_filesystem_linux.pdf`, `02_editores_de_texto.pdf`
- **Módulo 03**: `03_introduccion_a_git.pdf`, `03_configuracion_llaves_ssh.pdf`
- **Módulo 04**: `04_slides_bash_scripting.html`, `04_introduccion_a_regex.pdf`
- **Módulo 06**: `06_mastering_custom_docker_images.pptx`, `06_docker_data_architecture.pptx`
- **Módulo 07**: `07_desarrollo_backend_api.pdf`
- **Módulo 08**: `08_apunte_javascript.pdf`

---

## 🔗 Recursos útiles y enlaces recomendados por la cátedra

### 🏛️ Sitios oficiales
- **Web oficial del curso**: [https://www.intro-camejo.com.ar/](https://www.intro-camejo.com.ar/)
- **GitHub oficial de la cátedra**: [https://github.com/intro-camejo/web](https://github.com/intro-camejo/web)

### 🐧 Linux, terminal y entorno
- **The Linux Command Line (Libro de referencia en PDF/HTML)**: [https://linuxcommand.org/tlcl.php](https://linuxcommand.org/tlcl.php)
- **Top Linux Commands (DigitalOcean)**: [https://www.digitalocean.com/community/tutorials/linux-commands](https://www.digitalocean.com/community/tutorials/linux-commands)
- **Atajos interactivos de Vim (Vim CheatSheet)**: [https://vim.rtorr.com/](https://vim.rtorr.com/)
- **Instalación de WSL2 en Windows (Microsoft Docs)**: [https://learn.microsoft.com/es-es/windows/wsl/install](https://learn.microsoft.com/es-es/windows/wsl/install)

### 🖥️ Bash y expresiones regulares (Regex)
- **Probador interactivo de Regex (regex101 - Usado en clase)**: [https://regex101.com](https://regex101.com)
- **Tutorial interactivo de Regex desde cero (RegexLearn)**: [https://regexlearn.com/](https://regexlearn.com/)
- **Entorno online para probar Bash scripts (OnlineGDB)**: [https://www.onlinegdb.com/online_bash_shell](https://www.onlinegdb.com/online_bash_shell)

### 🗄️ SQL y bases de datos
- **Ejecutor de consultas SQL online (DB Fiddle)**: [https://www.db-fiddle.com/](https://www.db-fiddle.com/)
- **Documentación oficial de PostgreSQL 16**: [https://www.postgresql.org/docs/16/index.html](https://www.postgresql.org/docs/16/index.html)

### 🐳 Docker y contenedores
- **Documentación oficial de Docker**: [https://docs.docker.com/](https://docs.docker.com/)
- **Repositorio de imágenes oficiales (Docker Hub)**: [https://hub.docker.com/](https://hub.docker.com/)

### 🌐 Desarrollo Web, APIs y JavaScript
- **MDN Web Docs (Mozilla Developer Network - JS / HTTP)**: [https://developer.mozilla.org/es/](https://developer.mozilla.org/es/)
- **Depurador e inspección de tokens JWT (jwt.io)**: [https://jwt.io/](https://jwt.io/)
