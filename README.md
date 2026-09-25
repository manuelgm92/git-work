# git-work

Plantilla mínima para una página web de startup diseñada para practicar y demostrar el **flujo colaborativo profesional con Git y GitHub** (gestión de ramas, apertura de *issues*, *pull requests*, resolución de conflictos y versionado mediante *tags/releases*).

Este repositorio forma parte de práctica del flujo colaborativo completo (fork simulado, issue, rama, PR, revisión, conflicto, etiqueta y release) del módulo DPL.

## Índice

- [Entorno e instalación](#entorno-e-instalacion)
- [Paso a paso de la práctica](#paso-a-paso-de-la-práctica)
- [Configuración](#configuracion)
- [Comprobación](#comprobacion)
- [Problemas encontrados y solución](#problemas-encontrados-y-solución)

## Entorno e instalación

### Preparación del Entorno (Docker):

1. Comprobar e instalar Docker.

```bash
docker --version    # indica la versión de Docker instalada
docker info         # confirma que el dameon responde
```

Si Docker no está instalado, instálalo con las instrucciones oficiales para tu sistema ([Docker Engine](https://docs.docker.com/engine/install/) o [Docker Desktop](https://www.docker.com/products/docker-desktop/)) y repite la comprobación.

2. Crear la carpeta de trabajo y el contenedor de laboratorio.

```bash
mkdir -p dpl    # Crea la carpeta dpl
```

Para crear y ejecutar el contenedor, en **Mac** o **Linux**:
```bash
docker run -dit --name dpl-lab \
  -v $(pwd):/home/alumno/dpl \
  -p 80:80 -p 8080:8080 -p 8000:8000 \
  ubuntu:24.04
```

En **Windows**:
```powershell
docker run -dit --name dpl-lab `
  -v ${PWD}:/home/alumno/dpl `
  -p 80:80 -p 8080:8080 -p 8000:8000 `
  ubuntu:24.04
```

El comando docker run... nos devolverá un hash del contenedor de Docker creado.

dpl-lab es tu "máquina" del curso.

3. Entra en el laboratorio y crea el usuario de trabajo.
```bash
docker exec -it dpl-lab bash # accede a la bash del contenedor
useradd -m -s /bin/bash alumno # crea el usuario alumno con su home y bash
mkdir -p /home/alumno/dpl # crea la carpeta de trabajo dentro del contenedor
chown -R alumno:alumno /home/alumno/dpl # da la propiedad al usuario alumno
ls -ld /home/alumno/dpl # comprueba que el directorio pertenece a alumno:alumno
find /home/alumno/dpl -maxdepth 1 -printf '%u:%g %p\n' # comprueba la propiedad de los elementos
exit # sale de la bash del contenedor
```

4. Instala las herramientas base dentro del laboratorio.
```bash
docker exec dpl-lab bash -c "apt-get update && apt-get install -y curl git nano"
```

5. Crear el contenedor cliente.
```bash
docker run -dit --name dpl-cliente ubuntu:24.04
docker exec dpl-cliente bash -c "apt-get update && apt-get install -y curl dnsutils lftp openssh-client python3-pip"
```

dpl-cliente representa "otra máquina" desde la que probar los servicios del laboratorio (curl, dig, lftp, sftp, requests de Python)

6. Conecta ambos contenedores en una red propia.
```bash
docker network create dpl-net
docker network connect dpl-net dpl-lab
docker network connect dpl-net dpl-cliente
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dpl-lab # devuelve la IP de dpl-lab
```

7. Prueba la comunicación entre contenedores.
```bash
docker exec dpl-cliente bash -c "curl -s http://IP_DPL_LAB:80/ || echo 'sin servicio en el 80 (normal aún)'" # cambiar ID_DPL_LAB por la ip de dpl-lab
```

Devolverá: sin servicio en el 80 (normal aún)

--- 

### Preparación del git-wrok y espejo:

Para trabajar en solitario, se simula el flujo de dos usuarios (`user1` y `user2`) mediante un **repositorio espejo** en GitHub, ya que la plataforma no permite hacer *fork* de un repositorio propio.

1. Crea el repositorio principal `git-work` en GitHub (papel de `user1`) con licencia MIT y README.
2. Crea un segundo repositorio vacío llamado `git-work-espejo` en GitHub (papel de `user2`).


## Paso a paso de la práctica

### Paso 1 — user1: crear el repositorio remoto y la copia local

1. Crea el repositorio remoto `git-work` en GitHub (público, con licencia MIT y README inicial). 
2. Clona el repositorio en carpetal local (`ae1`) dentro del contenedor:

```bash
cd ~/dpl
git clone git@github.com:TU_USUARIO/git-work.git ae1
cd ae1
git config --local user.name "Nombre Apellidos"
git config --local user.email "correo@aula.local"
git config --local --list
git remote -v
```

**Resultado esperado**: el repositorio tiene `main` con un commit inicial (README + LICENSE) y `origin` apunta a tu repositorio.

### Paso 2 — user1: añadir la página y la hoja de estilos

```bash
mkdir -p css
# crea index.html, css/cover.css y .gitignore con el contenido anterior
git add index.html css/cover.css .gitignore
git commit -m "Añade la página de la startup y su hoja de estilos" \
           -m "Incluye index.html, css/cover.css (plantilla cover) y un .gitignore mínimo para entornos y logs."
git push origin main
git log --oneline
```

Con `nano index.html` se puede copiar y pegar el código o cgontenido:

- En `index.html`, copia el siguiente contenido:

```html
<!doctype html>
<html lang="es" class="h-100">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>git-work — startup</title>
    <link href="css/cover.css" rel="stylesheet">
  </head>
  <body class="d-flex h-100 text-center text-white bg-dark">
    <div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
      <header class="mb-auto">
        <h3 class="float-md-start mb-0">git-work</h3>
        <nav class="nav nav-masthead justify-content-center float-md-end">
          <a class="nav-link active" href="#">Inicio</a>
          <a class="nav-link" href="#">Equipo</a>
          <a class="nav-link" href="#">Contacto</a>
        </nav>
      </header>

      <main class="px-3">
        <h1>Tu startup, versionada.</h1>
        <p class="lead">Plantilla mínima para practicar el flujo colaborativo con Git y GitHub.</p>
        <p class="lead">
          <a href="#" class="btn btn-lg btn-secondary fw-bold border-white bg-white">Saber más</a>
        </p>
      </main>

      <footer class="mt-auto text-white-50">
        <p>Práctica DPL — flujo colaborativo con Git</p>
      </footer>
    </div>
  </body>
</html>
```
- En `cover.css`:
```css
/*
 * Globals
 */


/* Custom default button */
.btn-secondary,
.btn-secondary:hover,
.btn-secondary:focus {
  color: #333;
  text-shadow: none; /* Prevent inheritance from `body` */
}


/*
 * Base structure
 */

body {
  text-shadow: 0 .05rem .1rem rgba(0, 0, 0, .5);
  box-shadow: inset 0 0 5rem rgba(0, 0, 0, .5);
}

.cover-container {
  max-width: 42em;
}


/*
 * Header
 */

.nav-masthead .nav-link {
  padding: .25rem 0;
  font-weight: 700;
  color: rgba(255, 255, 255, .5);
  background-color: transparent;
  border-bottom: .25rem solid transparent;
}

.nav-masthead .nav-link:hover,
.nav-masthead .nav-link:focus {
  border-bottom-color: rgba(255, 255, 255, .25);
}

.nav-masthead .nav-link + .nav-link {
  margin-left: 1rem;
}

.nav-masthead .active {
  color: #fff;
  border-bottom-color: #fff;
}
```

En el caso de querer editar el contenido o abrir el código en Visual Studio Code. [Véase aquí.](#abrir-carpeta-contenedera-del-codigo-y-contenido-del-proyecto-en-visual-studio-code)

### Paso 3 — user1: añadir el workflow de integración continua

MkDocs consigue convertir archivos MarkDown en un sitio web de documentación profesional, limpio y ordenado de forma automática.

Configura MkDocs para generar documentación automática y añade el workflow de GitHub Actions en .github/workflows/ci.yml, mkdocs.yml y docs/index.md. Sube los cambios para validar que el linter y la construcción pasen en verde:

```yml
name: Construir la documentación

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el repositorio
        uses: actions/checkout@v4
      - name: Preparar Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Instalar MkDocs
        run: pip install mkdocs
      - name: Construir el sitio
        run: mkdocs build --strict
```
El workflow se compone de:

* `on`: eventos que lo disparan (`push` a `main`, cualquier `pull_request` y ejecución manual);
* `jobs.docs`: un trabajo en una máquina virtual Ubuntu de GitHub;
* `actions/checkout@v4`: descarga el repositorio en la máquina del runner;
* `actions/setup-python@v5` y `pip install mkdocs`: preparan el generador de documentación;
* `mkdocs build --strict`: construye el sitio y falla si hay enlaces rotos o páginas fuera de la navegación, de modo que cada cambio se valida automáticamente.

El generador necesita un mínimo de configuración en el repositorio: `mkdocs.yml` (título y navegación) y `docs/index.md` (portada de la documentación).

```yml
# mkdocs.yml
site_name: git-work
nav:
  - Inicio: index.md
```

```md
<!-- docs/index.md -->
# git-work

Documentación del repositorio colaborativo de la AE1.

- [Portada del sitio](index.md)
- [Repositorio remoto](https://github.com/USUARIO/git-work)
```

En la ubicación donde se encuentra `git-work` (~/dpl/ae1), ejecuta los siguientes comandos:
```bash
mkdir -p .github/workflows docs
# crea ci.yml, mkdocs.yml y docs/index.md con el contenido anterior
git add .github/workflows/ci.yml mkdocs.yml docs/index.md
git commit -m "Añade documentación con MkDocs e integración continua" \
           -m "Workflow que construye el sitio con mkdocs build --strict en cada push y pull request; la portada enlaza al sitio y al repositorio remoto."
git push origin main
````

**Resultado esperado**: en GitHub, pestaña **Actions**, aparece la ejecución del _workflow_ en verde.

### Paso 4 — user1: crear la issue del trabajo pendiente

Crea la primera issue en GitHub o mediante terminal:

En GitHub, **Issues > New issue**:

1. Título: Add custom text for startup contents.
2. Descripción: el texto de la portada debe personalizarse para la startup.
3. **Create**.

Anota el número (`#1`). Desde la terminal:

```bash
gh issue create --title "Add custom text for startup contents" \
  --body "Sustituir el texto genérico de la portada por el de la startup."
```

**Resultado esperado**: la issue `#1` está abierta y visible en el repositorio.

### Paso 5 — user2: configuración del espejo y clonado (Modalidad Individual)

Dado que realizas la práctica en solitario, vincula el espejo y clónalo en una carpeta independiente para simular al user2:

```bash
# Desde la carpeta de user1 (ae1), añade el espejo y sube la rama main
cd ~/dpl/ae1
git remote add espejo git@github.com:TU_USUARIO/git-work-espejo.git
git push espejo main

# Clona el espejo en una carpeta separada para user2 y añade upstream
cd ~/dpl
git clone git@github.com:TU_USUARIO/git-work-espejo.git ae1-user2
cd ae1-user2
git remote add upstream git@github.com:TU_USUARIO/git-work.git
git fetch upstream
```

En la **modalidad por parejas**:

En GitHub, abre `https://github.com/USUARIO_USER1/git-work` y pulsa **Fork > Create fork**. 
Después clona **tu fork** en tu equipo:

```bash
cd ~/dpl
git clone git@github.com:USUARIO_USER2/git-work.git ae1-user2
cd ae1-user2
git remote -v      # origin = tu fork; comprueba si existe upstream = repositorio de user1
```

Si `upstream` no aparece, añádelo a mano (lo necesitarás para sincronizar al final):

```bash
git remote add upstream git@github.com:USUARIO_USER1/git-work.git
git fetch upstream
Con GitHub CLI el fork y el clon se hacen en un paso:
```

Con GitHub CLI el fork y el clon se hacen en un paso:

```bash
gh repo fork USUARIO_USER1/git-work --clone
```

**Resultado esperado**: `git remote -v` muestra `origin` (tu fork) y `upstream` (el repositorio de user1).

### Paso 6 — user2: rama `custom-text` y Pull Request

Desde la carpeta de user2 (`ae1-user2`), crea la rama de trabajo, personaliza los textos de index.html y abre el Pull Request:

```bash
cd ~/dpl/ae1-user2
git switch -c custom-text
# Edita index.html con los cambios de tu startup
git add index.html
git commit -m "Personaliza la portada para la startup" \
           -m "Sustituye el texto genérico por el nombre, eslogan y propuesta de valor."
git push origin custom-text

# Sube también la rama a upstream para facilitar la PR en modalidad individual
git push upstream custom-text
```

Abre el Pull Request en GitHub seleccionando base `main` y compare `custom-text`, asegurándote de no usar la palabra "Closes #1" en la descripción todavía.

En **la modalidad por parejas**:

Trabaja siempre en una rama, nunca directamente sobre main:

```bash
git switch -c custom-text
# personaliza index.html: cambia el título, el eslogan y el nombre de la startup
git add index.html
git commit -m "Personaliza la portada para la startup" \
           -m "Sustituye el texto genérico por el nombre, el eslogan y la propuesta de valor de la startup."
git push -u origin custom-text
```

Abre el pull request hacia el repositorio de user1 (**Pull requests > New pull request**, base `USUARIO_USER1:main`, compare `USUARIO_USER2:custom-text`) o con:

```bash
gh pr create --base main --head USUARIO_USER2:custom-text \
  --title "Add custom text for startup contents" \
  --body "Personaliza la portada. Relacionado con #1."
```

En la descripción del PR no uses `Closes #1` todavía: la issue se cerrará al fusionar. Marca la casilla **Allow edits by maintainers** para que user1 pueda hacer commits en tu rama durante la revisión.

**Resultado esperado**: el PR está abierto, la integración continua se ejecuta sobre él y user1 recibe la notificación.

### Paso 7 — user1: probar el PR en local y mantener la conversación

Desde la carpeta de User 1 (`~/dpl/ae1`), descárgala y cámbiate a ella:
```bash
cd ~/dpl/ae1
git fetch espejo
git switch -c custom-text espejo/custom-text
```

1. Abre `index.html` en tu editor de código.
2. Modifica ligeramente el texto del pie de página (`<footer>`).
3. Guarda los cambios, haz commit y súbelos al repositorio principal (`origin`):

```bash
git add index.html
git commit -m "Ajusta el texto del pie de página" \
           -m "Mejora la redacción del pie propuesta en la revisión del PR #1."
git push origin custom-text
```

Como User 2: Simular la respuesta al comentario.

1. Cambia a tu carpeta de User 2 (`~/dpl/ae1-user2`)
2. Descarga el cambio que acaba de hacer User 1 en el repositorio `espejo` (`origin`):

```bash
cd ~/dpl/ae1-user2
git pull origin custom-text
```

4. Abre `index.html`, edita un poco el eslogan para atender la revisión y guarda el archivo. Haz commit y súbelo a su espejo (origin):

```bash
git add index.html 
git commit -m "Afina el eslogan de la portada" \
           -m "Atiende el comentario de la revisión."
git push origin custom-text
```

En la **modalidad por parejas**:

Para revisar el PR en tu máquina tienes dos opciones:

```bash
# Opción A: GitHub CLI descarga la rama del PR y la deja lista
gh pr checkout 1

# Opción B: manual, añadiendo un remoto al fork de user2
git remote add upstream git@github.com:USUARIO_USER2/git-work.git
git fetch upstream custom-text
git switch -c custom-text upstream/custom-text
```

Haz una mejora sobre la propuesta de user2 (por ejemplo, el texto del pie), confírmala y súbela a la rama del PR:

```bash
# edita index.html (texto del footer)
git add index.html
git commit -m "Ajusta el texto del pie de página" \
           -m "Mejora la redacción del pie propuesta en la revisión del PR #1."
git push upstream custom-text   # con gh pr checkout, usa: git push USUARIO_USER2 custom-text
```

La conversación se mantiene en la página del PR: comentarios, peticiones de cambio y respuestas. Cada parte debe aportar al menos un cambio más:

```bash
gh pr comment 1 --body "He ajustado el pie; ¿puedes revisar el eslogan?"
# user2 responde con un commit nuevo (desde su clon del fork)
git add index.html && git commit -m "Afina el eslogan de la portada" -m "Atiende el comentario de la revisión."
git push origin custom-text
```

**Resultado esperado**: la rama `custom-text` contiene commits de user1 y de user2, y el PR refleja la conversación.

### Paso 8 — user1: aprobar, fusionar y cerrar la issue

En **GitHub**, dentro del repositorio, **Pull requests**, comenta "Revisado y probado en local.". **Merge pull request > Confirm merge**


En la **modalidad por parejas**:

Aprueba el PR y fusiónalo. Si el PR incluye `Closes #1` en la descripción o en el mensaje de fusión, la issue se cierra automáticamente:

```bash
gh pr review 1 --approve --body "Revisado y probado en local."
gh pr merge 1 --merge --delete-branch --body "Fusiona el PR #1. Closes #1."
git switch main
git pull
git log --oneline --graph --all
```

**Resultado esperado**: `main` contiene el commit de fusión y la issue `#1` aparece cerrada.

User2 sincroniza su fork con el repositorio original:

```bash
cd ~/dpl/ae1-user2
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```
Con GitHub CLI: `gh repo sync USUARIO_USER2/git-work --source USUARIO_USER1/git-work --branch main`.





## Configuración
* Estructura del repositorio: Contiene los archivos base de la plantilla HTML/CSS en la raíz, la documentación bajo la carpeta `docs/`, y las acciones automatizadas en `.github/workflows/ci.yml`.

## Comprobación

Verificación rápida del estado del entorno y repositorios remotos:

```bash
git remote -v          # Comprueba los remotos configurados (origin, espejo, upstream)
git status             # Working tree limpio
git branch -avv        # Muestra las ramas locales y su vinculación remota
```

Comprobaciones rápidas de Docker:

```bash
docker --version    # indica la versión de Docker instalada
docker info         # confirma que el dameon responde
docker ps           # Indica que contenedores están en ejecución
```


`docker ps` devolverá algo similar a esto:
```bash
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS                                                                                                                           NAMES
bf63157e3fff   ubuntu:24.04   "/bin/bash"   9 minutes ago   Up 9 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   dpl-lab
```

## Problemas encontrados y solución

|Problema|Causa|Solución|
|---------------------|-----|--------|
|Error de permisos SSH al clonar (Permission denied (publickey))|Las claves SSH no están generadas o configuradas correctamente en el contenedor/entorno local.| Generar el par de claves (`ssh-keygen -t ed25519`), añadirlas al agente y registrarlas en la sección de ajustes de GitHub.|
|Archivos ocultos de metadatos (`._*`) al trabajar con pendrives|Al utilizar sistemas de archivos externos (macOS), se generan archivos automáticos de metadatos que interfieren en los permisos del contenedor.|Limpiar los archivos generados ejecutando find . -name `._*` -delete y ajustar permisos con chown.|
|No se puede abrir la carpeta del proyecto dentro del contenedor en Visual Studio Code|Visual Studio Code no está conectado al contenedor en ejecución o no está instalada la extensión necesaria.|Instalar la extensión `Dev Containers` y utilizar `Command + Shift + P` → `Dev Containers: Attach to Running Container...` → seleccionar `/dpl-lab` y abrir la carpeta del proyecto desde VS Code.|
Fallo en MkDocs Build (--strict)|El archivo de índices docs/index.md contenía enlaces rotos o referencias absolutas incompatibles con la estructura estricta del linter.|Simplificar las rutas relativas dentro de mkdocs.yml y docs/index.md apuntando de forma correcta a index.md.|
|Incompatibilidad de Fork propio en GitHub|	GitHub no permite hacer un fork de un repositorio de tu propia cuenta de usuario.|Implementar la modalidad individual con repositorio espejo (git-work-espejo), conectando las dos carpetas locales (ae1 y ae1-user2) mediante los remotos correspondientes[cite: 2].|




### Uso de pendrive para trabajar desde diferentes localizaciones y/o dispositivos.

Al trabajar desde un pendrive para poder trabajar desde casa o desde clase puede surgir algún problema en la preparación del entorno:

En el caso de macOS, al ejecutar el comando en un pendrive con un sistema de archivos macOS, se generan archivos ocultos de metadatos "._*" que pueden inducir a errores o a denegación de permisos.

```bash
chown -R alumno:alumno /home/alumno/dpl 
```

Salida:
```docker
root@8a5ff1d71003:/# chown -R alumno:alumno /home/alumno/dpl
chown: changing ownership of '/home/alumno/dpl/._README.md': Operation not permitted
```

Para solucionarlo se ejecuta este comando:

```bash
find . -name "._*" -delete
```
<br>

### Problema al clonar repositorio

```bash
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

El error se debe a que la clave privada y pública (id_ed25519) aún no se han creado dentro de la carpeta del usuario alumno en el contenedor, o la ruta no existe.

Para solucionarlo:
```bash
ssh-keygen -t ed25519 -C "alumno@dpl.local"   # genera el par de claves (passphrase vacía o de práctica)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub                     # copia esta clave pública en GitHub/GitLab
ssh -T git@github.com                         # comprueba la autenticación
git clone git@github.com:usuario/repositorio.git
```

Puede ser que al generar la clave, el usuario alumno no tenga permiso para confirmar la ruta, por lo que se debe salir del contenedor e iniciarlo en root, se ejecuta el siguiente comando como `root`:
```bash
chown -R alumno:alumno /home/alumno
```

Cambia al usuario `alumno` con el comando ```su - alumno``` y ejecuta los comandos anteriores.

Una vez generada la clave, crea una nueva SSH key en GitHub en Profile > Settings > SSH and GPG keys, pega la clave generada.

<br>

### Abrir carpeta contenedera del código y contenido del proyecto en Visual Studio Code

En el vscode del equipo antitrión se debe instalar la extensión `Dev Containers`, pulsa Comand + Shift + P introduce `Dev Containers: Attach to Running Container...` y selecciona `/dpl-lab`. Luego abre la carpeta desde vscode.

<br>


