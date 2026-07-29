# Módulo 03: Control de versiones (Git) y autenticación SSH

Guía de estudio completa para comprender el funcionamiento interno de **Git**, los comandos del flujo de trabajo y la seguridad mediante **Llaves SSH**.

---

## 1. Fundamentos de Git y sistemas de control de versiones

Un **Sistema de control de versiones (VCS - Version Control System)** es una herramienta que registra los cambios realizados en un conjunto de archivos a lo largo del tiempo, permitiendo recuperar versiones específicas, comparar cambios y colaborar en equipo.

### Sistemas centralizados vs distribuidos

```
Centralizado (SVN, CVS):
  [Dev A] ──┐
  [Dev B] ──┼──> [ Servidor Central (Único repositorio) ]
  [Dev C] ──┘   (Si se cae el servidor o no hay internet, no hay historial)

Distribuido (Git):
  [Dev A (Repo Completo)] ◄──┐
  [Dev B (Repo Completo)] ◄──┼──► [ Servidor Remoto (GitHub / GitLab) ]
  [Dev C (Repo Completo)] ◄──┘
```

- **Sistemas centralizados (SVN, CVS)**: Un único servidor central contiene todo el historial de versiones. Si el servidor falla o no hay conexión a internet, no se puede hacer commit ni consultar el historial.
- **Sistemas distribuidos (Git, Mercurial)**: Cada desarrollador posee una copia completa y local de todo el repositorio con su historial íntegro. Se puede hacer commits, crear ramas y consultar el historial totalmente fuera de línea (*offline*).

---

## 2. El modelo interno de Git

### Los tres estados y las tres zonas de Git

```
┌─────────────────────────┐      ┌─────────────────────────┐      ┌─────────────────────────┐
│    Working Directory    │      │  Staging Area (Index)   │      │    Local Repository     │
│ (Archivos modificados)  ├─────>│  (Cambios preparados)   ├─────>│  (Commits guardados)    │
└─────────────────────────┘      └─────────────────────────┘      └─────────────────────────┘
         Archivos en                      `git add`                      `git commit`
      edición local.
```

1. **Working Directory (Directorio de trabajo)**: Archivos reales en tu sistema operativo donde estás editando o escribiendo código.
2. **Staging Area / Index (Área de preparación)**: Zona intermedia donde seleccionás qué modificaciones específicas formarán parte de tu próximo commit.
3. **Local Repository (.git)**: La base de datos orientada a objetos dentro de la carpeta oculta `.git` donde Git almacena de forma permanente las capturas (*snapshots*) de tu proyecto.

### La estructura de objetos en Git

Git no almacena diferencias línea por línea (difs), sino capturas completas del sistema de archivos mediante 4 tipos de objetos identificados por un hash SHA-1 / SHA-256:

- **Blob**: Almacena únicamente el contenido de un archivo (sin su nombre ni permisos).
- **Tree**: Representa un directorio. Asocia nombres de archivo y permisos con sus correspondientes hashes de Blobs o Trees.
- **Commit**: Objeto que apunta a un Tree raíz y contiene metadatos: autor, fecha, mensaje y hash del commit padre (permitiendo construir el grafo del historial).
- **Tag**: Etiqueta que apunta a un commit específico (ej. `v1.0.0`).

---

## 3. Guía completa de comandos Git

### Configuración inicial de identidad
```bash
# Configurar nombre y correo global para firmar los commits
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"

# Definir 'main' como la rama por defecto al inicializar repositorios
git config --global init.defaultBranch main
```

### Flujo de trabajo básico
```bash
# Inicializar un nuevo repositorio local en la carpeta actual
git init

# Clonar un repositorio remoto existente
git clone git@github.com:usuario/repositorio.git

# Inspeccionar el estado de los archivos (tracked, untracked, modified, staged)
git status

# Agregar archivos al Staging Area
git add archivo.py       # Agrega un archivo específico
git add .                # Agrega todas las modificaciones del directorio actual

# Crear un commit en la historia local
git commit -m "feat(auth): implementar validación de contraseña"

# Consultar el historial de commits
git log --oneline --graph --all
```

