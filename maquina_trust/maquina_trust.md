## MÁQUINA INJECTION


### Introducción

En este documento, exploraremos la máquina "Trust" paso a paso, desde su despliegue inicial hasta la explotación de vulnerabilidades para obtener acceso privilegiado. Cubriremos técnicas de reconocimiento, explotación de servicios y escalada de privilegios, proporcionando una guía práctica para mejorar tus habilidades en hacking ético.

#### ¿Qué encontrarás aquí?

1. Despliegue y configuración de la máquina "Trust".
2. Escaneo de puertos y servicios.
3. Explotación de servicios web con herramientas como Gobuster e Hydra.
4. Escalada de privilegios para obtener acceso como usuario root.

### Despliegue de la maquina

#### ¡VAMOS A EMPEZAR!

Primero, vamos a ir a Dockerlabs y descargaremos el .ZIP de la máquina Trust.

Después de arrastrar los archivos a una carpeta, nos iremos a la terminal y nos dirigiremos hacia la carpeta con esos archivos.

Cuando estemos en la carpeta, desplegaremos la máquina mediante:

```
sudo bash auto_deploy.sh trust.tar
```

![Imagen maquina](imagenes/Foto_despliegue_maquina.png)

Ya teniendo la IP de la máquina, haremos ping para verificar si hay comunicación y comprobar la conexión:

```
ping <IP máquina>
```

### Escaneo de puertos 

Ahora deberíamos ver qué puertos están abiertos para saber cómo acceder a la máquina. Haremos un escaneo con Nmap utilizando las siguientes opciones:

```
nmap -p- - sS -sV -sC --min-rate 5000 -vvv -n -Pn <IP máquina>
```

![Imagen maquina](imagenes/nmap.png)

Antes de analizar los resultados, vamos a explicar qué hemos hecho en este comando y por qué no hemos utilizado otras opciones:

- Nmap: La herramienta para el escaneo de puertos.
- -p-: Escanea todos los puertos posibles (1-65535).
- -sS: Realiza un escaneo SYN (half-open) que es más rápido y menos detectable.
- -sV: Obtiene la versión de los servicios que están corriendo en los puertos abiertos.
- -sC: Utiliza los scripts por defecto de Nmap para obtener información adicional sobre los puertos y servicios.
- --min-rate 5000: Fuerza a Nmap a enviar una tasa mínima de 5000 paquetes por segundo para acelerar el escaneo.
- -vvv: Muestra información detallada en tiempo real sobre el proceso del escaneo.
- -n: No resuelve nombres de dominio, trabajando solo con direcciones IP.
- -Pn: Asume que el host está en línea y omite la fase de descubrimiento (ping scan).

En este punto, observaremos que los puertos 22 y 80 están abiertos. En este caso, el puerto 22 no lo usaremos de momento, ya que tiene el protocolo SSH y no tenemos ni usuario ni contraseña.

### Gobuster

Vamos a explorar el puerto 80, que utiliza el protocolo HTTP. Probando la IP en el navegador, deberíamos ver una página.

En este caso, estamos viendo una plantilla de Apache y su ubicación en /var/html/index.html.

Ahora sabiendo esto, vamos a usar una herramienta llamada Gobuster que nos ayudará a encontrar directorios ocultos en el servidor web. Además, lo filtraremos por extensiones para encontrar archivos específicos.

```
gobuster dir -u http:/<ip maquina> -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,sh,py
```

Vamos a ver qué hacen estas opciones:

gobuster dir: Ejecuta Gobuster en modo de búsqueda de directorios.
- -u http://<ip máquina>: Especifica la URL objetivo.
- -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt: Define la lista de palabras (wordlist) que Gobuster utilizará para probar nombres de directorios y archivos.
- -x html,php,sh,py: Especifica las extensiones de archivo a buscar (HTML, PHP, SH, PY).

Aquí vemos que nos sale un archivo secret.php (que lo hemos encontrado gracias al filtrado por extensiones).

Volvemos a la URL en el navegador y después de la IP añadimos /secret.php.

```
<Ip maquina>/secret.php
```

En este punto, veremos un mensaje de "¡Hola Mario!" y nos comenta que no se puede hackear. Mario parece estar seguro de ello, pero podemos intentar descifrar su contraseña.

### Hydra

Ya sabemos que el usuario es Mario y que tenemos el puerto SSH abierto. ¿Cómo continuamos? Bueno, usaremos Hydra, ya que nos ayudará a sacar la contraseña mediante un ataque de fuerza bruta.

```
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://<ip maquina> -t 64
```
¿Qué estamos haciendo exactamente?

Hydra: Herramienta para ataques de fuerza bruta en servicios de red.
- -l mario: Especifica el nombre de usuario a utilizar en el ataque.
- -P /usr/share/wordlists/rockyou.txt: Define la lista de contraseñas (wordlist) que Hydra utilizará para probar contra la cuenta de Mario.
- ssh://<ip máquina>: Indica que el objetivo es un servicio SSH en la IP especificada.
- -t 64: Define el número de tareas paralelas que Hydra ejecutará (64 en este caso) para acelerar el proceso.

Aquí veremos que con esta herramienta hemos encontrado la contraseña "chocolate".

Ya que tenemos usuario, contraseña y la IP, el siguiente paso es entrar mediante SSH.

```
ssh mario@<ip maquina>
```

### Estamos dentro

Después de haber entrado mediante SSH, ya vemos que estamos dentro como Mario. Ahora bien, necesitamos buscar binarios que nos ayuden a escalar privilegios.

Primero, probaremos si podemos usar sudo.

```
sudo -l
```

Vemos que Mario tiene permisos para ejecutar vim como superusuario.

Vim es un editor de texto en terminal bastante potente, similar a Nano.

¿Qué hacemos con esta información?

Podemos aprovechar los permisos de vim para ejecutar una shell con privilegios de root.

```
sudo -u root /usr/bin/vim -c `:!/bin/bash`
```

¿Qué hace el comando?

- sudo: Ejecuta un comando como superusuario o como otro usuario.
- -u root /usr/bin/vim: Ejecuta Vim como el usuario root.
- -c ':!/bin/bash': Utiliza la opción -c para ejecutar un comando específico en Vim. En este caso, el comando :!/bin/bash abre una shell (bash) con privilegios de root.

Con todo esto, hemos logrado obtener una shell como usuario root.

#### ¡Gracias por leer y buena suerte con tus futuros ejercicios de hacking ético!

