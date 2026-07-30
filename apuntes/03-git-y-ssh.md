# Módulo 03: Control de versiones con Git y autenticación por SSH

Guía de estudio conceptual y técnica completa sobre **Git**, su arquitectura interna de almacenamiento de objetos, estrategias de ramificación, rebase interactivo y autenticación criptográfica con **llaves SSH**.

---

## 1. Fundamentos de Git y sistemas de control de versiones (VCS)

Un **Version Control System (VCS)** es una herramienta fundamental en la ingeniería de software para gestionar y registrar los cambios realizados sobre el código fuente a lo largo del tiempo.

### Centralizado (SVN) vs. distribuido (Git)

```
SISTEMA CENTRALIZADO (ej. Subversion / SVN)
  [Desarrollador A] ──┐
  [Desarrollador B] ──┼──> [ Servidor Centralizado ]
  [Desarrollador C] ──┘    (Fallo único de punto central / Dependencia de conexión)

SISTEMA DISTRIBUIDO (ej. Git)
  [Dev A: Repo Completo] ◄──┐
  [Dev B: Repo Completo] ◄──┼──► [ Servidor Remoto (GitHub) ]
  [Dev C: Repo Completo] ◄──┘    (Cada nodo posee el historial completo / Operaciones locales)
```

- **VCS Centralizado**: Existe un único servidor que guarda el historial de revisiones. Si el servidor falla o se pierde la conexión a la red, los desarrolladores no pueden consultar el historial ni hacer commits.
- **VCS Distribuido (DVCS)**: Cada clonación genera una réplica exacta y autónoma de la base de datos del proyecto con todo su historial. Todas las operaciones de commit, ramificación y consulta de logs son **locales y ultra rápidas**.

---

## 2. El modelo interno y la arquitectura de Git

### Los tres estados y las tres zonas de trabajo

Git gestiona los archivos distribuyéndolos en tres áreas conceptuales:

```
┌─────────────────────────┐      ┌─────────────────────────┐      ┌─────────────────────────┐
│    Working Directory    │      │   Staging Area (Index)  │      │    Local Repository     │
│  (Directorio de trabajo)├─────>│  (Área de preparación)  ├─────>│   (Base de datos .git)  │
└─────────────────────────┘      └─────────────────────────┘      └─────────────────────────┘
      Archivos en                   `git add <file>`                    `git commit`
     modificación.
```

1. **Working Directory (Directorio de trabajo)**: Archivos del proyecto en el sistema de archivos del sistema operativo.
2. **Staging Area / Index (Área de preparación)**: Zona intermedia donde se agrupan de forma granular los cambios exactos que se incluirán en la próxima captura (*snapshot*).
3. **Local Repository (.git)**: Directorio oculto que actúa como la base de datos orientada a objetos donde Git guarda permanentemente las versiones como commits.

### La estructura de objetos en Git
A diferencia de otros VCS que guardan diferencias de texto (*diffs*), Git almacena el proyecto mediante **capturas completas de estado (snapshots)** usando un grafo acíclico dirigido (DAG) de 4 tipos de objetos identificados por su hash criptográfico SHA-1/SHA-256:

```
                  ┌─────────────────┐
                  │  COMMIT OBJECT  │ (Autor, Fecha, Mensaje, Padre)
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │   TREE OBJECT   │ (Representa un directorio)
                  └────┬───────┬────┘
                       │       │
            ┌──────────┘       └──────────┐
            ▼                             ▼
   ┌─────────────────┐           ┌─────────────────┐
   │   TREE OBJECT   │           │   BLOB OBJECT   │ (Contenido puro de archivo)
   └────────┬────────┘           └─────────────────┘
            │
            ▼
   ┌─────────────────┐
   │   BLOB OBJECT   │
   └─────────────────┘
```

- **Blob (Binary Large Object)**: Almacena exclusivamente el contenido de un archivo de texto o binario (sin metadatos como el nombre o los permisos).
- **Tree**: Representa una carpeta o directorio. Contiene punteros a Blobs (archivos) u otros Trees (subcarpetas) asociándolos con sus nombres y permisos POSIX.
- **Commit**: Contiene la referencia al Tree raíz del proyecto en ese instante, el autor, el committer, la marca de tiempo, el mensaje explicativo y el hash del commit anterior (padre).
- **Annotated Tag**: Puntero permanente etiquetado que referencia a un commit específico (ej. `v1.0.0`).

---

## 3. Comandos esenciales del flujo de trabajo

