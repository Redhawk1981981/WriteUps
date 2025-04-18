# 1. Encuentra las banderas

¡Bienvenidos a Lian_YU, esta caja CTF para principiantes con temática de Arrowverse! ¡Captura las banderas y diviértete!

## a) ¿Cuál es el directorio web que encontraste?

Lo primero que hacemos, como siempre, es un escaneo básico con nmap para descubrir los puertos abiertos y los servicios que están ejecutandose.

```
nmap -sV -T5 10.10.32.130
```

Vemos que la máquina objetivo tiene 4 puertos abiertos, el 21 con un servicio FTP, el 22 con un servicio SSH, el 80 con un servidor web Apache y el 111 con un servicio rpcbind.

![](IMG/Pasted%20image%2020250417231245.png)

Para comenzar, probamos a conectarnos por FTP de forma anónima y no está habilitado.

![](IMG/Pasted%20image%2020250417231420.png)

Así que accedemos a la web y recibimos la siguiente página en la que se nos hace una introducción de la sala, pero nada que parezca útil. En el código fuente no encontramos nada fuera de lo normal.

![](IMG/Pasted%20image%2020250417231457.png)

Como en otras ocasiones, usaremos Gobuster para listar los directorios de la página.

```
gobuster dir -u http://10.10.32.130 -w /usr/share/dirb/wordlists/common.txt
```

No nos devuelve gran cosa.

![](IMG/Pasted%20image%2020250417232112.png)

Probaremos de nuevo cambiando el diccionario por otro algo menos básico, aunque por lo tanto, tarde algo más.

```
gobuster dir -u http://10.10.32.130 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Ahora sí, antes de terminar incluso el escaneo (que dejaremos que siga mientras probamos), nos muestra un directorio llamado /island

![](IMG/Pasted%20image%2020250417232640.png)

Accedemos al directorio descubierto y obtenemos el siguiente mensaje:

![](IMG/Pasted%20image%2020250417232731.png)

Nos indica el código, pero no está. Viendo el código fuente de la página descubrimos que está escrito con letras blancas para que no se vea con el fondo.

![](IMG/Pasted%20image%2020250417232829.png)

Si seleccionamos el texto con el ratón, también se ve.

![](IMG/Pasted%20image%2020250417232923.png)

El código es "vigilante"

No tenemos mucho más, así que esperamos a que termine el escaneo con Gobuster que habíamos lanzado antes a ver si nos descubre algo más.

![](IMG/Pasted%20image%2020250417234828.png)

No nos descubre nada más, así que lanzaremos de nuevo Gobuster, pero en esta ocasión sobre el directorio que ya conocemos /island, a ver si tenemos mas suerte.

```
gobuster dir -u http://10.10.32.130/island -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Antes de terminar el propio escaneo, nos da un directorio llamado /2100. Lo visitaremos mientras continúa el escaneo.

![](IMG/Pasted%20image%2020250417233503.png)

Al visitar el directorio /2100, obtenemos lo siguiente:

![](IMG/Pasted%20image%2020250417233655.png)

 **Respuesta: 2100**

## b) ¿Cuál es el nombre del archivo que encontraste?

No podemos ver el video pues ya no está disponible, pero si miramos el código fuente de la página, encontramos la siguiente pista:

![](IMG/Pasted%20image%2020250417233752.png)

Nos dice algo de un ticket, pero no se muy bien donde puede estar, esperaremos a que termine el escaneo que aún se está realizando sobre /island por si encontráramos algún directorio adicional.

![](IMG/Pasted%20image%2020250418000143.png)

Como no nos ha descubierto nada nuevo, probaré de nuevo a hacer un escaneo con Gobuster, pero ahora sobre el directorio /2100

```
gobuster dir -u http://10.10.32.130/island/2100 -t 40 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

![](IMG/Pasted%20image%2020250418003638.png)

Y, no nos da ningún resultado, lo cuál me parece extraño.
Tras revisarlo todo de nuevo por si me hubiese saltado alguna pista, me he dado cuenta de que se nos hablaba de .ticket por lo que puede ser que no sea un directorio lo que buscamos, sino un archivo con esa extensión. Así que repetiré la búsqueda con Gobuster sobre el directorio /2100 pero buscando por archivos con la extensión .ticket. Realizamos la búsqueda así:

```
gobuster dir -u http://10.10.32.130/island/2100 -t 40 -x ticket -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Hemos tenido suerte y antes incluso de terminar el propio escaneo, ya nos muestra un archivo con extension .ticket

![](IMG/Pasted%20image%2020250418002841.png)

**Respuesta: green_arrow.ticket**

## c) ¿Cuál es la contraseña FTP?

Si visualizamos el archivo, por ejemplo, en el propio navegador, obtenemos un token.

![](IMG/Pasted%20image%2020250418003046.png)

