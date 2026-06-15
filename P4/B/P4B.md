# Sistemas Operativos Práctica 4B

## Docker y Docker Compose

## Docker

### 1. Utilizando sus palabras, describa qué es Docker y enumere al menos dos beneficios que encuentre para el concepto de contenedores.

Docker es una plataforma open-source que permite empaquetar y ejecutar una aplicación en containers livianos. Utiliza conceptos de cgroups y namespaces para proveer aislamiento y control de recursos.

Docker utiliza una arquitectura cliente-servidor, comunicandose a través de la REST API de Docker. El daemon de Docker escucha las request a esa API.

Algunos beneficios son:

- Provee una virtualización liviana, ya que los contenedores no tiene un SO completo aparte (no requieren hypervisor y son tratados como procesos por el SO host).
- Poseen todo lo necesario para que la aplicación o conjunto de ellas funcione, de forma aislada. Poseen un booteo más veloz. Por ello son portables.

### 2. ¿Qué es una imagen? ¿Y un contenedor? ¿Cuál es la principal diferencia entre ambos?

Una imagen es el template o molde de solo lectura con todas las instrucciones para construir un contenedor.

El contenedor es la instancia de la imagen en ejecución.

La diferencia es que la imagen es estática y provee la receta para crear un contenedor, mientras que este es la imagen en ejecución, por lo que es dinámico.

Como se explicará en el siguiente punto, los contenedores poseen una capa escribible, lo que constituye la principal diferencia entre un contenedor y una imagen. Cuando se elimina un contenedor, esa capa también se remueve.

### 3. ¿Qué es Union Filesystem? ¿Cómo lo utiliza Docker?

Es un mecanismo de montajes, no un nuevo file system. Permite que varios directorios sean montados en el mismo punto de montaje, apareciendo como un único fs. Las capas inferiores son read-only, mientras que la superior es escribible.

Cada imagen se compone de una serie de capas, donde solo la última (capa del container) es escribible. Esta permite almacenar datos generados durante la ejecución del contenedor. Las capas pueden ser reutilizables entre imágenes.

Al ejecutar el contenedor desde una imagen, se genera un union-filesystem donde las capas se apilan una sobre otra. Usando `chroot`, se establece el union-filesystem creado como directorio raíz del contenedor. Por último, se crea un nuevo directorio para el contenedor que permite modificar el filesystem (la capa escribible del contenedor, ya mencionada).

___

Cuando se dice que las imágenes Docker tienen capas se refiere a que la imagen no es un único archivo gigante, sino una colección de carpetas independientes empaquetadas.

Las capas son directorios que almacenan únicamente los cambios (archivos agregados, modificados o eliminados) producidos por una acción en el sistema (un comando de Docker). La imagen es el conjunto ordenado de estas capas. Estas acciones del sistema son las intrucciones de Docker (RUN, COPY, ADD) que le piden al motor de Docker que realice una modificación en el sistema de archivos.

Cada instrucción de modificación de archivos en el Dockerfile genera un diff que se almacena de forma aislada en su propia capa, permitiendo que si se cambia el código Docker no tenga que volver a procesar todas las demás capas, reutilizando almacenamiento.

Ej.:

El siguiente es un Dockerfile, que indica como crear una imagen.

```bash
FROM ubuntu:24.04        # Instrucción 1 (Base)
RUN apt-get update       # Instrucción 2
COPY app.py /app/        # Instrucción 3
```

- Capa 1 (FROM): contiene la estructura básica de Ubuntu.
- Capa 2 (RUN): no vuelve a copiar Ubuntu. Solo contiene archivos de la caché de paquetes descargados.
- Capa 3 (COPY): solo tiene el directorio `app/` con `app.py`.

Cada capa en la imagen representa una instrucción en el Dockerfile. Cuando un comando modifica el filesystem, se genera una nueva capa. Si se remueve un archivo, se genera una nueva capa pero el archivo sigue existiendo en la capa anterior (el Union fs lo oculta para que el contenedor no lo vea, pero no lo elimina porque las capas inferiores son de solo lectura).

La imagen le dice a Docker "esta imagen está constituida por la Capa 1 + Capa 2 + Capa 3, en ese orden estricto".

### 4. ¿Qué rango de direcciones IP utilizan los contenedores cuando se crean? ¿De dónde la obtiene?

Por defecto, Docker asigna direcciones IP privadas a los contenedores desde redes vituales que crea automáticamente en el host. El rango por defecto es `172.17.0.0/16`. A medida que se crean contenedores, asigna en este rango. Si se crean redes adicionales, suelen crearse subredes de bloques privados (ej.: `172.18.0.0/16`).

Docker utiliza un componente interno llamado IPAM (IP Address Managment) para administrar subredes y direcciones IP. Cuando crea una red busca bloques privados disponibles, evitando conflictos con redes ya existentes en el host. Reserva una subred para la nueva red Docker y asigna automáticamente una IP libre a cada contenedor nuevo conectado.

Si bien los bloques pueden pertenecer a los rangos privados definidos en el estándar, suele priorizarse `172.16.0.0/12`