### Configuración global de identidad
```bash
# Configurar firma de commits
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"

# Definir la rama principal por defecto
git config --global init.defaultBranch main
```

### Operaciones básicas e inspección
```bash
# Inicializar un nuevo repositorio
git init

# Clonar un repositorio remoto
git clone git@github.com:usuario/proyecto.git

# Inspeccionar el estado de los archivos
git status

# Inspeccionar las diferencias no preparadas
git diff

# Inspeccionar las diferencias preparadas en el Staging Area
git diff --staged

# Inspeccionar un commit específico
git show HASH_COMMIT

# Preparar cambios para el próximo commit
git add archivo.py
git add .

# Registrar un commit con mensaje semántico
git commit -m "feat(auth): agregar middleware de autenticación JWT"

# Consultar el historial en formato gráfico condensado
git log --oneline --graph --all
```

### Deshacer cambios y restauración (Restore, Reset y Revert)

#### 1. Restaurar archivos individuales (`git restore`)
- **`git restore archivo.py`**: Descarta los cambios locales no guardados en el Working Directory, volviendo el archivo a la versión del último commit.
- **`git restore --staged archivo.py`**: Quita un archivo del Staging Area (deshace `git add`), manteniendo las modificaciones en el disco.

---

#### 2. Mover el puntero de la rama (`git reset`)
`git reset` desplaza el puntero `HEAD` y la rama activa hacia un commit anterior (por ejemplo, `HEAD~1` representa retroceder 1 commit). La diferencia clave entre sus 3 modos radica en **dónde quedan los cambios deshechos**:

##### Efectos de `git reset HEAD~1` (Deshacer el último commit):

| Modo | Staging Area (Index) | Working Directory (Disco) |
| :--- | :--- | :--- |
| **`--soft`** | ✅ Conserva los cambios | ✅ Conserva los cambios |
| **`--mixed`** *(Por defecto)* | ❌ Quita los cambios del Staging | ✅ Conserva los cambios |
| **`--hard`** *(Destructivo)* | ❌ Quita los cambios del Staging | ❌ Elimina los cambios del disco |


##### Ejemplos prácticos de `git reset`:

* **`git reset --soft HEAD~1` (Soft Reset)**
  Deshace el último commit pero **deja todos sus archivos listos en el Staging Area**. Ideal si te equivocaste en el mensaje del commit o te olvidaste de incluir un archivo:
  ```bash
  # Deshacer el último commit conservando los cambios preparados (Staged)
  git reset --soft HEAD~1

  # Hacer la corrección y volver a comitear
  git add archivo_olvidado.py
  git commit -m "feat(auth): mensaje corregido con todos los archivos"
  ```

* **`git reset --mixed HEAD~1` (Mixed Reset - Por defecto)**
  Deshace el último commit y saca los archivos del Staging Area, pero **mantiene las modificaciones locales en el disco** como cambios no preparados:
  ```bash
  # Deshacer el commit y sacar todo del Staging (los cambios quedan en el disco)
  git reset HEAD~1  # o git reset --mixed HEAD~1
  ```

* **`git reset --hard HEAD~1` (Hard Reset - DESTRUCTIVO)**
  **Elimina por completo el commit y borra permanentemente todos los cambios** tanto del Staging como del disco. El proyecto vuelve exactamente a cómo estaba en el commit anterior:
  ```bash
  # ⚠️ ¡CUIDADO! Elimina el último commit y destruye los cambios locales
  git reset --hard HEAD~1
  ```

---

#### 3. Revertir commits en ramas públicas (`git revert`)
- **`git revert HASH_COMMIT`**: Es la alternativa **segura para repositorios remotos/compartidos**. En lugar de reescribir el historial (como hace `reset`), crea un **nuevo commit inverso** que deshace exactamente los cambios del commit indicado sin alterar el historial previo.

### El almacén temporal (`git stash`)
Permite guardar temporalmente los cambios sin terminar en el Working Directory sin necesidad de hacer un commit prematuro:

```bash
# Guardar cambios actuales en el stash
git stash push -m "WIP: trabajo en progreso de validación"

# Listar los stashes guardados
git stash list

# Recuperar y aplicar el último stash borrándolo de la lista
git stash pop

# Aplicar un stash específico sin eliminarlo
git stash apply stash@{0}
```

---

## 4. Estrategias de ramificación y fusión (branching)

Una **rama (branch)** en Git es simplemente un puntero móvil de 41 bytes que referencia a un hash de commit específico.

### `git merge` vs `git rebase`

#### Fusionar (`git merge`)
Integra los cambios de una rama destino creando un nuevo **Merge Commit** de 3 vías.

