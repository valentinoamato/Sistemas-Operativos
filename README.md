# Sistemas Operativos Práctica 1

### 1. ¿Qué es GCC?

El GNU Compiler Collection (GCC) es una colección de compiladores del proyecto GNU que soporta varios lenguajes de programación, arquitecturas de hardware y sistemas operativos. Es principalmente utilizado como compilador para C en los proyectos vinculados con GNU y el kernel de Linux.

### 2. ¿Qué es make y para que se usa?

El comando `make` en Linux es una herramienta utilizada para la automatización en el desarrollo de software. Este determina automáticamente que partes de un programa necesitan ser recompiladas y realiza dicha acción. Aun así, el comando no está limitado únicamente a programas, permitiendo ser usado para describir cualquier tarea donde ciertos archivos deban ser actualizados automáticamente a partir de otros, cuando estos últimos cambian.

Para utilizar el comando se debe crear un archivo *makefile*, que describe la relación entre los archivos en el programa que uno desarrolla y provee comandos para actualizar cada archivo. En el desarrollo de software, los ejecutables son actualizados a partir de archivos objeto, los cuales son creados mediante la compilación de código fuente. De esta forma, una vez creado el archivo *makefile* con las asociaciones correspondientes, cada vez que se realiza algún cambio en algún archivo de código fuente, el comando `make` realiza todas las pertinentes recompilaciones. `make` hace uso del archivo *makefile* y el timestamp de última modificación para decidir que archivos necesitan ser actualizados. Para ellos, lleva a cabo las acciones descritas en el *makefile*.

El uso de `make` permite ahorrar tiempo en el build de proyectos y evitar errores.

El *makefile* tiene la estructura:

```
target: dependencias
    comandos
```

Donde:

- target es lo llamado con el comando make.
- dependencias: son archivos asociados a dicha invocación.
- comandos: son las acciones que se realizan al invocar el target, utilizando las dependencias.

**Solo se ejecuta el comando si el target no existe o está desactualizado respecto a sus dependencias.**

Permite **automatizar builds basándose en dependencias y timestamps**. Permite describir cómo construir archivos a partir de otros, ejecutando comandos en caso de ser necesario.

`make` no distingue entre un target y un archivo, sino que considera que el target es un archivo. Si hay dependencia sobre un archivo `file`, entonces busca dicho archivo. Si no hay un archivo con ese nombre, busca una regla en el *makefile* para crearlo.

La secuencia al ejecutar `make` es la siguiente:

- Sin parámetros indicados, se intenta ejecutar el `all` del *makefile*. Este normalmente deriva a un target.
- Si el target representa un archivo, verifica si su timestamp es posterior al del archivo con el cual tiene dependencia. En caso de que lo sea, ejecuta el comando. Caso contrario, no lo hace.
- Si el target no representa un archivo, lo ejecuta siempre.

Si el archivo dependencia no existiera, buscaría una regla (target) con su nombre, para poder crearlo.

`.PHONY` permite indicarle a `make` que los targets señalados no representan archivos reales, sino que debe procederse directamente con la ejecución de sus comandos cuando son invocados. Permite evitar errores si hay archivos con dichos nombres.

`make` posee reglas implícitas que permiten este tipo de comportamiento. En este caso, sabe que para generar un archivo `.o`, debe compilar un archivo `.c`, por lo que busca un archivo `.c` con el mismo nombre y lo compila automáticamente. La regla es algo así como:

```
%.o: %.c
	cc -c %.c -o %.o
```

En este caso, se tiene además una regla adicional explícita que indica una dependencia `dlinkedlist.o: dlinkedlist.h`. Entonces, explícitamente `dlinkedlist.o` depende de `dlinkedlist.h` e implícitamente de `dlinkedlist.c`. Un cambio en el `.c` o `.h` produce una nueva compilación. Esta regla explícita no conflictúa con la regla implícita porque solo define una dependencia, sin cuerpo (comando). Si se definiera un comando, se sobreescribiría la regla implícita y ya no sería posible crear el archivo `dlinkedlist.o`.

En resumen, no es necesario definir una regla para crear `dlinkedlist.o`, ya que implícitamente `make` intenta crearlo compilando un archivo de igual nombre y extensión `.c`.


### 7. ¿Cuáles son los motivos por los que un usuario/a GNU/Linux puede querer re-compilar el kernel?

En resumen, un usuario podría requerir modificar el kernel para realizar ajustes en las opciones del mismo, eliminar características que no utilizará o agregar optimizaciones, por lo que requerirá recompilar el kernel.

### 8. ¿Cuáles son las distintas opciones y menús para realizar la configuración de opciones de compilación de un kernel? Cite diferencias, necesidades (paquetes adicionales de software que se pueden requerir), pro y contras de cada una de ellas.

El kernel de Linux se configura mediante el archivo `.config`. Este reside en el directorio de código fuente del kernel y contiene las instrucciones de qué es lo que el kernel debe compilar.

Existen 3 interfaces que permiten generar este archivo:

- `make config`: modo texto y secuencial. Es poco intuitivo.
- `make xconfig`: utiliza una interfaz gráfica mediante un sistema de ventanas. Puede no funcionar en todos los sistemas, ya que requiere entorno gráfico y librerías adicionales.
- `make menuconfig`: mediante la librería `ncurses`, genera una interfaz con paneles desde la propia terminal. Es el más utilizado.

Con esta herramienta se crea el archivo `.config` con las directivas de compilación. Esto permite automatizar el proceso de configuración. Por convención, el archivo de configuración suele ir en `/boot` junto con la imagen compilada del kernel. Se mantiene el `.config` para guardar configuraciones hechas y evitar realizar el proceso desde cero. Cada nueva versión puede valerse de un `.config` anterior y configurar solo los nuevos apartados.

Durante la configuración, puede determinarse si ciertas características serán built-in o formarán parte de un módulo. Un módulo es un fragmento de código que puede cargarse o descargarse en el espacio de direcciones del kernel, bajo demanda. Este se ejecuta en modo privilegiado y permite extender funcionalidad en el kernel sin reiniciar el sistema.

Si se define que una característica es built-in, esta supone un mayor uso de la memoria y puede incrementar el tiempo de arranque (pasa a formar parte del kernel). Sin embargo, su utilización es más eficiente y evita cargar un módulo adicional en memoria, el acceso es directo.
Si se define como módulo, se pierde eficiencia pero permite utilizar menos memoria, pues se carga a demanda. Además, en caso de modificación afecta únicamente al código del módulo y no al kernel en su totalidad.

Entonces, durante la configuración, cada feature puede ser seleccionado como built-in, módulo cargable o desactivarse.

Algunas configuraciones adicionales:

- `make olddefconfig`: utiliza `.config` existente y solo pregunta por nuevas opciones.
- `make defconfig`: genera configuración por defecto para la arquitectura.
- `make savedefconfig`: guarda versión mínima del `.config`.
- `make localmodconfig`: configura como built-in los módulos del kernel que se encuentran cargados en el momento, deshabilitando los módulos no utilizados.

### 9. Indique qué tarea realiza cada uno de los siguientes comandos durante la tarea de configuración/compilación del kernel:

*a. make menuconfig*

Genera una interfaz con paneles desde la terminal utilizando la librería `ncurses` para la creación del archivo `.config`.

*b. make clean*

Permite eliminar la mayoría de archivos generados, dejando los suficientes para hacer un build de los módulos externos. Ayuda a preparar el árbol de archivos para un nuevo build sin resetear la configuración.

*c. make (investigue la funcionalidad del parámetro -j)*

Compila el kernel completo siguiendo las directivas de `.config`. El parámetro `-j` permite indicar la cantidad de procesos simultáneaos para realizar procesamiento en paralelo.

*d. make modules (utilizado en antiguos kernels, actualmente no es necesario)*

Se utilizaba para compilar solo los módulos del kernel. Antigüamente se debía realizar `make` y `make modules`. Actualmente el comando `make` hace ambos.