### 5. ¿De qué manera puede lograrse que las datos sean persistentes en Docker? ¿Qué dos maneras hay de hacerlo? ¿Cuáles son las diferencias entre ellas?

Como se dijo antes, los cambios almacenados en la capa escribible son eliminados cuando el contenedor se remueve. Docker tiene dos opciones para almacenar datos en el host de forma persistente:

-  Volumes: almacenados en una parte del filesystem administrada por Docker. Mejor para proveer portabilidad entre sistemas. Si se cambia de máquina, Docker crea los volúmenes automáticamente, es más fácil de migrar.

- Bind Mounts: pueden estar en cualquier parte del fs. Pueden ser modificados por procesos que no sean Docker. No es tan portable.

Ambos deben ser montados en el contenedor.

### Taller:

El siguiente taller le guiará paso a paso para la construcción de una imagen Docker utilizando dos mecanismos distintos para los cuales deberá investigar y documentar qué comandos y argumentos utiliza para cada caso.

### 1. Instale Docker CE (Community Edition) en su sistema operativo. Ayuda: seguir las instrucciones de la página de Docker. La instalación más simple para distribuciones de GNU/Linux basadas en Debian es usando los repositorios.

### 2. Asegúrese de agregar al usuario con el que trabajará al grupo docker, por ejemplo en la VM, luego de esta operación deberá cerrar la sesión del usuario so en la terminal y volver a loguearse:

`~# adduser so docker`

### 3. Luego de volver a loguearse verifique que el usuario pertenece al grupo docker:

```
~$ ssh so@192.168.56.104
so@192.168.56.104's password:
Linux so 6.1.0-31-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.128-1
(2025-02-07) x86_64
The programs included with the Debian GNU/Linux system are free
software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Apr 18 22:10:44 2026 from 192.168.56.1
so@so:~$ groups
so cdrom floppy audio dip video plugdev users netdev bluetooth docker
```

### 4. Usando las herramientas (comandos) provistas por Docker realice las siguientes tareas:

**a. Obtener una imagen de la última versión de Ubuntu disponible. ¿Cuál es el tamaño en disco de la imagen obtenida? ¿Ya puede ser considerada un contenedor? ¿Qué significa lo siguiente: Using default tag: latest?**

La imagen se obtiene con `docker pull ubuntu`. La salida tiene la línea `Using default tag: latest`. Esto se debe a que, como no fue especificada una versión de Ubuntu en el comando, Docker asume el tag "latest" y trae la versión estable más reciente.

Con `docker images` pueden verse las imágenes, donde vemos que ocupa 160MB en disco.

Aún no puede ser considerada un contenedor. La imagen es simplemente un molde estático, inmutable y de solo lectura que contiene el sistema de archivos empaqueta. El contenedor es una instancia de la imagen.

**b. De la imagen obtenida en el punto anterior iniciar un contenedor que simplemente ejecute el comando ls -l.**

Se hace con `docker run ubuntu ls -l`. El contenedor se enciende, ejecuta el comando `ls -l` mostrando el listado de archivos de la raíz de la imagen de Ubuntu y luego se detiene. Esto ocurre porque un contenedor solo vive mientras el proceso principal que se le ordenó ejecutar esté activo.

**c. ¿Qué sucede si ejecuta el comando `docker [container] run ubuntu /bin/bash` (1)? ¿Puede utilizar la shell Bash del contenedor?**

El comando parece no hacer nada y devuelve el control al host. No se puede utilizar la shell Bash. Esto se debe a que Bash es un proceso interactivo que requiere una terminal de entrada de texto y de salida. Al ejecutar el comando a secas, Docker lanza Bash, pero como no hay canales de comunicación abiertos, Bash detecta el fin de archivo (EOF) inmediatamente y finaliza de forma abrupta, provocando el apagado del contenedor.

*i. Modifique el comando utilizado para que el contenedor se inicie con una terminal interactiva y ejecutarlo. ¿Ahora puede utilizar la shell Bash del contenedor? ¿Por qué?*

Se ejecutó `docker run -it ubuntu /bin/bash`. Ahora se puede utilizar la shell Bash del contenedor. El argumento `-i` (interactive) mantiene abierto el canal de entrada estándar (stdin) del contenedor y el argumento `-t` (tty) asigna una terminal virtual.

*ii. ¿Cuál es el PID del proceso bash en el contenedor? ¿Y fuera de éste?*

Se ejecuta `ps -ef`. Dentro del contenedor el PID de `/bin/bash` es 1. Fuera de este el PID es 8068.

*iii. Ejecutar el comando lsns. ¿Qué puede decir de los namespace?*

Los identificadores de namespaces (PID, NET, MNT, UTS, IPC) son diferentes de los del SO host. El proceso en el contenedor se encuentra en un entorno de aislamiento total a nivel de kernel.

*iv. Dentro del contenedor cree un archivo con nombre sistemas-operativos en el directorio raíz del filesystem y luego salga del contenedor (finalice la sesión de Bash utilizando las teclas Ctrl + D o el comando exit).*

