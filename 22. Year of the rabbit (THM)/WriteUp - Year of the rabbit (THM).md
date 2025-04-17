# 1. Flags

¡Que empiece el Año Nuevo con buen pie!  
¿Podrás acceder a la caja del Año del Conejo sin caerte por un agujero?

## a) Cuál es la flag de usuario?

Empezamos con un escaneo básico con nmap para descubrir puertos abiertos y servicios.

```
nmap -sV -T5 10.10.153.64
```

Vemos que la máquina objetivo tiene 3 puertos abiertos, el 21 con un servicio FTP, el 22 con un servicio SSH y el 80 con un servidor web Apache.

![](IMG/Pasted%20image%2020250417211819.png)

Lo primero que haremos será intentar conectar por FTP de forma anónima, por si lo tuviera habilitado.

```
ftp 10.10.153.64
```

Y vemos que no, no podemos acceder.

![](IMG/Pasted%20image%2020250417212001.png)

Así que echaremos un vistazo a la página web. Recibimos la página por defecto de Apache y en el código fuente de la página no hay nada fuera de lo normal.

![](IMG/Pasted%20image%2020250417212114.png)

No tiene tampoco "robots.txt".

![](IMG/Pasted%20image%2020250417212243.png)

Así que listaremos los directorios usando Gobuster a ver si encontramos algo interesante.

```
gobuster dir -u http://10.10.153.64 -w /usr/share/dirb/wordlists/common.txt
```

Y descubrimos un directorio llamado /assets. Lo visitaremos.

![](IMG/Pasted%20image%2020250417212400.png)

Dentro de /assets, encontramos dos archivos.

![](IMG/Pasted%20image%2020250417212439.png)

El primero de ellos "RickRolled.mp4" se trata del video músical de Rick Astley de su canción "Never gonna give you up", que aunque sea un temazo, no nos aporta nada.

![](IMG/Pasted%20image%2020250417212543.png)

El segundo archivo "style.css", es una hoja de estilos html, la cual nos da el siguiente mensaje:

![](IMG/Pasted%20image%2020250417212757.png)

Nos dice que echemos un vistazo a la página /sup3r_s3cr3t_fl4g.php, lo hacemos.

![](IMG/Pasted%20image%2020250417212902.png)

Nos indica que desactivemos javascript y nos redirecciona de nuevo al video de antes.

Así que desactivamos javascript en Brave.

![](IMG/Pasted%20image%2020250417213336.png)

Y accedemos de nuevo, nos vuelve a aparecer el video de antes, y nos dice que la pista está en el video. Tras revisar bien el video, el código fuente de la página, y alguna que otra prueba mas con el video, realizo una búsqueda en internet por alguna pista, pues ya no sabía mas que hacer, y nos sugieren que analicemos la petición con "Burpsuite", así que usando "Zap", descubrimos un directorio oculto /WExYY2Cv-qU

![](IMG/Pasted%20image%2020250417214513.png)

Accedemos a él y encontramos un archivo de imagen.

![](IMG/Pasted%20image%2020250417214706.png)

La abrimos.

![](IMG/Pasted%20image%2020250417214732.png)

No nos dice nada, así que supongo que la imagen ocultará algo. Comenzamos ejecutando el comando strings sobre la imagen.

```
strings Hot_Babe.png
```

Y encontramos una pista, el nombre de usuario para el servicio FTP (ftpuser) y un listado con contraseñas y se nos indica que una de ellas es la correcta.

![](IMG/Pasted%20image%2020250417220045.png)

Creamos un fichero txt con el listado de contraseñas.

![](IMG/Pasted%20image%2020250417220241.png)

Y usaremos Hydra sobre el servicio FTP usando el archivo de contraseñas que hemos creado.

```
hydra -l ftpuser -P rabbit.txt ftp://10.10.153.64
```

Obtenemos la contraseña.

![](IMG/Pasted%20image%2020250417220447.png)