*e. make modules_install*

Instala los módulos compilados en el sistema, ubicados en `/lib/modules/<kernel-version>/`, en el directorio correspondiente. También genera dependencias y prepara módulos para ser cargados con `modprobe`.

*f. make install*

Instala el kernel en el sistema. Realiza automáticamente la copia de la imagen del kernel, el `System.map` (archivo generado en la compilación que mantiene una tabla que mapea direcciones con símbolos, como nombres variables o funciones) y el `.config` a `/boot`.

### 10. Una vez que el kernel fue compilado, ¿dónde queda ubicada su imagen? ¿dónde debería ser reubicada? ¿Existe algún comando que realice esta copia en forma automática?

Al realizar la compilación del kernel, su imagen queda ubicada en el path relativo `arch/<arquitectura>/boot` (partiendo desde el directorio donde fue compilado. `<arquitectura>` es la arquitectura del sistema, como x86 por ejemplo).

Por convención debe ser reubicada en `/boot`. Mediante el comando `make install` se realiza esta reubicación automáticamente, además de las operaciones adicionales descritas en el inciso anterior.

### 11. ¿A qué hace referencia el archivo initramfs? ¿Cuál es su funcionalidad? ¿Bajo qué condiciones puede no ser necesario?

El `initramfs` es un sistema de archivos temporal que se monta en memoria durante el arranque del sistema. Este contiene ejecutables, drivers y módulos necesarios para lograr iniciar el sistema. Luego del proceso de arranque, este se desmonta. Permite montar el verdadero root file system.

### 12. ¿Cuál es la razón por la que una vez compilado el nuevo kernel, es necesario reconfigurar el gestor de arranque que tengamos instalado?

Los bootloader, como GRUB, son los encargados de conocer que kernels existe, brindar la opción de elegir cuál arrancar y pasarle parámetros al mismo.

Como se mencionó, luego de compilar el kernel y ejecutar `make install`, la imagen del kernel y otros archivos son copiados a `/boot`, pero el bootloader no detecta esto automáticamente. Esto produce que el nuevo kernel no aparezca en el menú, no pueda seleccionarse y, por lo tanto, no pueda bootearse. Por ello, luego de compilar el kernel es necesario realizar una reconfiguración del bootloader. Con GRUB v2, esto se lleva a cabo mediante `update-grub2`.

### 13. ¿Qué es un módulo del kernel? ¿Cuáles son los comandos principales para el manejo de módulos del kernel?

Como se definió anteriormente, un módulo es un fragmento de código que puede cargarse o descargarse en el espacio de direcciones del kernel, bajo demanda, sin necesidad de reiniciar el sistema. Este se ejecuta en modo privilegiado y permite extender funcionalidad en el kernel sin reiniciar el sistema.

Suelen ser archivos `.ko` (kernel object). Permiten agregar funcionalidad sin recompilar el kernel. Permiten mayor flexibilidad, reducir el tamaño del kernel base y agregar funcionalidad por demanda.

Algunos comandos:

- `lsmod`: lista los módulos cargados.
- `insmod <module.ko>`: inserta un módulo manualmente (no resuelve dependencias).
- `modprobe <module>`: carga módulos con resolución automática de dependencias.
- `rmmod <module>`: elimina un módulo.
- `modinfo <module>`: muestra información de un módulo.

### 14. ¿Qué es un parche del kernel? ¿Cuáles son las razones principales por las cuáles se deberían aplicar parches en el kernel? ¿A través de qué comando se realiza la aplicación de parches en el kernel?

Un parche es un conjunto de modificaciones aplicadas al código fuente del kernel para corregir, mejorar o extender su funcionalidad. Permite aplicar actualizaciones sobre una versión base. Se basa en archivos `.diff` o `.patch`, que indican qué agregar y qué quitar. Se utilizan para corregir errores, agregar funcionalidad, correcciones de seguridad, optimización, compatibilidad, etc.

Se aplican con el comando `patch`, de la forma `patch -p1 < <archivo.patch>`. Esto modifica los archivos fuente del kernel y aplica los cambios automáticamente.

El parche debe corresponder a la versión correcta del kernel. Luego de aplicarlo, se debe recompilar el kernel.

### 15. Investigue la característica Energy-aware Scheduling incorporada en el kernel 5.0 y explique brevemente con sus palabras:

El **Energy-Aware Scheduling (EAS)** es una mejora del scheluder de Lunux que introduce un criterio adicional al decidir donde ejecutar procesos. No tiene en cuenta solo el rendimiento, sino también el consumo energético. El scheduler utiliza modelos de energía del hardware para estimar cuánto costará ejecutar una tarea en cada CPU.

*a. ¿Qué característica principal tiene un procesador ARM big.LITTLE?*

Estos procesadores combinan en un mismo sistema dos tipos de núcleo: núcleos **big** (potentes, con mayor rendimiento y consumo energético) y **LITTLE** (menos potentes, menos consumo y mayor eficiencia energética).

Entonces, se caracterizan por la heterogeneidad de sus CPUs, lo que obliga al scheduler a tomar decisiones más inteligentes sobre la planificación de procesos.


### 16. Investigue la system call memfd_secret() incorporada en el kernel 5.14 y explique brevemente con sus palabras

Esta system call introduce un mecanismo específico para manejar memoria altamente sensible, con restricciones fuertes de acceso.


El propósito de esta system call es crear regiones de memoria que sean inaccesibles para otros componentes del sistema, incluso dentro del mismo kernel en ciertos contextos.

Para ello, crea un file descriptor asociado a esta memoria. Esta no es accesible por otros procesos, no puede ser accedida mediante mecanismo habituales del kernel (como lectura directa) y está protegida contra accesos accidentales o maliciosos.

El mecanismo se apoya en la MMU para restringir accesos.

# Sistemas Operativos Práctica 2

## System Calls

## Conceptos generales

### 2. ¿Para qué sirve la macro syscall? Describa el propósito de cada uno de sus parámetros.

Existen tanto una función llamada `syscall` como una macro `_syscall`.

La función `syscall` realiza una llamada al sistema genérica. Esta tiene la forma `long int syscall(long int sysno, ...)`, donde `sysno` es el número de la system call. Cada llamada al sistema se identifica con un número. Los argumentos restantes serán aquellos que la system call que se invoca recibe, por lo que pueden variar dependiendo de la llamada que se efectúe. En caso de indicar más argumentos de los que la system call recibe, los restantes serán ignorados.
La función retorna un valor que corresponde al retorno de la system call, exceptuando que esta haya fallado. En dicho caso retornará -1.
Esta es una función de alto nivel de la `libc`.

La macro `_syscallX` invoca una system call sin soporte de la librería `libc`. Se debe conocer el prototipo de la system call a invocar, tal como cuantos argumentos recibe, sus tipos y el tipo de retorno. Tiene la forma `_syscallX(type,name,type1,arg1,type2,arg2,...)` donde la X posee un valor entre 0-6 y representa el número de argumentos que recibe la system call. `type` es el tipo de retorno y `name` es el nombre de la system call.
Esta macro permite generar funciones wrappers para las system calls que se invocan.
Esta macro es de bajo nivel pero se encuentra en desuso en nuevas versiones.

### 3. Ejecute el siguiente comando e identifique el propósito de cada uno de los archivos que encuentra: `ls -lh /boot | grep vmlinuz`

Aparecen las distintas versiones de `vmlinuz`. Este es la imagen del kernel comprimida `vmlinux`, para bootear el sistema operativo. El archivo resultante de la compresión contiene los elementos esenciales del kernel. Su tamaño reducido permite optimizar la eficiencia y el uso de memoria durante el inicio del sistema.

El archivo original (`vmlinux`) es la imagen del kernel sin comprimir. Esta no puede ser booteada. Contiene el código completo del kernel. Suele ser usado para desarrollar y debuggear el kernel.

