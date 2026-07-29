# Introducción al desarrollo de software (Cátedra Camejo - FIUBA)

Repositorio centralizado con guías completas de estudio por unidad y bibliografía para la asignatura **Introducción al desarrollo de software (75.18 / Cátedra Camejo)** de la Facultad de Ingeniería de la Universidad de Buenos Aires (FIUBA).

> [!NOTE]
> **Notas sobre el material, organización y autoría:**
> - **Autoría y herramientas**: Este repositorio y los apuntes de la carpeta `apuntes/` fueron desarrollados en base a las diapositivas y programa de la materia, con revisión y estructuración asistida por IA.
> - **Organización**: La división y numeración de módulos en `apuntes/` responde a un criterio propio para optimizar el estudio personal, por lo que puede diferir del orden cronológico estricto de las clases.
> - **Bibliografía**: El material en `bibliografia/` es parcial. Las guías y la bibliografía oficial completa están disponibles en la [web oficial del curso](https://www.intro-camejo.com.ar/).

---

## 📑 Guías de estudio en apuntes

Los contenidos teóricos completos para estudiar cada tema se encuentran en la carpeta [`apuntes/`](./apuntes):

- [`01-introduccion-e-ingenieria.md`](./apuntes/01-introduccion-e-ingenieria.md) - Introducción, SDLC, metodologías ágiles (Scrum/Kanban), DRY/KISS/YAGNI, verificación/validación y entornos.
- [`02-entorno-y-linux.md`](./apuntes/02-entorno-y-linux.md) - Linux, arquitectura POSIX, jerarquía FHS (`/bin`, `/etc`), permisos octales (`chmod`/`chown`), comandos CLI y editor Vim.
- [`03-git-y-ssh.md`](./apuntes/03-git-y-ssh.md) - Git (working dir, staging, commits), objetos internos, ramas, merge vs rebase y llaves SSH (`ed25519`).
- [`04-bash-y-regex.md`](./apuntes/04-bash-y-regex.md) - Bash scripting, modo estricto (`set -euo pipefail`), variables especiales, evaluación `(( ))`, arreglos, redirecciones, pipes y Regex PCRE (`grep`/`sed`/`awk`).
- [`05-bases-de-datos-y-sql.md`](./apuntes/05-bases-de-datos-y-sql.md) - Modelo relacional, PK/FK, orden de ejecución en SQL, JOINs (Inner/Left/Right/Full), subconsultas y `HAVING`.
- [`06-docker-y-contenedores.md`](./apuntes/06-docker-y-contenedores.md) - Contenedores vs VMs (namespaces/cgroups), Dockerfile optimizado, volúmenes, inicialización de DBs (`/docker-entrypoint-initdb.d/`) y Docker Compose.
- [`07-desarrollo-backend.md`](./apuntes/07-desarrollo-backend.md) - Modelo cliente-servidor, protocolo HTTP/HTTPS, verbos/códigos de estado, principios REST, arquitectura en 3 capas y JWT.
- [`08-javascript.md`](./apuntes/08-javascript.md) - JavaScript (ES6+), tipos de datos, arrays/objetos, Event Loop, asincronía (`Promise`, `async/await`), prototipos, clases, DOM y Fetch API.

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
