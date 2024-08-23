## MÁQUINA INJECTION

¡VAMOS A EMPEZAR!
![hola](Dockerlabs/Foto_de_la_maquina_injection.png)
Primero, vamos a ir a Dockerlabs y descargaremos el .ZIP de la máquina Injection.

Después de arrastrar los archivos a una carpeta, nos iremos a la terminal y nos dirigiremos hacia la carpeta con esos archivos.

Cuando estemos en la carpeta, desplegaremos la máquina mediante:

```
sudo bash auto_deploy.sh injection.tar
```

Ya teniendo la IP de la máquina, haremos ping para verificar si hay comunicación y comprobar la conexión:

```
ping <IP máquina>
```

Ahora deberíamos ver qué puertos están abiertos para saber cómo acceder a la máquina. Haremos un escaneo con NMAP utilizando las siguientes opciones:

```
nmap -p- --open -sT --min-rate 5000 -vvv -n -Pn <IP máquina>
```

Antes de analizar los resultados, vamos a explicar qué hemos hecho en este comando y por qué no hemos utilizado otras opciones:

Nmap: La herramienta para el escaneo de puertos.
-p-: Escanea todos los puertos.
--open: Muestra solo los puertos abiertos.
-sT: No requiere permisos root, funciona en cualquier entorno, establece una conexión más completa y, además, confirma que el puerto realmente está disponible para la comunicación.

¿Y por qué no -sS?
Es cierto que es más rápido, sigiloso, más eficiente, y es el preferido para las auditorías, pero requiere permisos root y no establece una conexión completa.

--min-rate 5000: Fuerza a Nmap a enviar una tasa mínima de paquetes.
-vvv: Proporciona información muy detallada sobre lo que hace Nmap y muestra los resultados durante el escaneo.
-n: No intenta resolver las direcciones IP a nombres de dominio.
-Pn: Asume que el host está en línea y no realiza ping.

(Opcional pero recomendable) 
-sV: Para ver la versión de los servicios en los puertos escaneados.

Ahora bien, en este punto observaremos que los puertos 22 y 80 están abiertos. En este caso, el puerto 22 no lo usaremos de momento, ya que tiene el protocolo SSH y no tenemos ni usuario ni contraseña.

Así que vamos a explorar el puerto con el protocolo HTTP. Con esto podemos probar si al poner la IP en el navegador podemos ver una página.

En este caso, estamos viendo una página de registro. Aquí podremos hacer dos cosas:

1. Probar todas las combinaciones comunes que suelen haber por defecto (root, admin, user, password, 123...).

2. Probar SQL Injection (SQLi). Antes de inyectar código SQL, quiero matizar que hay tres tipos de SQLi:
2.1. In-Band.
2.2. Out-of-Band.
2.3. Blind.
   
Viendo que hay tres tipos, debemos saber cuál vamos a utilizar y el porqué. En nuestro caso, usaremos el primer tipo, In-Band, ya que es el más común y es el que nos sirve ahora para hacer este ejercicio de una manera más rápida. Aun así, recomiendo que exploren los otros dos tipos para ver qué función tienen.

Ahora bien, ¿cómo atacaremos esta página de registro? Muy fácil. Usaremos este payload:

```
' OR 1=1 --
```

Al igual que antes con NMAP, vamos a ver qué hace este payload:

': Esta comilla cierra la cadena de texto que debería contener un valor (en este caso, el usuario).
OR 1=1: Nos ayuda a que esta condición siempre sea verdadera. Al usar OR, se está diciendo "selecciona registros donde la condición original sea cierta o donde 1 sea igual a 1", lo que significa que siempre seleccionará registros.
--: Esto nos evita la molestia de pensar cuál es la contraseña, ya que el código siguiente a esto estará comentado.
(Información adicional)
-- o -- -: Ambos hacen la misma función.
': Antes de la comilla simple, puedes poner algo si te apetece.
¿Se pueden usar otros métodos? Sí, UNION, por ejemplo, pero eso lo probaremos en otras máquinas.

Vamos a verlo un poco mejor con este ejemplo:

El registro con código SQL:

```
SELECT * FROM users WHERE username = 'usuario' AND password = 'contraseña';
```
El registro con código SQL y con la inyección:

```
SELECT * FROM users WHERE username = '' OR 1=1 -- ' AND password = '';
```

Ahora, después de todo esto, ya podemos ver un mensaje que dice "¡Bienvenido, Dylan!" y con una cadena de números y letras que podemos pensar que es una contraseña.