```
Historial real con Merge:
  A ─── B ─── C ─── M (Merge Commit) [main]
         └── D ─── E                 [feature]
```
- **Ventaja**: Mantiene el historial cronológico exacto de lo que ocurrió.
- **Desventaja**: Produce grafos ramificados y complejos en equipos grandes.

#### Rebasar (`git rebase`)
Desplaza la base de la rama actual al extremo de la rama principal destino, aplicando uno a uno los commits nuevamente.

```
Historial lineal con Rebase:
  A ─── B ─── C ─── D' ─── E' [main/feature]
```
- **Ventaja**: Mantiene una historia perfectamente limpia y lineal.
- **Regla de oro del rebase**: **NUNCA hacer rebase sobre ramas públicas o compartidas** (`main`/`develop`), ya que reescribe el historial y romperá las copias de otros desarrolladores.

### Rebase interactivo (`git rebase -i`)
Permite reescribir, limpiar, consolidar o eliminar commits locales antes de enviarlos al servidor remoto:

```bash
# Iniciar rebase interactivo de los últimos 3 commits
git rebase -i HEAD~3
```

Opciones disponibles en el editor interactivo:
- **`pick`**: Conserva el commit tal como está.
- **`reword`**: Conserva el commit pero permite modificar el mensaje.
- **`squash`**: Combina el commit con el commit anterior, fusionando los mensajes.
- **`fixup`**: Combina el commit con el anterior descartando el mensaje actual.
- **`drop`**: Elimina completamente el commit.

### Resolución teórica de conflictos de merge
Ocurre cuando Git detecta modificaciones contradictorias sobre las mismas líneas de un archivo en ambas ramas:

```html
<<<<<<< HEAD (Tu rama actual)
const API_URL = "https://api.produccion.com";
=======
const API_URL = "https://api.staging.com";
>>>>>>> feature/config (Rama entrante)
```
1. Inspeccionar las marcas de conflicto (`<<<<<<<`, `=======`, `>>>>>>>`).
2. Editar el archivo dejando la versión definitiva correcta.
3. Ejecutar `git add archivo.py` y completar la fusión con `git commit`.

---

## 5. Estrategias de trabajo en equipo (workflows)

| Modelo workflow | Características principales | Caso de uso recomendado |
| :--- | :--- | :--- |
| **Git Flow** | Ramas permanentes (`main`, `develop`) y ramas temporales (`feature/*`, `release/*`, `hotfix/*`). | Proyectos de software tradicionales con ciclos de release planificados y versionado semántico. |
| **GitHub Flow** | Una rama principal `main` siempre estable. Creación de ramas `feature/*`, apertura de **Pull Requests (PR)** y despliegue directo a producción tras la aprobación. | Equipos con despliegue continuo (SaaS, Web Apps). |
| **Trunk-Based** | Todos los desarrolladores envían pequeños cambios frecuentemente a la rama principal (`trunk`), usando *Feature Flags* para ocultar código incompleto. | Equipos de alta madurez con entrega continua (CI/CD ultra rápido). |

---

## 6. Autenticación criptográfica por llaves SSH

SSH (Secure Shell) utiliza criptografía asimétrica de llave pública/privada para autenticar las operaciones remotas (`git push`, `git fetch`) de forma cifrada y sin contraseña.

```
┌─────────────────────────────────────────────────────────────┐
│                   EQUIPO LOCAL DEL DESARROLLADOR            │
│  Llave Privada (~/.ssh/id_ed25519) [Permisos 600 - SECRETA] │
└──────────────────────────────┬──────────────────────────────┘
                               │ Firma digital cifrada
┌──────────────────────────────▼──────────────────────────────┐
│                    SERVIDOR REMOTO (GitHub)                 │
│  Llave Pública (~/.ssh/id_ed25519.pub) [PÚBLICA]            │
└─────────────────────────────────────────────────────────────┘
```

### Configuración paso a paso (Algoritmo Ed25519)
```bash
# 1. Generar el par de llaves asimétricas Ed25519
ssh-keygen -t ed25519 -C "tu.email@ejemplo.com"

# 2. Iniciar el agente SSH en segundo plano
eval "$(ssh-agent -s)"

# 3. Registrar la llave privada en el agente local
ssh-add ~/.ssh/id_ed25519

# 4. Copiar la llave pública (.pub) para vincularla en la consola web de GitHub
cat ~/.ssh/id_ed25519.pub

# 5. Probar la autenticación
ssh -T git@github.com
```