*v. Corrobore si el archivo creado existe en el directorio raíz del sistema operativo anfitrión (host). ¿Existe? ¿Por qué?*

El archivo no existe en el root del SO host. Esto se debe a que el archivo fue creado dentro de la capa de escritura del contenedor específico, la cual está aislada del sistema de archivos de la máquina host gracias al namespace de montaje (mnt).

**d. Vuelva a iniciar el contenedor anterior utilizando el mismo comando (con una terminal interactiva). ¿Existe el archivo creado en el contenedor? ¿Por qué?**

Tampoco existe. Esto se debe a que se corrió un nuevo contenedor limpio a partir de la imagen, por lo que contiene los archivos definidos en las capas de solo lectura de la misma. El archivo quedó guardado en el contenedor viejo, que ahora está apagado. Si el contenedor se eliminara, ese archivo se perdería para siempre.

**e. Obtenga el identificador del contenedor (container_id) donde se creó el archivo y utilícelo para iniciar con el comando docker start -ia container_id el contenedor en el cual se creó el archivo.**

*i. ¿Cómo obtuvo el container_id para para este comando?*

Los identificadores de los contenedores se obtienen con `docker ps -a`.

*ii. Chequee nuevamente si el archivo creado anteriormente existe. ¿Cuál es el resultado en este caso? ¿Puede encontrar el archivo creado?*

Una vez con el id, se inicia el contenedor con `docker start -ia <container-id>`. Si el archivo existe. Esto sucede porque `docker start` despierta al contenedor preexistente, conservando su capa de lectura/escritura intacta.

**f. ¿Cuántos contenedores están actualmente en ejecución? ¿En qué estado se encuentra cada uno de los que se han ejecutado hasta el momento?**

Se verifica con `docker ps -a`. Hay 0 contenedores en ejecución. Todos están en estado Exited y el último indica Up x minutes.

**g. Elimine todos los contenedores creados hasta el momento. Indique el o los comandos utilizados.**

Se pueden eliminar uno a uno con `docker rm <container-id>` o de forma másiva con `docker container prune -f` o `docker rm $(docker ps -aq)`

### 5. Creación de una imagen a partir de un contenedor. Siguiendo los pasos indicados a continuación genere una imagen de Docker a partir de un contenedor:

**a. Inicie un contenedor a partir de la imagen de Ubuntu descargada anteriormente ejecutando una consola interactiva de Bash.**



**b. Instale el servidor web Nginx, https://nginx.org/en/, en el contenedor utilizando los siguientes comandos (2):**

```
export DEBIAN_FRONTEND=noninteractive
export TZ=America/Buenos_Aires
apt update -qq
apt install -y --no-install-recommends nginx
```

**c. Salga del contenedor y genere una imagen Docker a partir de éste. ¿Con qué nombre se genera si no se especifica uno?**

Primero obtenemos su id con `docker ps -a`. Despues usamos `docker commit <container-id>`. Consultamos con `docker images -a` y vemos que se genera con el nombre `<untagged>`. Repository se refiere al nombre de la imagen y tag a la versión. En mi caso aparece todo en la columna IMAGE.

**d. Cambie el nombre de la imagen creada de manera que en la columna Repository aparezca nginx-so y en la columna Tag aparezca v1.**

Se obtiene el id del contenedor y se cambia con `docker tag <container-id> <nombre:tag>`.

___

Aclaraciones:

(1) Los corchetes indican que el argumento container es opcional, pero no son parte del comando a ejecutar.

(2) Los dos primeros comandos exportan dos variables de ambiente para que la instalación de una de las dependencias de nginx (el paquete tzdata) no requiera que interactivamente se respondan preguntas sobre la ubicación geográfica a utilizar

___

**e. Ejecute un contenedor a partir de la imagen nginx-so:v1 que corra el servidor web nginx atendiendo conexiones en el puerto 8080 del host, y sirviendo una página web para corroborar su correcto funcionamiento. Para esto:**

*I. En el Sistema Operativo anfitrión (host) sobre el cual se ejecuta Docker crear un directorio que se utilizará para este taller. Éste puede ser el directorio nginx-so dentro de su directorio personal o cualquier otro directorio - para los fines de este enunciado haremos referencia a éste como /home/so/nginx-so, por lo que en los lugares donde se mencione esta ruta usted deberá reemplazarla por la ruta absoluta al directorio que haya decidido crear en este paso.*

*II. Dentro de ese directorio, cree un archivo llamado index.html que contenga el código HTML de este gist de GitHub: `https://gist.github.com/ncuesta/5b959fce1c7d2ed4e5a06e84e5a7efc8`.*

*III. Cree un contenedor a partir de la imagen nginx-so:v1 montando el directorio del host (/home/so/nginx-so) sobre el directorio /var/www/html del contenedor, mapeando el puerto 80 del contenedor al puerto 8080 del host, y ejecutando el servidor nginx en primer plano (3). Indique el comando utilizado.*

