# 1) Objetivo

El objetivo en esta máquina, es conseguir la flag que se encuentra dentro del archivo root.txt.
Nos advierten que el tema y las advertencias de seguridad que encontramos en esta prueba, son parte del desafío.

# 2) Reconocimiento y obtener acceso

Para comenzar, como siempre, haremos un escaneo de puertos con nmap. Hemos ejecutado:

```
nmap -A -sV 10.10.96.153
```

![](IMG/Pasted%20image%2020250304005436.png)

Descubrimos que la maquina tiene 4 puertos abiertos, el 80, con un servidor web, el 22, con un servicio ssh y el 139 y el 445 ambos con un servicio samba de windows.

Primero miramos la pagina web, a ver si hay algo interesante.

Nada, solo la pagina por defecto de un servidor web apache.

![](IMG/Pasted%20image%2020250304005711.png)

Usaremos feroxbuster (hasta ahora hemos usado en todas las máquinas gobuster, pero probaremos ahora con feroxbuster, para variar un poco y porque segun he leido en internet, es mas rápido y muestra los resultados de forma mas clara) para tratar de conseguir un listado de los directorio para ver si encontramos algo más. Ejecutamos:

Si queremos usarlo con el listado por defecto:

```
feroxbuster --url http://10.10.96.153/
```

Si quisieramos usarlo con un listado concreto añadiríamos:

```
--wordlist /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt  
```

En nuestro caso, hemos obtenido mejor resultado con el listado por defecto.

![](IMG/Pasted%20image%2020250304012643.png)

Hemos encontrado un directorio llamado /test, y otro llamado /wordpress, el cual parece que tiene el panel de administración en /wp-admin.

Comprobamos ambos directorios mientras sigue el escaneo de feroxbuster en busca de algo más.

Entrando a /test, encontramos diverssos popups de alerta, son los que nos habian advertido en el enunciado, no podemos hacer nada, ni cerrarlos ni nada.

![](IMG/Pasted%20image%2020250304012946.png)

Si accedemos a wordpress/wp-admin, efectivamente, es el panel de administración de wordpress, necesitaríamos un usuario y contraseña.

![](IMG/Pasted%20image%2020250304013122.png)

Mientras tanto, ya ha terminado el escaneo con feroxbuster, y no ha encontrado nada más que parezca interesante, como vemos en el resumen final:

![](IMG/Pasted%20image%2020250304013214.png)

Visitamos también la pagina de wordpress por si hubiera algo interesante, pero no hay nada especial o que me llame la atención.

![](IMG/Pasted%20image%2020250304013508.png)

Y de momento poco más que exprimir al wordpress, necesitamos un usuario y contraseña, o encontrar otra forma de obtener acceso. Probaremos ahora con los puertos Samba de Windows, que como vimos antes la máquina tiene dos puertos abiertos con este servicio. Para hacer un escaneo/enumeración de estos, usaremos enum4linux, que es una aplicación que sirve exactamente para eso, para enumerar servicios samba de windows, así probaremos si conseguimos algo de información útil. Ejecutamos:

```
enum4linux 10.10.96.153
```

![](IMG/Pasted%20image%2020250304014952.png)

De entre toda la información que obtenemos, la mas interesante es la siguiente:

Nos dice que el servidor permite iniciar sesión sin usuario ni contraseña:

![](IMG/Pasted%20image%2020250304015145.png)

Nos dice los directorios/servicios compartidos, entre ellos, el mas interesante es lo que parece ser una unidad de disco compartida con el nombre "websvr".

![](IMG/Pasted%20image%2020250304015328.png)

Y obtenemos nombres de usuario permitidos en el servidor, entre ellos, me llama la atención uno concretamente, que no parece ser un usuario genérico, el que tiene el nombre "scamsite".

![](IMG/Pasted%20image%2020250304015456.png)

Y esa es toda la información relevante conseguida con esta herramienta.

Pues lo siguiente que hacemos, viendo que no se necesita usuario ni contraseña para acceder por samba, lo intentamos, para ello usamos "smbclient", que es una herramienta que nos permite conectarnos a un servidor samba. Lo usamos así (si, hacen falta ese chorro de contra-barras):

```
smbclient \\\\10.10.96.153\\websvr
```

![](IMG/Pasted%20image%2020250304020304.png)

Y efectivamente, nos ha dejado conectar solo pulsando intro, sin introducir contraseña, así que listamos el contenido usando:

```
dir
```

Vemos un archivo llamado "enter.txt", lo descargamos a nuestra máquina atacante para poder abrirlo y ver que contiene, para ello ejecutamos:

```
get enter.txt
exit
```

![](IMG/Pasted%20image%2020250304020538.png)

Ahora, en nuestra máquina, abrimos el archivo.

```
nano enter.txt
```

![](IMG/Pasted%20image%2020250304020709.png)

En el archivo, encontramos varias pistas:

- Un directorio que desconocíamos, llamado /subrion, que según nos indican no funciona y que hay que editarlo desde el panel.
- Las credenciales para dicho directorio, deduzco que para el panel de edición que nos menciona. el usuario parece ser admin y aparece el hash de la contraseña que según nos dicen está "cocinada con una formula mágica". lo de "cocinada" puede referirse a cyberchef, página que nos ayuda a cifrar descifrar hashes de contraseñas, así que probaremos eso lo primero.

Accedemos a la web de cyberchef y agregamos en input el hash que hemos obtenido, el cual es Base64, lo que nos da como output "Scam2021"