Lo pasaremos a [Cyberchef ](https://gchq.github.io/CyberChef/)dicho token para comprobar si esta codificado de algún modo y oculta alguna información.

Probando a decodificar desde las diferentes Bases, descubrimos que está codificado en Base58, y nos da el siguiente resultado:

![](IMG/Pasted%20image%2020250418003711.png)

Suponemos que puede ser una contraseña, por lo que ya tendríamos unas credenciales posibles:

user: vigilante
pass: !#th3h00d

**Respuesta: !#th3h00d**

## d) ¿Cuál es el nombre del archivo con la contraseña SSH?

Intentaremos conectar usandolas por FTP, y conseguimos conectarnos correctamente.

![](IMG/Pasted%20image%2020250418004110.png)

Una vez conectados, listaremos el contenido y veremos varios archivos. Los descargaremos y les echaremos un vistazo.

![](IMG/Pasted%20image%2020250418004213.png)

![](IMG/Pasted%20image%2020250418004358.png)

Comenzaremos por el archivo que no es una imagen, ".other_user", el cual nos habla de otro personaje, Slade Wilson.

![](IMG/Pasted%20image%2020250418004608.png)

La primera de las imágenes, "Queen's gambit.png" Nos muestra el barco de los Queen.

![](IMG/Pasted%20image%2020250418012316.png)

La segunda, "Leave_me_alone.png", no muestra nada.

![](IMG/Pasted%20image%2020250418012413.png)

Y la tercera y ultima imagen "aa.jpg" muestra la imagen del personaje Slade Wilson.

![](IMG/Pasted%20image%2020250418012514.png)

A simple vista ninguna de las imágenes muestra nada interesante, así que supongo que pueden tener algo oculto con esteganografía. Así que analizamos cada una de ellas con Steghide para comprobar en primer lugar si ocultan algo.

Vemos, que de las tres imágenes, "aa.jpg" parece tener algo oculto, y está protegido con contraseña.

![](IMG/Pasted%20image%2020250418013417.png)

Como no tenemos diccionario que se nos haya proporcionado, trataremos de usar el diccionario "Rockyou.txt" por si tuviéramos suerte y la contraseña estuviera en el. Usaremos "Stegseek" para tratar de hacer fuerza bruta. Lo usamos así:

```
stegseek -sf aa.jpg /usr/share/wordlists/rockyou.txt
```

Hemos tenido suerte y encontramos la siguiente información:

![](IMG/Pasted%20image%2020250418014045.png)

Hay un archivo zip y la contraseña era "password". Ya ha extraído el contenido en el archivo "aa.jpg.out", el cual abriremos para ver que esconde. Como sabemos que es un zip, pues nos indicaba "stegseek" que el archivo original se llamaba "ss.zip", lo descomprimiremos primero.

```
unzip aa.jpg.out
```

Nos ha dado como resultado dos archivos, "passwd.txt" y "shado"

![](IMG/Pasted%20image%2020250418014342.png)

Abrimos primero el archivo "shado", y vemos lo que podría ser un nombre de usuario o una contraseña.

![](IMG/Pasted%20image%2020250418014439.png)

Y en "passwd.txt" encontramos este texto hablando sobre el personaje Oliver Queen:

![](IMG/Pasted%20image%2020250418014523.png)

Tenemos varías posibilidades de nombre de usuario y contraseña, pero no se cuál es cuál, tengo slade, oliver, queen, gambit y M3tahuman. Que son las que aparentemente me han ido resultando posibles. He probado combinaciones con ellas para conectarme por SSH, y finalmente las credenciales correctas eran:

user: slade
pass: M3tahuman

Es lo mas lógico pues esta parte del reto estaba muy dedicada a este personaje y es un metahumano.

![](IMG/Pasted%20image%2020250418015052.png)

**Respuesta: shado**

## e) user.txt

Una vez conectados por SSH, buscaré el archivo user.txt para obtener la flag.

![](IMG/Pasted%20image%2020250418015136.png)

**Respuesta: THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}**

## f) root.txt

Ahora, trataré de escalar privilegios para obtener la flag de root. Lo primero que haremos será comprobar los privilegios de root que tiene el usuario usando sudo -l, y vemos que puede ejecutar el comando pkexec como root. Es un comando de linux que nos permite ejecutar archivos con los permisos de otro usuario.

![](IMG/Pasted%20image%2020250418015338.png)

Buscaremos, como ya hemos hecho otras veces en [GTFOBins](https://gtfobins.github.io/).

![](IMG/Pasted%20image%2020250418015746.png)

Y usaremos el comando sugerido:

```
sudo pkexec /bin/sh
```

Inmediatamente, obtenemos una shell root.

![](IMG/Pasted%20image%2020250418015941.png)

Podemos buscar el archivo root.txt (esta vez no está en /root/root.txt), y obtenemos la flag.

![](IMG/Pasted%20image%2020250418020038.png)

**Respuesta: THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}**