Se creó con el comando `docker run -p 8080:80 -v /home/so/nginx-so:/var/www/html nginx-so:v1 nginx -g 'daemon off;'`

Donde:

- `-p 8080:80`: mapea el puerto 8080 del host al 80 interno del contenedor.
- `-v /home/so/nginx-so:/var/www/html`: realiza un montake de tipo Bind Mount, vinculando la carpeta del host `/home/so/nginx-so` sobre la carpeta `/var/www/html` de Nginx en el contenedor.
- `nginx -g 'daemon off`: ejecuta en primer plano.

**f. Verifique que el contenedor esté ejecutándose correctamente abriendo un navegador web y visitando la URL http://localhost:8080**

**g. Modifique el archivo index.html agregándole un párrafo con su nombre y número de alumno. ¿Es necesario reiniciar el contenedor para ver los cambios?**

No es necesario ya que se usó un Bind Mount (`-v`). Este es un método que permite linkear un directorio específico de la máquina host directo a un directorio en el contenedor, permitiendo sincronización en tiempo real entre ambos.

**h. Analice: ¿por qué es necesario que el proceso nginx se ejecute en primer plano? ¿Qué ocurre si lo ejecuta sin -g 'daemon off;'?**

Como se indicó antes, un contenedor vive mientras el proceso principal que lo inició esté activo. Por defecto, Nginx nace, genera procesos hijos para atender conexiones y se mueve a segundo plano, terminando el proceso de arranque inicial. Si se ejecuta sin `-g 'daemon off;'`, el proceso inicial finaliza de inmediato, lo que causa que Docker asuma que el contenedor terminó sus tareas y proceda a apagarlo automáticamente.

### 6. Creación de una imagen Docker a partir de un archivo Dockerfile. Siguiendo los pasos indicados a continuación, genere una nueva imagen a partir de los pasos descritos en un Dockerfile.

**a. En el directorio del host creado en el punto anterior (/home/so/nginx-so), cree un archivo Dockerfile que realice los siguientes pasos:**

*i. Comenzar en base a la imagen oficial de Ubuntu.*

*ii. Exponer el puerto 80 del contenedor.*

*iii. Instalar el servidor web nginx.*

*iv. Copiar el archivo index.html del mismo directorio del host al directorio /var/www/html de la imagen.*

*v. Indicar el comando que se utilizará cuando se inicie un contenedor a partir de esta imagen para ejecutar el servidor nginx en primer plano: nginx -g 'daemon off;'. Use la forma exec (4) para definir el comando, de manera que todas las señales que reciba el contenedor sean enviadas directamente al proceso de nginx. Ayuda: las instrucciones necesarias para definir los pasos en el Dockerfile son FROM, EXPOSE, RUN, COPY y CMD.*

Dockerfile generado:

```dockerfile
# i. Comenzar en base a la imagen oficial de Ubuntu
FROM ubuntu:latest

# iii. Instalar el servidor web nginx
RUN apt-get update && apt-get install -y --no-install-recommends nginx && rm -rf /var/lib/apt/lists/*

# iv. Copiar el archivo index.html del host a la imagen
COPY index.html /var/www/html/

# ii. Exponer el puerto 80 del contenedor
EXPOSE 80

# v. Ejecutar nginx en primer plano usando la forma exec
CMD ["nginx", "-g", "daemon off;"]
```

**b. Utilizando el Dockerfile que generó en el punto anterior construya una nueva imagen Docker guardándola localmente con el nombre nginx-so:v2.**

Se hace con el comando `docker build -t <nombre:tag> <context>`. En este caso, sobre `/home/so/nginx-so` se ejecuta `docker build -t nginx-so:v2 .`. "." indica que se tomará el Dockerfile del actual directorio.

**c. Ejecute un contenedor a partir de la nueva imagen creada con las opciones adecuadas para que pueda acceder desde su navegador web a la página a través del puerto 8090 del host. Verifique que puede visualizar correctamente la página accediendo a http://localhost:8090.**

Se hizo con `docker run -d -p 8090:80 nginx-so:v2`.

**d. Modifique el archivo index.html del host agregando un párrafo con la fecha actual y recargue la página en su navegador web. ¿Se ven reflejados los cambios que hizo en el archivo? ¿Por qué?**

No, el cambio no se vio reflejado. Esto se debe a que no se usó el Bind Mount (`-v`), sino que se copió el archivo al directorio del contenedor. Por lo tanto, el archivo editado en el host anfitrion y el del contenedor son distintos. El archivo en el contenedor forma parte de las capas inmutables de la imagen de nginx-so:v2.

**e. Termine el contenedor iniciado antes y cree uno nuevo utilizando el mismo comando. Recargue la página en su navegador web. ¿Se ven ahora reflejados los cambios realizados en el archivo HTML? ¿Por qué?**

Tampoco se ven reflejados los cambios, porque el nuevo contenedor nace a partir de la imagen estática nginx-so:v2, la cual sigue conteniendo la versión vieja del archivo `index.html` grabada en su estructura de capas.