![](IMG/Pasted%20image%2020250304021841.png)

Ya aparentemente tenemos un usuario (admin), y una contraseña (Scam2021), probaremos a acceder al panel que nos mencionaban.

```
10.10.96.153/subrion/panel
```

Y efectivamente, tenemos el acceso a un panel de administración.

![](IMG/Pasted%20image%2020250304022153.png)

Usaremos las credenciales que tenemos, admin y Scam2021, y conseguimos acceder al panel de administración de lo que es un CMS subrion.

![](IMG/Pasted%20image%2020250304022307.png)

Echando un vistazo al panel, buscamos la versión del CMS subrion, para, como ya hemos hecho en otras ocasiones, buscar una vulnerabilidad de dicho cms. vemos en la esquina inferior izquierda, muy chiquitito, que es la versión 4.2.1.

![](IMG/Pasted%20image%2020250304022539.png)

En realidad, ahora que lo pienso, también teníamos la versión en la pantalla de login, lo que es el ansía por usar las credenciales descubiertas...

![](IMG/Pasted%20image%2020250304022733.png)

Pues ahora, sabiendo la versión, usaremos searchsploit para buscar un exploit que pueda servirnos. Ejecutamos:

```
searchsploit subrion cms
```

![](IMG/Pasted%20image%2020250304023021.png)

Usaremos el archivo de python 49346.py, que nos permite subir archivos de forma arbitraria.

![](IMG/Pasted%20image%2020250304023725.png)

Para poder usarlo, primero debemos "descargarlo" desde el "almacén" de searchsploit, para lo que usamos:

```
searchsploit -m php/webapps/49876.py
```

![](IMG/Pasted%20image%2020250304023902.png)

Ahora, vamos a la ruta donde se ha descargado (en la que estamos vaya...), y lo ejecutamos, poniendo la ip de la máquina objetivo con la ruta del panel de administración y las credenciales que tenemos, quedando el comando así:

```
python3 49876.py -u http://10.10.96.153/subrion/panel/ -l admin -p Scam2021
```

![](IMG/Pasted%20image%2020250304024514.png)

Y ha funcionado sin problemas, dandonos una shell, ahora, podemos intentar ver un listado de los usuarios registrados, para ello usamos:

```
cat /etc/passwd
```

vemos que hay un usuario llamado "scamsite", por lo que ya tendríamos el nombre de usuario registrado en wordpress, nos haría falta la contraseña.

![](IMG/Pasted%20image%2020250304024837.png)

buscaremos entonces mas información, si hacemos:

```
ls
```

nos lista el contenido del directorio en el que estamos, en el que solo está el archivo subido por el exploit, por lo que podemos navegar un directorio atrás y listarlo, con:

```
ls ../
```

y veremos mas directorios y archivos, pero nada relacionado con wordpress.

![](IMG/Pasted%20image%2020250304025439.png)

Probamos navegando dos directorios atrás:

```
ls ../../
```

![](IMG/Pasted%20image%2020250304025530.png)

Y aquí si vemos el directorio "wordpress" entramos al directorio y listamos el contenido, usando:

```
ls ../../wordpress
```

Y encontramos el archivo de configuración de wordpress, donde debemos encontrar, o eso espero, alguna credencial de wordpress. Lo abrimos para verlo usando:

```
cat ../../wordpress/wp-config.php
```

![](IMG/Pasted%20image%2020250304030157.png)

Hemos encontrado la contraseña para la base de datos de wordpress, ahora tenemos dos credenciales nuevas, el nombre de usuario que encontramos antes, scamsite, y la contraseña ImAScammerLOL!123! con estas credenciales, he intentado conectarme al panel de administración de Wordpress, pero no son correctas, por lo que he probado a conectarme por ssh, que recordemos, tenemos el puerto 22 abierto, de este modo si conseguimos conectarnos, podremos ver los archivos en busca de root.txt. así que lo probamos ejecutando:

```
ssh scamsite@10.10.96.153
```

y usamos como contraseña ImAScammerLOL!123!

![](IMG/Pasted%20image%2020250304030644.png)

Hemos conseguido conectarnos por ssh, sin problemas. Ahora, como ya hicimos en máquinas anteriores, lo primero que haremos será listar los permisos sudo que este usuario tiene en el servidor ssh, para ello ejecutamos:

```
sudo -l
```

![](IMG/Pasted%20image%2020250304030913.png)

Nos dice que podemos usar como sudo el comando iconv, que según he encontrado en internet, nos permite cambiar la codificación del formato de archivos. Cosa con lo que no tengo ni idea de que puedo hacer.

He intentado entrar a un directorio /root por si existiera, y si, existe, pero no tengo acceso.

![](IMG/Pasted%20image%2020250304031850.png)

Ya aquí no sabía que mas hacer, pero buscando en internet, usando iconv, y suponiendo que el archivo root.txt que buscamos está dentro del directorio /root, si usamos:

```
sudo iconv -f 8859_1 -t 8859_1 "/root/root.txt"
```

En el que por lo visto usamos el formato de codificación 8859_1 para la entrada y el mismo para la salida, del archivo root.txt, vemos el contenido del archivo. (esta parte para mí basicamente me ha parecido magia porque no conocía el comando iconv, y de no ser por haber encontrado ayuda en internet, no habría sabido usarlo)

![](IMG/Pasted%20image%2020250304032254.png)

El caso es que hemos conseguido la flag que estaba dentro del archivo root.txt.

**root.txt: 851b8233a8c09400ec30651bd1529bf1ed02790b**