### 5. ¿Para qué sirve el siguiente archivo? `arch/x86/entry/syscalls/syscall_64.tbl`

El archivo `arch/x86/entry/syscalls/syscall_64.tbl` corresponde a la tabla de system call para la arquitectura x86 de 64 bits. Esta tabla es una estructura de datos del kernel que mapea números de system calls a su correspondiente función en el kernel. Cada system call tiene un número único. Cuando un proceso de usuario invoca una system call, pasa este número al kernel. El kernel utiliza la tabla para encontrar la correspondiente función y ejecutarla.

El formato de las entradas de la tabla es: `<number> <abi> <name> <entry point> [<compat entry point> [noreturn]]`

- `number`: número que identifica la system call
- `abi`: Application Binary Interface, especifica la interfaz binaria y convención para invocar que sigue la system call (puediendo ser common, 64 o 32).
- `name`: nombre de la system call
- `entry point`: función del kernel vinculada a la system call 

### 6. ¿Para qué sirve la herramienta strace? ¿Cómo se usa?

`strace` es una herramienta para el diagnóstico, debugging y monitoreo de procesos en Linux, lo que incluye system calls invocadas, envío de señales y cambios en el estado del proceso. Se utiliza principalmente para debuggear programas, interceptar system calls y monitorear procesos activos. Permite obtener información acerca de como un programa interactúa con el sistema.
Resulta especialmente útil para solucionar problemas de programas para los cuales no se tiene el código fuente

Concretamente para lo concerniente a esta práctica, interesa su utilidad para rastrar system calls.

`strace` ejecuta un comando especificado hasta que este termina. Durante su ejecución, intercepta y registra todas las system calls invocadas por un proceso y las señales que recibe. Imprime el nombre de cada system call, sus argumentos y su valor de retorno.

Su forma de uso es `strace [flag] command [args]`. Por ejemplo, `strace -e trace=write ls` intercepta la system call `write` cuando es invocada por el comando `ls`.

### 7. ¿Para qué sirve la herramienta ausyscall? ¿Cómo se usa?

`ausyscall` es una herramienta que permite mapear nombres de system calls con números. Dada una arquitectura, el programa imprime el mapeo del nombre de una system call indicada a su número correspodiente o viceversa (indicando el número). Se usa de la forma: `ausyscall [arch] name | number | --dump | --exact`. `--dump` imprime la tabla entera, mientras que  `--exact` indica que se busca el string exacto y no coincidencias con substrings. Por ejemplo: `usyscall $(uname -m) write --exact ` devuelve `1` mientras que `usyscall $(uname -m) 1` devuelve `write`.

___

## Práctica guiada


**- ¿Para qué sirven los macros SYS_CALL_DEFINE?**

La macro permite definir el cuerpo de las system calls, creando una función con la correspondiente firma. Simplifica el proceso de agregar nuevas system calls, proveyendo una forma estandarizada de declararlas. Esta macro simplifica el proceso de crear system calls, manejando varios requerimientos de código boilerplate, como el pasaje de parámetros. Utilizar esta macro permite que los metadatos sobre la nueva system call estén disponibles para otras herramientas.
Se utiliza de la forma `SYSCALL_DEFINEn()`, donde n es el número de argumentos que recibirá la system call. La macro recibe como argumento el nombre de la system call seguido por los parámetros de la misma como argumentos (indicando tipo y nombre).

**- ¿Para que se utilizan la macros for_each_process y for_each_thread?**

La macro `for_each_process` permite iterar sobre la lista completa de tasks de la task struct (implementada como circular doubly-linked list).
La macro `for_each_thread` itera sobre todos los threads de una task dada.

## Módulos y Drivers


### 2. ¿Qué es un driver? ¿Para qué se utiliza?

Un driver es componente de software que le permite al kernel comunicarse con dispositivos, permitiendo operar con ellos sin necesidad de conocer los detalles específicos del hardware de cada uno.
El driver es la capa intermedia entre el hardware del dispositivo y el kernel. Actúan una API para comunicar este último con el dispositivo, implementando el código específico para operar con este.

### 3. ¿Por qué es necesario escribir drivers?

Es necesario escribir drivers para abstraer al kernel de los detalles del hardware de cada dispositivo, incrementando la interoperabilidad del sistema. Si se quiere comunicar con un dispositivo específico, basta con descargar el driver que permite dicha comunicación. El kernel invocará funciones expuestas por el driver y este ejecutará el código específico para el dispositivo.

### 4. ¿Cuál es la relación entre módulo y driver en GNU/Linux?

Un driver permite la comunicación con dispositivos de hardware mientras que un módulo es una porción de código que puede ser cargada y descargada dinámicamente. Los drivers **pueden** ser implementados como módulos, permitiendo su carga dinámica. Esto es especialmente útil con dispositivos que podrían ser añadidos o removidos del sistema. Además, los módulos del kernel permiten agregar nuevas características y solucionar errores en drivers existentes sin que esto impacte en la totalidad del kernel.

Sin embargo, hay que tener en cuenta que no todos los módulos del kernel son drivers. También existen módulos que podrían proveer servicios o extender la funcionalidad del kernel, más allá de comunicarse con un dispositivo externo.
Además, los drivers no necesariamente se implementan como módulos. Aunque esto los dota de mayor flexibilidad, también podrían ser parte del código principal del kernel.

### 5. ¿Qué implicancias puede tener un bug en un driver o módulo?

Dado que los drivers y módulos se ejecutan en modo kernel, tienen permisos ilimitados y acceso irrestricto al hardware. Por lo tanto, un bug en ellos puede producir errores graves que conduzcan a un kernel panic y afecten a todo el sistema.

### 6. ¿Qué tipos de drivers existen en GNU/Linux?

Se pueden clasificar en tres tipos:

- Character Device Drivers: drivers utilizados para dispositivos que transfieren información de forma secuencial, byte a byte. Por ejemplo: teclado, puerto serial.

- Block Device Drivers: utilizados para dispositivos que transfieren datos en bloques de tamaño fijo, tal como HDDs o SSDs.

- Network Device Drivers: estos drivers son responsables de gestionar interfaces de red, tal como adaptadores Wi-Fi o Ethernet.

### 7. ¿Qué hay en el directorio /dev? ¿Qué tipos de archivo encontramos en esa ubicación?

En el directorio `/dev` se encuentras los device nodes, que son archivos especiales que representan dispositivos de hardware. Cuando un driver es cargado, generalmente se crea su correspondiente device node. Los procesos pueden interactuar con los dispositivos leyendo o escribiendo estos archivos.

### 8. ¿Para qué sirven el archivo /lib/modules/<version>/modules.dep utilizado por el comando modprobe?

`modules.dep` es un archivo de texto generado por el comando `depmod` que lista las dependencias de cada módulo. Es un texto informativo legible para humanos. Su contraparte funcional es el binario `modules.dep.bin`. Es utilizado por herramientas como `modprobe`, que permite cargar y descargar módulos.

Las dependencias de un módulo le permiten funcionar y deben ser respetadas al momento de cargarlo o descargarlo. Por ello, este archivo resulta de gran utilidad.

### 9. ¿En qué momento/s se genera o actualiza un initramfs?

El `initramfs` es un sistema de archivos temporal que se monta en memoria durante el arranque del sistema. Este contiene ejecutables, drivers y módulos necesarios para lograr iniciar el sistema.

- Este es generado durante el proceso de booteo del sistema, permitiendole al kernel acceder a los archivos y drivers necesarios.
- Cuando un nuevo kernel es instalado, se ejecuta el comando `update-initramfs`, lo que genera o actualiza el mismo. Este comando garantiza que el `initramfs` contiene los drivers y archivos necesarios para que el nuevo kernel bootee correctamente.
- En caso de realizar cambios en lo drivers, como agregar nuevos, el archivo `initramfs` también debe ser actualizado.
- Si se realizan cambios en el file system, tal como activar RAID o encriptación, podría ser necesario actualizarlo.