**f. Vuelva a construir una imagen Docker a partir del Dockerfile creado anteriormente, pero esta vez dándole el nombre nginx-so:v3. Cree un contenedor a partir de ésta y acceda a la página en su navegador web. ¿Se ven reflejados los cambios realizados en el archivo HTML? ¿Por qué?**

Esta vez si se ve reflejado el cambio, ya que la nueva imagen fue creada copiando el archivo ya modificado, generando una nueva capa de imagen actualizada para la versión v3.

___

Aclaraciones:

(3) Para iniciar el servidor nginx en primer plano utilice el comando nginx -g 'daemon off;'

(4) La documentación oficial de Docker describe las tres formas posibles para indicar el comando principal de una imagen: https://docs.docker.com/engine/reference/builder/#cmd

___


## Docker Compose

### 1. Utilizando sus palabras describa, ¿qué es docker compose?

Dcoker Compose es una herramienta de orquestación local que permite definir y gestionar aplicaciones de contenedores múltiples. En lugar de ejecutar manualmente una larga secuencia de comandos `docker run` individuales con configuraciones complejas de redes, volúmenes y variables de entorno para cada componente de la aplicación (base de datos, backend, frontend), Docker Compose permite unificar y automatizar todo el ciclo de vida de la infraestructura de la aplicación en un único entorno coordinado, encendiendo o apagando toda la arquitectura con un solo comando.

Básicamente facilita el despliegue de aplicaciones compuestas por múltiples contenedores.

### 2. ¿Qué es el archivo compose y cual es su función? ¿Cuál es el “lenguaje” del archivo?

El archivo Compose (nombrado `docker-compose.yml` o  `compose.yaml`) es un archivo de configuración declarativo. Su función es actuar como la receta donde se describe detalladamente el detalle de toda la infraestructura de la aplicación: que contenedores se necesitan, cómo se comunican, que discos usan y que límites de hardware poseen.

Está escrito en YAML, con una estructura indentada por espacios de clave-valor y listas.

### 3. ¿Cuáles son las versiones existentes del archivo docker-compose.yaml existentes y qué características aporta cada una? ¿Son compatibles entre sí?¿Por qué?

No confundir versión de Docker Compose (V1 obsoleta y V2 actual) con versión de docker-compose.yaml (v1, v2.x y v3.x).

Hoy en día se usa Docker Compose V2 (comando `docker compose`, sin el guión). En los archivos no se aclara la versión, aunque en archivos viejos suele estar y funciona.

En archivos .yaml v3.x es la última versión numerada, antes de la Compose Specification que decidió quitar la obligatoriedad de indicar la versión en el archivo.

Existen 3 grandes líneas de versiones de archivos docker-compose.yaml:

- Versión 1 (Legado): la versión original. No requería declarar una versión al inicio. Todos los contenedores se creaban en una única red por defecto y no soportaba características avanzadas de volúmenes o configuraciones de clúster.

- Versión 2 (Orientada a Entornos Locales): Introdujo la declaración explícita version: '2'. Aportó la capacidad de definir redes personalizadas (networks) y volúmenes (volumes) como bloques independientes, permitiendo topologías de red complejas en una misma máquina física.

- Versión 3 (Orientada a Clúster/Swarm): Introdujo la directiva deploy para orquestar contenedores a través de múltiples servidores físicos usando Docker Swarm. Removió algunos controles de hardware locales de la v2 (como límites estrictos de memoria por CPU en el bloque común) a favor de la escalabilidad distributiva.

La v1 de los archivos .yaml está obsoleta. Las versiones v2.x, v3.x no son completamente compatibles, ya que hay propiedades de v2.x que fueron eliminadas en v3.x, por lo que utilizarlas podría causar un error.

A partir de las versiones modernas de Docker (2020 en adelante), se unificaron los esquemas en la Compose Specification. Hoy en día ya no es necesario escribir la etiqueta version: '3.x' al principio del archivo. El motor de Compose lee el archivo de forma dinámica y habilita las características disponibles según la versión de Docker instalada.

### 4. Investigue y describa la estructura de un archivo compose. Desarrolle al menos sobre los siguientes bloques indicando para qué se usan:

**a. services**

Bloque raíz principal. Contiene la definición de cada uno de los contenedores individuales que formarán parte de la aplicación.

**b. build**

Se utiliza cuando el contenedor no se descarga de internet, sino que debe ser construido localmente. Especifica la ruta del directorio del host que contiene el archivo Dockerfile

**c. image**

Indica el nombre de la imagen que se utilizará para levantar el contenedor. Si está junto al bloque build, define el nombre que recibirá la imagen resultante tras compilarse; si está solo, le ordena a Compose descargar la imagen desde un registro como Docker Hub.

**d. volumes**

Permite montar almacenamiento persistente. Puede declarar un Bind Mount (vincular una carpeta específica del host al contenedor) o un Named Volume (un volumen gestionado por Docker) para que los datos no se pierdan al apagar el entorno.

**e. restart**