Ya con las credenciales, tratamos de conectarnos por FTP:

```
ftp ftpuser@10.10.153.64
```

Nos conectamos sin problemas.

![](IMG/Pasted%20image%2020250417220708.png)

Una vez conectados, listamos el contenido y vemos que hay un archivo de texto.

![](IMG/Pasted%20image%2020250417220811.png)

Lo descargamos para ver su contenido.

```
get Eli's_Creds.txt
```

![](IMG/Pasted%20image%2020250417221143.png)

Vemos el siguiente contenido

![](IMG/Pasted%20image%2020250417221216.png)

Me suena el código pues lo estuve viendo para hacer un reto CTF basado en él, se trata del lenguaje "Brainfuck", el cual dicen que es el lenguaje mas complicado que existe. Lo copiamos y usamos alguna web que nos los descifre, por ejemplo [md5decrypt](https://md5decrypt.net/en/Brainfuck-translator/).

Nos da como resultado un nombre de usuario y una contraseña.

![](IMG/Pasted%20image%2020250417221649.png)

User: eli
Password: DSpDiM1wAEwid

Como ya tenemos acceso por FTP, no tiene sentido que estas credenciales sean para el mismo servicio, así que intentamos conectarnos con ellas por SSH.

(Se me ha acabado el tiempo en la máquina de Tryhackme y no me he dado cuenta, así que vuelvo a arrancarla, cambiará la IP).

```
ssh eli@10.10.223.218
```

Se conecta sin problemas y nos da una pista, nos indica que accedamos al directorio secreto "s3cr3t" en el que hay un mensaje oculto.

![](IMG/Pasted%20image%2020250417222153.png)

Así que buscamos la ruta del directorio.

```
find / -name s3cr3t 2> /dev/null
```

![](IMG/Pasted%20image%2020250417222923.png)

Accedemos a la ruta encontrada y listamos los archivos (ocultos incluidos)

![](IMG/Pasted%20image%2020250417223102.png)

Abrimos el archivo y vemos que en el texto que contiene nos muestra otro password, este suponemos que será el password de usuario en el sistema para el usuario "Gwendoline", pues lo escribe un tal "root".

![](IMG/Pasted%20image%2020250417223224.png)

Así que tratamos de identificarnos como root con ese usuario y esa contraseña.

user: gwendoline
pass: MniVCQVhQHUNI

```
su gwendoline
```

Cambiamos correctamente de usuario, y podemos buscar el archivo user.txt para obtener la flag de usuario.

![](IMG/Pasted%20image%2020250417223714.png)

**Respuesta: THM{1107174691af9ff3681d2b5bdb5740b1589bae53}**

## b) Cuál es la flag de root?

Comprobaremos en primer lugar si tenemos algún permiso como root con el usuario usando sudo -l.

Vemos que tenemos permisos de root usando el editor de texto vi, sobre el archivo user.txt

![](IMG/Pasted%20image%2020250417224027.png)

Tras buscar en internet encontramos una manera de explotarlo, usando la vulnerabilidad con [CVE-2019-14287](https://nvd.nist.gov/vuln/detail/CVE-2019-14287)

![](IMG/Pasted%20image%2020250417224826.png)

Para explotarla, ejecutamos lo siguiente:

```
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```

Con lo que accedemos al archivo user.txt como root.

![](IMG/Pasted%20image%2020250417224942.png)

Ahora, según nos indican, solo tenemos que llamar una consola de comandos bash usando:

```
:!/bin/bash
```

![](IMG/Pasted%20image%2020250417225227.png)

Al pulsar intro, obtenemos un shell root.

![](IMG/Pasted%20image%2020250417225307.png)

Ahora, solo nos queda buscar el archivo root.txt (que está en el sitio clásico /root/root.txt) y obtener la flag.

![](IMG/Pasted%20image%2020250417225348.png)

**Respuesta: THM{8d6f163a87a1c80de27a4fd61aef0f3a0ecf9161}**