### 10. ¿Qué módulos y drivers deberá tener un initramfs mínimamente para cumplir su objetivo?

El `initramfs` tiene como objetivo permitir que se monte el sistema de archivos raíz real durante el arranque. Por ello, requiere módulos y drivers indispensables para acceder a este.

Debe tener:

- Driver para dispostivo de almacenamiento: el kernel necesitará acceder al dispositivo de almacenamiento donde está `/`.
- Driver del file system: el kernel debe poder interpretar el formato del root filesystem. Ej.: `ext4`, `btrfs`.

Luego, podría ser necesario incluir soporte para particiones, LVM o dispositivos de red, según sea el caso.
El archivo `initramfs` debería incluir todos los drivers específico para cualquier dispositivo de hardware del sistema, ya que el kernel necesitará cargar los módulos necesarios.


#### Desarrollando un módulo simple para Linux

**b. ¿Para qué sirve la macro MODULE_LICENSE? ¿Es obligatoria?**

La macro informa al kernel bajo que licencia se distribuye el modelo. Esto es importante pues el kernel de Linux es GPL, y ciertas funciones internas solo están disponibles para módulos con licencias compatibles. No es obligatoria en el sentido de que el módulo puede compilarse sin ella, pero se emitirá un mensaje indicando que "kernel tainted", lo que implica que el muchas funciones del kernel podrían no estar disponibles para el módulo.

# Sistemas Operativos Práctica 3

## Threading (ULT y KLT)

### 6. Qué retornan las siguientes funciones:

*a. getpid()*

Retorna el PID del proceso actual.

*b. getppid()*

Retorna el PID del proceso padre del actual.

*c. gettid()*

Retorna el ID del thread actual. Si es un ambiente single-threaded, coincide con el PID del proceso.

*d. pthread_self()*

Retorna el ID del thread actual a nivel Pthreads (el asignado cuando se llamó a pthread_create). Pthreads es una librería que permite crear KLTs.

*e. pth_self()*

Retorna el ID del thread actual a nivel GNU Pth (librería para crear ULTs).

### 7. ¿Qué mecanismos de sincronización se pueden usar? ¿Es necesario usar mecanismos de sincronización si se usan ULT?

Para memoria compartida se pueden utilizar mecanismos para la gestión de la concurrencia como semáforos, barreras, mutex, condicionales.
A nivel archivos, pueden utilizarse E/S sincrónicas con bloqueos de lectura/escritura.
El objetivo es garantizar la sincronización por exclusión mutua y condición.

Tanto los ULTs como KLTs de un mismo proceso comparten espacio de direcciones, por lo que para mantener la consistencia de los datos, es necesario acceder a los recursos compartidos de forma sincronizada. Para ambos es necesario utilizar mecanismos de sincronización.

### 8. Procesos

*a. ¿Qué utilidad tiene ejecutar fork() sin ejecutar exec()?*

La syscall `fork()` crea un nuevo proceso que comparte las páginas del padre, retornando 0 en el hijo y el PID del hijo en el padre. Si se escribe alguna de las páginas en el hijo, por la política Copy On Write, se duplica esa página, con los nuevos contenidos para el hijo.

Es útil si se quiere que dos procesos ejecuten el mismo código, sin interferir con el otro (debido al COW).

*b. ¿Qué utilidad tiene ejecutar fork() + exec()?*

`exec()` es una system call que permite ejecutar un nuevo programa en el proceso actual, reemplazando las páginas en el proceso.

La combinación `fork()` + `exec()` permite crear un nuevo proceso idéntico al padre y reemplazar el programa en ejecución por otro. En síntesis, permite ejecutar un nuevo programa.

*c. ¿Cuál de las 2 asigna un nuevo PID fork() o exec()?*

La syscall `fork()` asigna un nuevo PID al proceso hijo.


*e. ¿Qué consecuencias tiene no hacer wait() sobre un proceso hijo?*

Permanecen en la tabla para que el padre pueda obtener información sobre ellos y, en particular, leer su exit status.

En definitiva, un proceso zombie se produce cuando un proceso hijo termina pero el proceso padre no ha llamado a `wait()` o `waitpid()` (para un hijo específico) para leer su status.
Si un padre termina, los procesos zombies existentes son adoptados por el proceso `init` por defecto.

*f. ¿Quién tendrá la responsabilidad de hacer el wait() si el proceso padre termina sin hacer wait()?*

Los procesos zombie cuyos padres terminaron sin hacer `wait()` son adoptados por `init()`. Este ejecuta `wait()` periodicamente, permitiendo que el proceso zombie sea desalocado de la tabla.


### 10. User Level Threads

*a. ¿Por qué los ULTs no se pueden ejecutar en paralelo sobre múltiples núcleos?*

Esto se debe a que estos son creados y administrados por la aplicación de usuario, pero el Kernel no los conoce. Este ve y planifica únicamente procesos. Por ello, aunque son ligeros en cuestión de costo de creación y permiten flexibilidad en la planificación, al no ser estructuras conocidas por el Kernel, no pueden utilizarse como unidades de ejecución para explotar el paralelismo.

*b. ¿Qué ventajas tiene el uso de ULTs respecto de los KLTs?*

Las ventajas ya fueron mencionadas anteriormente. Principalmente son:

- No intervención del Kernel en el intercambio de hilos en ejecución (todos comparten el mismo espacio).
- Flexibilidad para que el usuario desarrollador defina como planificar los hilos.
- Pueden ejecutarse en cualquier SO, ya que no dependen de él.

*c. ¿Qué relaciones hay entre getpid(), gettid() y pth_self() (en GNU Pth)?*

`getpid()` retorna el PID del proceso actual. `gettid()` retorna el TID del hilo actual a nivel Kernel (en single-threaded coincide con el PID). `pth_self()` retorna el ID del thread actual a nivel de la librería GNU Pth.

*d. ¿Qué pasaría si un ULT realiza una syscall bloqueante como read()?*

Si no se tienen KLTs, todos los ULTs son agrupados y planificados bajo el proceso que lo creó. El bloque de un hilo produce el bloqueo de todo el proceso, es decir, de todos los demás hilos.

*e. ¿Qué tipos de scheduling pueden tener los ULTs? ¿Cuál es el más común?*

El scheduling de los ULTs puede ser definido por el programador, aun si la planificación difiere de la que usa el Kernel para los procesos. Podría usarse Round Robin, prioridades, etc.

Suele utilizarse el `SCHED_OTHER`, que es la política de planificación por defecto de los procesos de usuario en Linux. Su principal objetivo es asegurar el fairness en la ejecución, asignando porciones de tiempo variables a cada proceso para mantener el tiempo de cada uno de forma lo más equitativa posible.

### 11. Global Interpreter Lock

*a. ¿Qué es el GIL (Global Interpreter Lock)? ¿Qué impacto tiene sobre programas multi-thread en Python y Ruby?*

El GIL es un mecanismo de sincronización por exclusión mutua que utilizan algunos lenguajes interpretados para asegurar que solo un hilo en ejecución pueda ejecutar código del lenguaje a la vez por cada proceso.
Esta idea rompe el paralelismo, ya que deben ejecutarse de a uno, turnándose.

El GIL se creó debido a la gestión de memoria basada en el conteo de referencias que hacían lenguajes como Ruby o Python. Permitir el paralelismo de hilos haría que potencialmente varios hilos puedan modificar este conteo (causando corrupción de datos o fugas de memoria). Para evitarlo, se diseñó un mecanismo para bloquear todo el intérprete.

El impacto en programas multi-threaded depende de si las tareas son CPU-bound o I/O-bound. Para las primeras la performance se degrada inmensamente, ya que prácticamente se ejecuta de forma secuencial. Para aquellas orientas a I/O el impacto es casi nulo, porque cuando un hilo se bloquea por una operación E/S, libera el GIL y otro puede continuar con la ejecución.