Define la política de reinicio del contenedor administrada por el demonio de Docker ante fallos. Sus valores comunes son no (por defecto), always (reiniciar siempre si se cae), on-failure (solo si falla con un código de error distinto de cero) y unless-stopped.

**f. depends_on**

Establece el orden de arranque y parada de los servicios. Si el servicio backend tiene depends_on: - db, Compose se asegurará de crear y encender el contenedor db antes de intentar iniciar el contenedor backend.

**g. environment**

Permite inyectar variables de entorno dentro del espacio de usuario del contenedor. Se utiliza para pasar credenciales, llaves de API o configuraciones dinámicas al software en ejecución (por ejemplo, MYSQL_ROOT_PASSWORD=secreto).

**h. ports**

Mapea puertos entre la máquina anfitriona (host) y el contenedor mediante la sintaxis HOST:CONTAINER (ej. 8080:80). Expone el puerto al exterior del host permitiendo tráfico desde redes externas.

**i. expose**

Expone puertos del contenedor, pero únicamente para los demás servicios dentro de la misma red virtual de Docker. No los mapea hacia la máquina física host, protegiendo servicios internos (como bases de datos) del acceso externo general.

**j. networks**

Define las redes virtuales aisladas a las cuales se acoplarán los contenedores. Permite agrupar servicios para que se comuniquen entre sí mediante DNS interno y aislarlos de otros contenedores que compartan el mismo host físico.

### 5. Conceptualmente: ¿Cómo se podrían usar los bloques “healthcheck” y “depends_on” para ejecutar una aplicación Web dónde el backend debería ejecutarse si y sólo si la base de datos ya está ejecutándose y lista?

Por defecto, depends_on solo espera a que el contenedor de la base de datos esté en estado "ejecutándose" (running) a nivel de proceso de kernel. Sin embargo, una base de datos como PostgreSQL o MySQL puede tardar varios segundos en inicializar sus tablas internas y escuchar conexiones, tiempo durante el cual el backend intentará conectarse, fallará y morirá.

Para solucionar esto, se combinan ambos bloques extendiendo la sintaxis de la siguiente manera:

1. En la Base de Datos (db): Se configura un bloque healthcheck. Este le indica a Docker un comando interno que debe ejecutar periódicamente (por ejemplo, mysqladmin ping o un script de conexión) para verificar que la base de datos no solo esté corriendo, sino que esté "lista y saludable".

2. En la Aplicación Web (backend): Dentro del bloque depends_on, en lugar de pasar una lista simple, se especifica una condición estricta:

```yaml
services:
  db:
    image: mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 3

  backend:
    image: mi-app-web
    depends_on:
      db:
        condition: service_healthy  # <- Condición crítica de bloqueo
```

Mecanismo resultante: Al hacer compose up, Docker encenderá el contenedor db. El contenedor backend se mantendrá en estado de espera bloqueado hasta que el healthcheck de db cambie exitosamente su estado de estatus a healthy (saludable). En ese instante, Compose libera el bloqueo e inicia el backend de forma segura.

### 6. Indique qué hacen y cuáles son las diferencias entre los siguientes comandos:

**a. docker compose create y docker compose up**

- `create`: Lee el archivo YAML y prepara las estructuras en el sistema (descarga imágenes, crea contenedores, configura redes y volúmenes virtuales), pero los deja en estado apagado/detenido.

- `up`: Hace todo lo anterior (si no está creado) y enciende los contenedores de inmediato (start), acoplando los canales de log a la terminal a menos que se use el flag -d.

**b. docker compose stop y docker compose down**

`stop`: Envía una señal SIGTERM para pausar/detener la ejecución de los procesos dentro de los contenedores. Los contenedores físicos se quedan guardados en el disco en estado Exited, manteniendo sus modificaciones intactas.

`down`: Va mucho más allá. Detiene los procesos y DESTRUYE por completo los contenedores, las redes y los layouts virtuales creados para ese archivo. Libera por completo los recursos de memoria y espacio de disco de ejecución del host.

**c. docker compose run y docker compose exec**

docker compose run vs docker compose exec

- `run`: Se usa para crear un nuevo contenedor temporal a partir de la definición de un servicio y ejecuta un comando específico en él (por ejemplo, docker compose run backend python manage.py migrate).

- `exec`: No crea nada nuevo. Se mete dentro de un contenedor que ya está encendido y corriendo en ese momento para ejecutar un proceso de diagnóstico (por ejemplo, abrir una terminal interactiva: docker compose exec backend bash).

**d. docker compose ps**

Muestra una lista simplificada y formateada únicamente de los contenedores que pertenecen a ese entorno de Compose específico, indicando su ID, el comando principal, su estado actual (Up, Exited) y los puertos expuestos.

**e. docker compose logs**

Funde los flujos de salida estándar (stdout y stderr) de todos los contenedores del entorno y los imprime cronológicamente en tu pantalla. Te permite ver la traza de errores del sistema sin necesidad de revisar contenedor por contenedor.

**nota:**

Entorno de Compose se refiere al límite lógica que agrupa a un conjunto específico de contenedores, redes y volúmenes definidos en un archivo YAML. Básicamente son todos los contenedores del archivo. Estos comparten directorio base y pertenecen a la misma red privada virtual