### Ramificación y fusión (Branching & Merging)

Las **ramas** en Git son simplemente punteros móviles y ligeros que apuntan a un commit determinado.

```bash
# Crear una nueva rama
git branch feature/login

# Cambiar a una rama existente
git switch feature/login
# (O sintaxis tradicional: git checkout feature/login)

# Crear y cambiar a una nueva rama en un solo paso
git switch -c feature/login

# Listar todas las ramas locales
git branch -a

# Fusionar la rama 'feature/login' dentro de la rama actual
git merge feature/login

# Eliminar una rama local integrada
git branch -d feature/login
```

#### Diferencia entre `git merge` y `git rebase`

```
Con Merge: (Conserva la historia real con un commit de fusión)
  A ─── B ─── C ─── M (Merge Commit)
         └── D ─── E (Feature)

Con Rebase: (Reescribe la historia de forma lineal)
  A ─── B ─── D' ─── E' ─── C
```

- **`git merge`**: Une dos ramas creando un commit especial de fusión (*Merge Commit*). Mantiene intacta la historia cronológica real.
- **`git rebase`**: Mueve la base de tu rama actual al extremo de la rama destino, volviendo a aplicar tus commits uno a uno. Produce una historia perfectamente lineal. **Regla de oro**: NUNCA hacer rebase sobre ramas públicas o compartidas.

### Operaciones con remotos
```bash
# Enlazar el repositorio local a un servidor remoto en GitHub
git remote add origin git@github.com:usuario/repositorio.git

# Enviar los commits locales al repositorio remoto
git push -u origin main

# Descargar las novedades del remoto SIN fusionar con tu código local
git fetch origin

# Descargar e integrar automáticamente los cambios remotos (equivalente a fetch + merge)
git pull origin main
```

### Deshacer cambios (Undo)
```bash
# Descartar los cambios no guardados en un archivo (volver al último commit)
git restore archivo.py

# Quitar un archivo del Staging Area pero conservar sus modificaciones en el disco
git restore --staged archivo.py

# Volver a un commit anterior borrando todo el trabajo posterior (PELIGRO)
git reset --hard HASH_COMMIT

# Deshacer los cambios de un commit previo creando un nuevo commit inverso de seguridad
git revert HASH_COMMIT
```

---

## 4. Autenticación criptográfica por llaves SSH

SSH (Secure Shell) permite autenticarse con servidores remotos como GitHub de forma segura sin necesidad de ingresar usuario y contraseña en cada petición.

### Criptografía asimétrica

Funciona mediante un par de claves matemáticas:

```
[ Tu Computadora ]                                   [ Servidor GitHub ]
┌──────────────────────────┐                         ┌──────────────────────────┐
│  Llave Privada (SECRET)  │                         │  Llave Pública (PÚBLICA) │
│   ~/.ssh/id_ed25519      ├────── (Autenticación) ─►│   Agregada en Ajustes    │
└──────────────────────────┘                         └──────────────────────────┘
```

- **Llave privada (`id_ed25519`)**: Se almacena localmente en tu equipo con permisos restringidos (`600`). **Jamás debe compartirse**.
- **Llave pública (`id_ed25519.pub`)**: Se sube al servidor remoto (GitHub/GitLab). Cualquiera puede verla.

### Configuración paso a paso de llaves SSH

```bash
# 1. Generar un par de llaves usando el algoritmo moderno Ed25519
ssh-keygen -t ed25519 -C "tu.email@ejemplo.com"

# 2. Iniciar el agente SSH en segundo plano
eval "$(ssh-agent -s)"

# 3. Agregar la llave privada al agente SSH
ssh-add ~/.ssh/id_ed25519

# 4. Copiar el contenido de la llave pública para pegarla en GitHub
cat ~/.ssh/id_ed25519.pub

# 5. Probar la conexión segura con GitHub
ssh -T git@github.com
# Salida esperada: Hi user! You've successfully authenticated...
```