En la actualidad, tanto en Python (con CPython como implementación de referencia para el lenguaje) como Ruby (CRuby), poseen distintos mecanismos para intentar solventar estas problemáticas

# Sistemas Operativos Práctica 4A

## cgroups & namespaces

### 2. Crear un subdirectorio llamado sobash dentro del directorio root. Intente ejecutar el comando chroot /root/sobash. ¿Cuál es el resultado? ¿Por qué se obtiene ese resultado?

Se obtiene `/usr/sbin/chroot: failed to run command ‘/bin/bash’: No such file or directory`.

Esto se debe a que cuando se ejecuta `chroot` sin especificar un programa adicional, el comando intenta lanzar el intérprete de comandos predeterminado dentro del nuevo entorno raíz. Como dicho directorio está completamente vacío, no existe el archivo `/bin/bash` dentro de él. El error indica que no se encuentra el ejecutable que debe iniciar la sesión interactiva dentro del entorno aislado. 

### 3. Cree la siguiente jerarquía de directorios dentro de sobash:

```
sobash/
├── bin
├── lib
│ └── x86_64-linux-gnu
└── lib64
```

Se construye una estructura mínima que simule un sistema Linux esencial.

### 4. Verifique qué bibliotecas compartidas utiliza el binario `/bin/bash` usando el comando `ldd /bin/bash`. ¿En qué directorio se encuentra `linux-vdso.so.1`? ¿Por qué?

El comando `ldd` muestra las dependencias de de un programa o biblioteca compartida. Utiliza las siguientes bibliotecas compartidas:

```
linux-vdso.so.1 (0x00007ffec7b51000)
libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f9a79663000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9a79482000)
/lib64/ld-linux-x86-64.so.2 (0x00007f9a797de000)
```

`linux-vdso.so.1` no se encuentra en ningún directorio del disco (en la salida no se indica ningún path). Esto se debe a que `linux-vdso.so.1` (Virtual Dynamic Shared Object) es una biblioteca virtual compartida que el kernel inyecta automáticamente en el espacio de memoria de cada proceso. Su finalidad es optimizar el rendimiento, permitiendo que los binarios ejecuten ciertas system calls muy comunes directamente en el espacio de usuario, evitando un camboio de contexto hacia el espacio del kernel. Al ser proveída por el kernel desde RAM, no es necesario ni posible coíarla a la chroot jail.

### 5. Copie en /root/sobash el programa `/bin/bash` y todas las librerías utilizadas por el programa bash en los directorios correspondientes. Ejecute nuevamente el comando `chroot` ¿Qué sucede ahora?

El comando se ejecuta exitosamente e ingresa con una terminal interactiva al entorno aislado. Se ejecuta un proceso bash que cree que `/root/sobash` es la raíz `/`.

### 6. ¿Puede ejecutar los comandos cd "directorio" o echo? ¿Y el comando ls? ¿A qué se debe esto?

Los comandos `cd` y `echo` pueden ejecutarse correctamente, ya que ambos son comandos internos de la shell. Su código fuente está compilado e integrado directamente en el binario `/bin/bash`. Sin embargo, el comando `ls` no puede ejecutarse (`bash: ls: command not found`) ya que `ls` es un programa binario independiente que reside en `/usr/bin/ls`. Como la terminal se encuentra en la chroot jail y no se copió el binario de `ls`, no es posible encontrar el binario.

**e. Ejecute el comando “ps aux” ¿Qué procesos ve? ¿Por qué (pista: ver el contenido de /proc)?**

No se ve ningún proceso y el directorio `/proc` está vacío. El comando `ps` lee los archivos que se encuentra en `/proc` (un sistema de archivos virtual provisto por el kernel para acceder a información del sistema). Como la jaula está recién creada y su `/proc` interno está vacío, `ps` no tiene nada que listar.

**f. Monte /proc con “mount -t proc proc /proc” y vuelva a ejecutar “ps aux” ¿Qué procesos ve? ¿Por qué?**

Ahora se ven todos los procesos que se ejecutan en la máquina física real (host), incluyendo todos los procesos de usuario y del kernel. El sistema de archivos `/proc` está vinculado directamente al kernel de la máquina. Como el entorno chroot creado comparte el mismo kernel, al montar el directorio se puede acceder al estado global del sistema. `chroot` aisla los archivos pero no los procesos.

**g. Acceda a /proc/1/root/home/so ¿Qué sucede?**

Pueden verse todos los archivos del home del usuario `so`, que deberían estar bloqueados e inaccesibles. Esto se debe a que el proceso con PID 1 es el proceso de inicio del SO, fuera de la jaula. El kernel expone en la ruta `/proc/1/root` un enlace simbólico que apunta a la raíz real del sistema físico. Al acceder a esta ruta del kernel, el proceso enjaulado "escapa del chroot", pudiendo acceder a todo el sistema de archivos.

**h. ¿Qué conclusiones puede sacar sobre el nivel de aislamiento provisto por chroot?**

- Aislamiento parcial y superficial: `chroot` proporciona aislamiento a nivel sistema de archivos, modifica la percepción en la forma de acceder a los archivos para un proceso, pero no altera su relación con el SO global.

- Inexistencia de aislamiento de procesos: un proceso dentro de chroot comparte el mismo espacio de nombres de procesos que el host. Puede ver e interactuar con procesos externos.

- Falsa seguridad: el aislamiento es fácilmente vulnerable si el proceso dentro de la jaula se ejecuta con privilegios. Existen múltiples técnicas de escape de la jaula, por lo que no es ideal para la seguridad.

- Para lograr aislamiento real y seguro, como requieren los contenedores, se necesitan otras herramientas más sofisticadas.


## Control Groups

### 1. ¿Dónde se encuentran montados los cgroups? ¿Qué versiones están disponibles?

Los cgroups se encuentran montados en el directorio virtual `/sys/fs/cgroup/`. Al ser un sistema de archivos virtual (`cgroupfs`), no ocupa espacio en el disco duro, sino que se monta en memoria principal.

Existen diferencias entre ambas versiones de cgroups:

- cgroups v1 (Modelo multi-jerarquía): cada recurso o subsistema (CPU, memoria, I/O) se gestiona de forma independiente en su propio árbol de directorios (ej.: `/sys/fs/cgroup/cpu`). Un proceso puede pertenecer a varios grupos en distintos subsistemas, pero no a distintos grupos de un mismo subsistema.

- cgroups v2 (Modelo de jerarquía unificada): reduce la complejidad de la v1. Implementa un único árbol jerárquico unificado donde todos los controladores de recursos se aplican de forma conjunta a los mismos grupos de procesos.

### 2. ¿Existe algún controlador disponible en cgroups v2? ¿Cómo puede determinarlo?

Cada cgroup de la jerarquía v2 contiene archivos `cgroup.controllers` (indica controladores disponibles en un cgroup) y `cgroup.subtree_control` (controladores que se habilitarán en los cgroups hijos).

### 3. Analice qué sucede si se remueve un controlador de cgroups v1 (por ej. `Umount /sys/fs/cgroup/rdma`).

Al ejecutar el comando `umount`, se desmonta ese subsistema específico del sistema de archivos de cgroups. El directorio correspondiente al subsistema quedará vacío y ya no se podrá modificar las reglas de recursos de este.

A nivel kernel, el subsistema continúa activo y las estructuras que controlan los procesos no se destruyen automáticamente. Si se vuelve a montar, todavía estarían ahí.


## Namespaces

### 1. Explique el concepto de namespaces.

Un Namespace es una característica del kernel de Linux que abstrae los recursos globales del sistema, de forma que un proceso tenga la ilusión de poseer su propia instancia aislada de dicho recurso.

Limitan lo que el proceso puede ver y, en consecuencia, lo que puede usar. Las modificaciones del recurso quedan contenidas en el namespace.

### 2. ¿Cuáles son los posibles namespaces disponibles?

- PID (Process ID): Aísla la numeración y el árbol de procesos. Permite que un proceso hijo sea el PID 1 dentro de su entorno sin interferir con el PID 1 real del host.