### 7. ¿Qué tipo de volúmenes puede utilizar con docker compose? ¿Cómo se declara cada tipo en el archivo compose?

Docker Compose puede utilizar principalmente los dos tipos de persistencia de datos clásicos de Docker:

> Tipo 1: Named Volumes (Volúmenes Gestionados)

Docker administra una carpeta reservada en el sistema de archivos interno del host (habitualmente en /var/lib/docker/volumes/). El usuario no se preocupa por las rutas físicas de su máquina.

Se deben registrar en un bloque global de raíz volumes al final del archivo y luego invocarlos dentro del servicio:

```yaml
services:
  db:
    image: postgres
    volumes:
      - mi-data-postgres:/var/lib/postgresql/data

volumes:
  mi-data-postgres: # <- Declaración global del volumen nombrado
```

> Tipo 2: Bind Mounts (Montajes de Directorio)

Vinculan de forma explícita una carpeta con una ruta absoluta o relativa específica de tu máquina física (host) hacia el contenedor.

Se definen directamente dentro del servicio mapeando la ruta del host a la del contenedor utilizando puntos (.) para rutas relativas:

```yaml
services:
  web:
    image: nginx
    volumes:
      - ./html-local:/var/www/html # <- Montaje directo de directorio local
```

### 8. ¿Qué sucede si en lugar de usar el comando `docker compose down` utilizo `docker compose down -v/--volumes`?

Si usas `docker compose down` a secas, Docker destruye los contenedores y las redes, pero respeta y preserva los Named Volumes creados de forma global. La próxima vez que ejecutes docker compose up, la base de datos se encenderá conservando toda la información histórica que tenía guardada.

Si agregas el flag `-v` o `--volumes`, Docker procederá a ejecutar una limpieza destructiva absoluta: además de borrar los contenedores y redes, borrará y eliminará del disco duro todas las carpetas físicas de los Named Volumes asociados al archivo.

## Ejercicio guiado - Instanciando un Wordpress y una Base de Datos

Dado el siguiente código de archivo compose:

```yaml
version: "3.9"

services:
db:
    image: mysql:5.7
    networks:
        - wordpress
    volumes:
        - db_data:/var/lib/mysql
    restart: always
    environment:
        MYSQL_ROOT_PASSWORD: somewordpress
        MYSQL_DATABASE: wordpress
        MYSQL_USER: wordpress
        MYSQL_PASSWORD: wordpress
wordpress:
    depends_on:
        - db
    image: wordpress:latest
    networks:
        - wordpress
    volumes:
        - ${PWD}:/data
        - wordpress_data:/var/www/html
    ports:
        - "127.0.0.1:8000:80"
    restart: always
    environment:
        WORDPRESS_DB_HOST: db
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpress
        WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
    wordpress_data: {}
networks:
    wordpress:
```

## Preguntas

Intente analizar el código ANTES de correrlo y responda:

### ¿Cuántos contenedores se instancian?

Se instancian dos contenedores, correspondientes a los bloques definidos bajo la directiva raíz `services`. Corresponde a `db` y `wordpress`.

### ¿Por qué no se necesitan Dockerfiles?

No se necesitan porque ambos servicios utilizan imágenes oficiales precompiladas y listas para usar (mysql:5.7 y wordpress:latest) directamente desde el registro público de Docker Hub.

El Dockerfile solo es obligatorio cuando necesitas compilar código propio o personalizar profundamente el sistema de archivos del contenedor antes de ejecutarlo.

### ¿Por qué el servicio identificado como “wordpress” tiene la siguiente línea?

```bash
depends_on:
    - db

```

Establece el orden secuencial de inicio de los servicios. Wordpress es una aplicación web que requiere la base de datos para funcionar. Si intenta arrancar antes que esta, fallará. Con esta directiva, le ordenamos a Docker Compose que cree y encienda primero el contenedor db, y una vez que su proceso esté corriendo, proceda a iniciar el contenedor wordpress.

### ¿Qué volúmenes y de qué tipo tendrá asociado cada contenedor?

El entorno tendrá asociados tres montajes en total, divididos en dos tipos:

> Named Volumes (Volúmenes Nombrados): Gestionados automáticamente por Docker en una ruta interna del host.

- Servicio db: Tiene asociado el volumen nombrado db_data, montado en la ruta interna /var/lib/mysql.

- Servicio wordpress: Tiene asociado el volumen nombrado wordpress_data, montado en la ruta interna /var/www/html.

> Bind Mount (Montaje de Directorio): Vinculación directa con una carpeta del sistema físico del host.

- Servicio wordpress: Tiene asociado el volumen ${PWD}:/data. Vincula el directorio actual de trabajo del host (PWD) con la carpeta /data interna del contenedor.

### ¿Por que uso el volumen nombrado

```bash
volumes:
    - db_data:/var/lib/mysql
```

### para el servicio db en lugar de dejar que se instancie un volumen anónimo con el contenedor?

