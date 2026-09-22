# git-work

Plantilla mínima para una página web de startup diseñada para practicar y demostrar el **flujo colaborativo profesional con Git y GitHub** (gestión de ramas, apertura de *issues*, *pull requests*, resolución de conflictos y versionado mediante *tags/releases*).

## Índice

- [Instalación](#instalacion)
- [Configuración](#configuracion)
- [Comprobación](#comprobacion)
- [Problemas encontrados y solución](#problemas-encontrados-y-solucion)

### Preparación del Entorno:

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

### Instalación:
##### Paso 1 - Crear el repositorio remoto y la copia local

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

##### Paso 2 - Añadir la página y la hoja de estilos.
```bash
mkdir -p css
# crea index.html, css/cover.css y .gitignore con el contenido anterior
git add index.html css/cover.css .gitignore
git commit -m "Añade la página de la startup y su hoja de estilos" \
           -m "Incluye index.html, css/cover.css (plantilla cover) y un .gitignore mínimo para entornos y logs."
git push origin main
git log --oneline
```

Con `nano name_file` se puede copiar y pegar el código o contenido:

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

#### Paso 3 — user1: añadir el workflow de integración continua

Con MkDocs se consigue convertir archivos MarkDown en un sitio web de documentación profesional, limpio y ordenado de forma automática.
Par añadir el workflow aplica los siguientes comandos y contenidos:

- `nano mkdocs.yml`
```yml
# mkdocs.yml
site_name: git-work
nav:
  - Inicio: index.md
```

- `nano docs/index.md`
```md
<!-- docs/index.md -->
# git-work

Documentación del repositorio colaborativo de la AE1.

- [Portada del sitio](../index.html)
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





## Configuración
Ficheros que hay que tocar y variables que hay que definir.

## Comprobación

Comprobación de versión de Docker instalada:

```mk
docker --version    # indica la versión de Docker instalada
docker info         # confirma que el dameon responde
```

Comprobación de que el contenedor está corriendo:
```bash
docker ps
```
Devolverá algo similar a esto:
```bash
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS                                                                                                                           NAMES
bf63157e3fff   ubuntu:24.04   "/bin/bash"   9 minutes ago   Up 9 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   dpl-lab
```

## Problemas encontrados y solución

##### Uso de pendrive para trabajar desde diferentes localizaciones y/o dispositivos.

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

##### Problema al clonar repositorio

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

##### Abrir carpeta contenedera del código y contenido del proyecto en Visual Studio Code
En el vscode del equipo antitrión se debe instalar la extensión `Dev Containers`, pulsa Comand + Shift + P introduce `Dev Containers: Attach to Running Container...` y selecciona `/dpl-lab`. Luego abre la carpeta desde vscode.

<br>

##### Error al construir la documentación con MkDocs.

El archivo `docs/index.md` con el siguiente contenido:
```md
<!-- docs/index.md -->
# git-work

Documentación del repositorio colaborativo de la AE1.

- [Portada del sitio](../index.html)
- [Repositorio remoto](https://github.com/USUARIO/git-work)
```

Produce un fallo, ya que `mkdocs build --strict` construye el sitio y falla si hay enlaces rotos o páginas fuera de la navegación. Por lo que hay que quitar `- [Portada del sitio](../index.html)` del contenido, para ello, ejecuta `nano docs/index.md`y elimina esa línea.

Se elimina esa línea ya que `docs/index.md` ya funciona como índice de la documentación de MkDocs.