- NET (Network): Aísla los dispositivos de red, las direcciones IP, las tablas de enrutamiento, las reglas de firewall (iptables) y los puertos de escucha (sockets).

- MNT (Mount): Aísla los puntos de montaje del sistema de archivos. Permite que un proceso tenga una vista de discos y carpetas montadas totalmente diferente a la del host.

- UTS (UNIX Timesharing System): Aísla el nombre del host (hostname) y el nombre de dominio NIS.

- IPC (Inter-Process Communication): Aísla los recursos de comunicación entre procesos, como las colas de mensajes POSIX y los segmentos de memoria compartida System V.

- USER: Aísla las identidades de usuarios y grupos (UID y GID). Permite que un usuario común y corriente en el host sea visto como el superusuario root (UID 0) dentro de su namespace.

- CGROUP: Oculta y aísla la vista de la jerarquía de los grupos de control.

- TIME: Aísla los relojes del sistema (permite cambiar la fecha/hora de un entorno sin alterar el reloj del servidor físico).

### 5. Usando el comando unshare crear un nuevo namespace de tipo UTS.

El comando unshare permite ejecutar un programa desvinculando ciertos namespaces heredados de su padre.

**a. unshare --uts sh (son dos (- -) guiones juntos antes de uts)**

Crea una nueva shell con un namespace UTS propio y aislado.

**b. ¿Cuál es el nombre del host en el nuevo namespace? (comando hostname)**

El mismo que tenía fuera de la nueva shell, ya que al crear el espacio, el Kernel clona inicialmente los valores de texto del entorno del padre.

**c. Ejecutar el comando lsns. ¿Qué puede ver con respecto a los namespace?.**

```bash
        NS TYPE   NPROCS   PID USER             COMMAND
4026531834 time      115     1 root             /sbin/init
4026531835 cgroup    115     1 root             /sbin/init
4026531836 pid       115     1 root             /sbin/init
4026531837 user      115     1 root             /sbin/init
4026531838 uts       110     1 root             /sbin/init
4026531839 ipc       115     1 root             /sbin/init
4026531840 net       115     1 root             /sbin/init
4026531841 mnt       111     1 root             /sbin/init
4026532448 mnt         1   275 root             ├─/lib/systemd/systemd-udevd
4026532449 uts         1   275 root             ├─/lib/systemd/systemd-udevd
4026532513 mnt         1   325 systemd-timesync ├─/lib/systemd/systemd-timesyncd
4026532557 uts         1   325 systemd-timesync ├─/lib/systemd/systemd-timesyncd
4026532613 uts         1   530 root             ├─/lib/systemd/systemd-logind
4026532614 mnt         1   530 root             └─/lib/systemd/systemd-logind
4026531862 mnt         1    38 root             kdevtmpfs
4026532558 uts         2  1050 root             sh  # Nueva línea
```

La última línea es nueva y corresponde al nuevo namespace UTS creado. Su identificador es distinto al del namespace UTS global y su comando líder es `sh`. 

**d. Modificar el nombre del host en el nuevo hostname.**

Se hace con `hostname <name>`. Si chequeamos el hostname, vemos que en esta terminal cambió.

**e. Abrir otra sesión, ¿cuál es el nombre del host anfitrión?**

Se ve el nombre original de la máquina virtual intacto. Las modificaciones hechas dentro de la shell con `unshare` no tiene impacto en el SO pruncipal, ya que el namespace UTS que se creó está aislado.

**f. Salir del namespace (exit). ¿Qué sucedió con el nombre del host anfitrión?**

El nombre permanece sin cambios (conserva el nombre original).

# Sistemas Operativos Práctica 4B

## Docker y Docker Compose

## Docker

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

**c. Salga del contenedor y genere una imagen Docker a partir de éste. ¿Con qué nombre se genera si no se especifica uno?**

Primero obtenemos su id con `docker ps -a`. Despues usamos `docker commit <container-id>`. Consultamos con `docker images -a` y vemos que se genera con el nombre `<untagged>`. Repository se refiere al nombre de la imagen y tag a la versión. En mi caso aparece todo en la columna IMAGE.

**d. Cambie el nombre de la imagen creada de manera que en la columna Repository aparezca nginx-so y en la columna Tag aparezca v1.**

Se obtiene el id del contenedor y se cambia con `docker tag <container-id> <nombre:tag>`.

___

**g. Modifique el archivo index.html agregándole un párrafo con su nombre y número de alumno. ¿Es necesario reiniciar el contenedor para ver los cambios?**

No es necesario ya que se usó un Bind Mount (`-v`). Este es un método que permite linkear un directorio específico de la máquina host directo a un directorio en el contenedor, permitiendo sincronización en tiempo real entre ambos.

**h. Analice: ¿por qué es necesario que el proceso nginx se ejecute en primer plano? ¿Qué ocurre si lo ejecuta sin -g 'daemon off;'?**

Como se indicó antes, un contenedor vive mientras el proceso principal que lo inició esté activo. Por defecto, Nginx nace, genera procesos hijos para atender conexiones y se mueve a segundo plano, terminando el proceso de arranque inicial. Si se ejecuta sin `-g 'daemon off;'`, el proceso inicial finaliza de inmediato, lo que causa que Docker asuma que el contenedor terminó sus tareas y proceda a apagarlo automáticamente.


**d. Modifique el archivo index.html del host agregando un párrafo con la fecha actual y recargue la página en su navegador web. ¿Se ven reflejados los cambios que hizo en el archivo? ¿Por qué?**

No, el cambio no se vio reflejado. Esto se debe a que no se usó el Bind Mount (`-v`), sino que se copió el archivo al directorio del contenedor. Por lo tanto, el archivo editado en el host anfitrion y el del contenedor son distintos. El archivo en el contenedor forma parte de las capas inmutables de la imagen de nginx-so:v2.

**e. Termine el contenedor iniciado antes y cree uno nuevo utilizando el mismo comando. Recargue la página en su navegador web. ¿Se ven ahora reflejados los cambios realizados en el archivo HTML? ¿Por qué?**

Tampoco se ven reflejados los cambios, porque el nuevo contenedor nace a partir de la imagen estática nginx-so:v2, la cual sigue conteniendo la versión vieja del archivo `index.html` grabada en su estructura de capas.

**f. Vuelva a construir una imagen Docker a partir del Dockerfile creado anteriormente, pero esta vez dándole el nombre nginx-so:v3. Cree un contenedor a partir de ésta y acceda a la página en su navegador web. ¿Se ven reflejados los cambios realizados en el archivo HTML? ¿Por qué?**

Esta vez si se ve reflejado el cambio, ya que la nueva imagen fue creada copiando el archivo ya modificado, generando una nueva capa de imagen actualizada para la versión v3.

___


## Docker Compose

### 1. Utilizando sus palabras describa, ¿qué es docker compose?

Dcoker Compose es una herramienta de orquestación local que permite definir y gestionar aplicaciones de contenedores múltiples. En lugar de ejecutar manualmente una larga secuencia de comandos `docker run` individuales con configuraciones complejas de redes, volúmenes y variables de entorno para cada componente de la aplicación (base de datos, backend, frontend), Docker Compose permite unificar y automatizar todo el ciclo de vida de la infraestructura de la aplicación en un único entorno coordinado, encendiendo o apagando toda la arquitectura con un solo comando.

Básicamente facilita el despliegue de aplicaciones compuestas por múltiples contenedores.

### 2. ¿Qué es el archivo compose y cual es su función? ¿Cuál es el “lenguaje” del archivo?

El archivo Compose (nombrado `docker-compose.yml` o  `compose.yaml`) es un archivo de configuración declarativo. Su función es actuar como la receta donde se describe detalladamente el detalle de toda la infraestructura de la aplicación: que contenedores se necesitan, cómo se comunican, que discos usan y que límites de hardware poseen.