Por una cuestión crítica de persistencia y control de los datos.

Si dejas que se instancie un volumen anónimo, Docker creará una carpeta con un hash aleatorio indescifrable en el host. Cuando ejecutes docker compose down, ese volumen anónimo se destruirá o quedará huérfano. Si vuelves a levantar el entorno, la base de datos nacerá completamente vacía y habrás perdido todo tu sitio web.

Al usar un volumen nombrado, la información de MySQL se guarda bajo el identificador único db_data. No importa cuántas veces destruyas, apagues o actualices los contenedores; los datos de tus tablas sobrevivirán intactos en el disco del host y se reasociarán automáticamente al contenedor en el próximo arranque.

### ¿Qué genera la línea

```bash
volumes:
    - ${PWD}:/data
```

### en la definición de wordpress?

Genera un Bind Mount dinámico. La variable de entorno del sistema operativo `${PWD}`  se traduce en la ruta absoluta de la carpeta exacta del host donde estás parado ejecutando el comando de Compose.

Esto mapea de forma bidireccional tu directorio de trabajo actual con la carpeta /data dentro del contenedor de WordPress. Cualquier archivo que dejes en esa carpeta desde tu máquina física aparecerá inmediatamente en el contenedor, y viceversa.

### ¿Qué representa la información que estoy definiendo en el bloque environment de cada servicio? ¿Cómo se “mapean” al instanciar los contenedores?

Representa la configuración de inicio de las aplicaciones mediante Variables de Entorno.

Durante la fase de creación del contenedor, el demonio de Docker inyecta estas claves y valores directamente en el espacio de usuario del proceso. Cuando los binarios internos de MySQL y WordPress arrancan (con PID 1), leen estas variables de la memoria RAM para autoconfigurarse. Así, MySQL sabe qué contraseña de root setear y qué base de datos crear, mientras que WordPress lee esas mismas credenciales para saber cómo autenticarse contra MySQL de forma automática.

### ¿Qué sucede si cambio los valores de alguna de las variables definidas en bloque “environment” en solo uno de los contenedores y hago que sean diferentes? (Por ej: cambio SOLO en la definición de wordpress la variable WORDPRESS_DB_NAME)

El sistema dejará de funcionar y la aplicación web mostrará el clásico error: "Error establishing a database connection".

Por ejemplo, si cambias WORDPRESS_DB_NAME a wordpress_test en el servicio de WordPress, pero dejas MYSQL_DATABASE: wordpress en el servicio db: MySQL creará una base de datos llamada wordpress, pero WordPress intentará conectarse a una llamada wordpress_test que no existe. Al no coincidir los parámetros de comunicación (apuntando a bases de datos, usuarios o contraseñas distintas), la autenticación fallará.

### ¿Cómo sabe comunicarse el contenedor “wordpress” con el contenedor “db” si nunca doy información de direccionamiento?

Se comunican a través del Servicio de DNS Interno de Docker y la Red Virtual Compartida.

Al declarar el bloque raíz networks e indicar que ambos servicios pertenecen a la red llamada wordpress, Docker Compose crea una red privada virtual de tipo bridge aislada. El motor de Docker incluye un servidor DNS interno para esa red. Cuando el contenedor de WordPress intenta conectarse al host db (especificado en WORDPRESS_DB_HOST: db), le pide al DNS de Docker que resuelva ese nombre. El DNS traduce la palabra db en la dirección IP privada dinámica interna (por ejemplo, 172.18.0.2) que el kernel le asignó al contenedor de MySQL en ese momento.

### ¿Qué puertos expone cada contenedor según su Dockerfile? (pista: navegue el sitio https://hub.docker.com/_/wordpress y https://hub.docker.com/_/mysql para acceder a los Dockerfiles que generaron esas imágenes y responder esta pregunta.)

- Dockerfile de mysql:5.7: Contiene la instrucción EXPOSE 3306. Es el puerto estándar por defecto para el motor de bases de datos MySQL.

- Dockerfile de wordpress:latest (basado en Apache): Contiene la instrucción EXPOSE 80. Es el puerto estándar de HTTP para servir páginas web.

### ¿Qué servicio se “publica” para ser accedido desde el exterior y en qué puerto? ¿Es necesario publicar el otro servicio? ¿Por qué?

Se publica únicamente el servicio wordpress a través de la directiva ports: - "127.0.0.1:8000:80". Se accede desde el exterior (el host) abriendo un navegador web e ingresando a http://localhost:8000.

El contenedor de WordPress se comunica con el contenedor de MySQL de forma interna, directa y segura de "red a red" dentro de la malla virtual de Docker (a través del puerto expuesto 3306). El usuario final solo necesita interactuar con la interfaz gráfica web de WordPress en el puerto 8000. Publicar el puerto de la base de datos hacia el exterior de la máquina física host solo serviría para abrir una brecha de seguridad innecesaria, exponiendo el motor de datos a posibles ataques externos de fuerza bruta.

Por ello, no se publica el puerto de la base de datos, solo de la web wordpress.