Está escrito en YAML, con una estructura indentada por espacios de clave-valor y listas.

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

# Sistemas Operativos Práctica 5

## Seguridad

### 5. ¿Qué es ASLR (Address Space Layout Randomization)? ¿Linux provee ASLR para los procesos de usuario? ¿Y para el kernel?

ASLR es una técnica de seguridad defensiva a nivel de sistema operativo diseñada para prevenir ataques de explotación de memoria, como stack buffer overflow.

Sin ASLR, cada vez que un programa se ejecuta, el sistema operativo mapea sus componentes en las mismas direcciones de memoria RAM exactas, por lo que un atacante podría averiguar dichas direcciones.

ASLR rompe esta predictibilidad. Cada vez que el programa se inicia, el gestor de memoria virtual del sistema operativo reorganiza de forma aleatoria las direcciones base donde se ubican la pila, el heap y las librerías.

Linux implementa ASLR de forma nativa para el espacio de usuario. Para que funcione plenamente, los programas deben compilarse como ejecutables independientes de la posición (código PIE - Position Independent Executable), algo que hacen por defecto todos los compiladores modernos (como gcc).

El kernel de Linux incluye su propia implementación de ASLR llamada KASLR. Introducida para proteger al propio núcleo del sistema de exploits que buscan escalar privilegios, KASLR aleatoriza la dirección de memoria física y virtual donde se descomprime y aloja el código del kernel de Linux durante el proceso de arranque (boot). Una vez que la máquina arranca, la posición del kernel se queda fija hasta el próximo reinicio.

### 6. ¿Cómo se activa/desactiva ASRL para todos los procesos de usuario en Linux?

En Linux, la configuración de ASLR se gestiona a través del sistema de archivos virtual `/proc`, mediante el parámetro del kernel `kernel.randomize_va_space`. Este parámetro acepta tres valores:

- 0: desactivado. El diseño del espacio de direcciones es completamente estático.
- 1: activado parcial. Se aleatorizan las posiciones de la pila, memoria compartida y librerías, pero el heap permanece fijo.
- 2: activado completo. Se aleatorizan todos los segmentos, incluyendo el heap.

## B - Ejercicio introductorio: Buffer Overflow simple

La ejecución produce la salida `access pointer: 0x7ffc6d19415f, password pointer: 0x7ffc6d194140, distance: 31`.


### 4. Volver a ejecutar pero ingresar una password lo suficientemente larga para sobreescribir access. Usar distance como referencia para establecer la longitud de la password.

Si se pone una contraseña lo suficientemente larga (32 caracteres o más), se sobreescribe el valor "0" de access, lo que produce que este habilite a realizar el login.

El valor access comienza en 0 y solo cambia a 1 si se ingresa la contraseña correcta. Desbordando el buffer es posible sobreescribir su valor a uno distinto de 0, lo que habilita a que la función de login permita el acceso.

### 5. Después de realizar la explotación, reflexiona sobre las siguientes preguntas:

**a. ¿Por qué el uso de gets() es peligroso?**

La función `gets()` en C es utilizada para leer un string desde la entrada estándar hasta que se encuentra un caracter de salto de línea. Sin embargo, se desaconseja su uso ya que esta función no realiza chequeos de límites de caracteres, produciendo que sea vulnerable a ataques de buffer overflow.

**b. ¿Cómo se puede prevenir este tipo de vulnerabilidad?**

Se deben usar funciones seguras que exigen especificar el tamaño máximo del buffer destino (`fgets()`, `strncpy()`), evitando sus alternativas inseguras que no controlan los límites (`gets()`, `strcpy()`).

Se deben realizar chequeos de los tamaños de inputs antes de copiarlo a una estructura de datos interna.

Existen lenguajes como Rust que eliminan estas vulnerabilidades, ya que su compilador manejan la memoria de forma automática y prohíben los accesos fuera de límites.

**c. ¿Qué medidas de seguridad ofrecen los compiladores modernos para evitar estas vulnerabilidades?**

Los compiladores modernos implementan protecciones automatizadas como:

- Stack Canaries: el compilador inyecta un valor numérico aleatorio secreto en la pila antes de la dirección de retorno. Si esta se pisa y el programa detecta que el "canario" cambió, aborta la ejecución.

- NX / DEP (No-Execute / Data Execution Prevention): las secciones no ejecutables están marcadas con el bit NX, por lo que cualquuier script malicioso inyectado dentro de la pila no será ejecutado.

- PIE (Position Independent Executable): Compila el binario de forma que sus instrucciones de código ejecutable no dependan de posiciones fijas.

## C - Ejercicio: Buffer Overflow reemplazando dirección de retorno

**a. ¿Qué efecto tiene setear el bit setuid en un programa si el propietario del archivo es root? ¿Qué efecto tiene si el usuario es por ejemplo nobody?**

Si el propietario es root, el programa se ejecuta con el dominio de protección y los privilegios totales del root, sin importar que un usuario no privilegiado lo haya invocado.

Si el propietario es nobody, el programa se ejecutará con los privilegios del usaurio de mínimos recursos del sistema. Cualquier usuario que ejecuta el archivo verá su dominio reducido dinámicamente al entorno restringido de nobody, limitando su acceso a archivos o recursos del sistema.

**b. ¿Cómo ASLR ayuda a evitar este tipo de ataques en un escenario real donde el programa no imprime en pantalla el puntero de la función objetivo?**

En un escenario real, el atacante no conoce las posiciones de memoria porque no se imprimen en pantalla. Como ASLR cambia aleatoriamente la dirección de la función privileged_fn() en cada reinicio, el atacante tendría que "adivinar" un número hexadecimal de 64 bits. Al fallar en el intento, el payload apuntará a una dirección de memoria inválida o vacía, causando un Error de Segmentación (Segmentation Fault) que colapsará el programa de inmediato.

**c. ¿Cómo podría evitar este tipo de ataques en un módulo del kernel de Linux? ¿Qué mecanismo debería estar habilitado?**

Para evitar desbordamientos dentro del espacio de memoria del kernel:

- Mecanismo de Compilación: El kernel debe compilarse habilitando el uso de Stack Canaries dentro de las funciones del propio kernel.

- Mecanismo de Sistema Operativo: Debe estar habilitado KASLR (Kernel Address Space Layout Randomization). Esto garantiza que la estructura interna del mapa de memoria del kernel sea aleatoria en cada arranque del sistema, previniendo que un bug en un driver o módulo se convierta en una fuente para inyección de exploits.

## C - Ejercicio SystemD


### 1. Investigue los comandos:

**a. systemctl enable**

Configura un servicio para que arranque de forma automática durante el booteo del SO.

**b. systemctl disable**

Cancela el arranque automático de un servicio al iniciar el sistema.

**c. systemctl daemon-reload**

Fuerza a SystemD a reexplorar el disco y recargar en memoria principal todos los archivos de configuración de todas las unidades. Debido a que SystemD tiene un mapa estático en memoria de los servicios, el comando se usa cuando se edita manualmente el archivo `.service` en `/etc/systemd/system/`.

**d. systemctl start**

Inicia una unidad de servicio en la sesión actual.

**e. systemctl stop**

Detiene un servicio que se encuentra en ejecución

**f. systemctl status**

Muestra un informe en tiempo real sobre el estado de un servicio (indica si el servicio está activo, muestra el PID, ruta del archivo de configuración, cantidad de memoria y CPU consumida, últimos logs).

**g. systemd-cgls**

Lista de forma recursiva todo el árbol de cgroups activos en el sistema operativo, mostrando que procesos pertenecen a qué servicio.

En cuestiones de seguridad, se puede utilizar para verificar que SystemD ha aislado un servicio dentro de su propia porción de la jerarquía de cgroups.

**h. journalctl -u [unit]**

Filtra y muestra de forma exclusiva los logs generados por una unidad especifica (ej.: `journalctl -u nginx.service`).

### 2. Investigue las siguientes opciones que se pueden configurar en una unit service de systemd:

**a. User y Group**

Define bajo que dominio de protección de UNIX (UID/GID) se ejecutará el binario del servicio. Si no se especifican, los servicios se ejecutan como root. Setear User=nobody degrada inmediatamente los privilegios del proceso al nacer, impidiendo que pueda escribir en archivos del sistema o realizar acciones administrativas si el servicio llega a ser comprometido por un atacante.

**b. ProtectHome**

Controla el acceso del servicio a los directorios de los usuarios. Puede setearse como true o read-only (acceso de solo lectura). Se implementa mediante un mount namespace.

**c. PrivateTmp**

Cuando se setea en true, SystemD intercepta los accesos del servicio a las carpetas temporales globales `/tmp` y `/var/tmp`. No puede acceder a esos directorios globales, sino a una "réplica" vacía.

**d. ProtectProc**

Restringe la visibilidad que tiene el servicio sobre la información de otros procesos del sistema operativo (restringe acceso a `/proc`). Para ello debe setearse `ProtectProc=invisible`.

**e. MemoryAccounting, MemoryHigh y MemoryMax**

Activan y configuran los límites de hardware del cgroup de memoria para el servicio:

- MemoryAccounting=true: Habilita a SystemD a medir y llevar la contabilidad exacta de cuántos bytes de RAM consume el servicio.

- MemoryHigh: Establece un límite "blando". Si el servicio pasa este valor, el kernel ralentiza los procesos del servicio y aplica una recolección de memoria agresiva.

- MemoryMax: Establece el límite "duro". Si el servicio intenta superarlo, el kernel activa el OOM Killer (Out Of Memory Killer) y destruye el proceso inmediatamente enviando un SIGKILL.

Un proceso es un programa en ejecución, que ha sido cargado en memoria y al cual el procesador le asigna ciclos de cómputo. El proceso posee su identificador (PID), descriptores de archivos, espacio de memoria aislado e hilo de ejecución.

Un servicio (daemon) es un concepto lógico. Es un programa diseñado para ejecutarse en segundo plano de forma continua, persistente y sin interacción directa con un usuario, con el objetivo de proveer una función constante al sistema o la red. Un serivicio puede generar varios procesos para atender una demanda.

___


## D - AppArmor

AppArmor es un móudlo de seguridad del kernel de Linux basado en el Mandatory Access Control (MAC). A diferencia del control de acceso clásico basado en dominios y permisos, donde el dueño de un archivo puede decidir quien lo lee, AppArmor impone restricciones globales definidas por el administrador del sistema.

AppArmor funciona asociando a los programas ejecutables un perfil de seguridad. Este perfil define de forma estricta y detallada qué recursos puede tocar el proceso, tales como:

- Qué archivos puede leer, escribir o ejecutar (usando rutas absolutas).

- Qué capacidades del kernel (capabilities) puede invocar (ej. abrir un socket de red, montar discos).

- Qué operaciones de red puede realizar.

___

## E - CopyFail



**a. ¿Es una vulnerabilidad en espacio de usuario o del kernel?**

Es una vulnerabilidad del kernel de Linux. Especificamente, el fallo se encuentra en el subsistema de gestión de memoria virtual del kernel.

**b. ¿Por qué el exploit abre un archivo con suid?**

El exploit busca archivos con el bit `setuid` activo (como el comando `su`), porque estos archivos pertenecen al usuario root y el SO le otorga permisos de lectura a los usuarios comunes para poder ejecutarlos.

Al mapear este archivo binario privilegiado de solo lectura en su memoria virtual usando mmap(), el exploit aprovecha la vulnerabilidad para engañar al kernel y escribir directamente sobre la copia en caché de memoria de ese archivo de root, logrando inyectar código malicioso dentro de un binario del sistema que el kernel ejecutará con máximos privilegios.

**c. ¿La página modificada se almacena luego en disco?**

Sí. Aunque el exploit modifica la página de memoria en la memoria RAM, el subsistema de almacenamiento de Linux considera que esa página de memoria está dirty. Eventualmente el daemon del kernel del host se encarga de vaciar la caché y escribe los cambios en el disco duro físico, corrompiendo de forma permanente el archivo ejecutable original del sistema con el código del atacante.


### 3. ¿Por qué se dice que puede permitir saltar el aislamiento de containers?

Los contenedores de Docker comparten el mismo kernel del sistema operativo host. El aislamiento de Docker se basa puramente en fronteras lógicas (Namespaces y cgroups).

Dado que CopyFail es una vulnerabilidad que afecta directamente al código del kernel común, un proceso atacante atrapado dentro de un contenedor puede ejecutar el exploit de Dirty COW. Al corromper la memoria del kernel compartido, el exploit puede modificar archivos binarios que no pertenecen al contenedor, sino que pertenecen al sistema operativo host de la máquina física. Al lograr escribir en el disco del host desde la memoria compartida, el atacante rompe por completo la barrera del contenedor, logrando un escape de contenedor.

### 4. ¿Qué condiciones deben darse para que CopyFail permita afectar el sistema host si se ataca un container?

Para que un ataque desde un contenedor logre comprometer con éxito el sistema host a través de CopyFail, deben confluir las siguientes condiciones:

- Kernel Vulnerable en el Host: El sistema operativo anfitrión debe estar corriendo una versión desactualizada del kernel de Linux que contenga el bug original en el manejo de Copy-On-Write.

- Acceso a Funciones de Memoria: El contenedor no debe tener bloqueadas las llamadas al sistema críticas necesarias para explotar la vulnerabilidad.

- Mecanismos de Mitigación Desactivados o Ausentes: El perfil de AppArmor o Seccomp por defecto de Docker (que suele bloquear el acceso directo a ciertas syscalls o la modificación de montajes sensibles del kernel en /proc) debe estar modificado, relajado o deshabilitado para permitirle al proceso del contenedor interactuar de forma agresiva con las páginas del kernel.

## F - Secrets en Docker Compose (no confundir con memfd_secret)

### 1. Investigue los secrets de docker-compose

Los secrets son un mecanismo nativo diseñado para almacenar e inyectar de forma segura información confidencial sensible (como contraseñas) dentro de los contenedores.

**a. ¿Qué ventajas tienen sobre las env vars?**

Con las variables de ambiente, cualquiera con acceso al host puede leer las contraseñas en texto plano, ejecutando comandos de docker. Con secrets las credenciales no se guardan en metadatos del contenedor.

Con secrets, la información confidencial se monta como archivos temporales en RAM. Además, solo los procesos que tengan permisos explícitos de lectura sobre la ruta del archivo secreto pueden consumirlo. Con las variables de ambiente, el valor vive de forma permanente en los entornos del contenedor. Cualquier librería que lea el entorno global de la app tiene acceso a los secretos.

**b. ¿Cómo se comparan con los secrets en docker swarm?**

Aunque la sintaxis en el archivo YAML de Compose es muy parecida, su comportamiento varía drásticamente según el modo de ejecución:

- En Docker Swarm (Modo Producción): Los secretos son centralizados y cifrados de forma nativa por Docker dentro del almacén del clúster. Cuando un contenedor se despliega, el nodo mánager viaja por la red de forma segura y le entrega el secreto únicamente en la memoria RAM del nodo donde corre el contenedor. El host nunca guarda el secreto en su disco duro.

- En Docker Compose Local (Modo Desarrollo): Al no haber un clúster ni un almacén cifrado corporativo, Docker Compose emula este comportamiento de forma local. Los secretos en Compose suelen leerse desde un archivo físico en tu directorio del host (ej. `./password.txt`) y se montan de forma transparente dentro del contenedor en la ruta `/run/secrets/secreto_nombre`.

La principal diferencia es que en Compose local, la seguridad depende de proteger correctamente los permisos del archivo de origen en el disco duro de la máquina física, mientras que en Swarm el secreto está integrado y blindado nativamente dentro de la arquitectura de la plataforma de orquestación.
