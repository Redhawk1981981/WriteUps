# 1. Implementar y hackear

## a) Flag de usuario

Comenzaremos haciendo un escaneo básico con Nmap para descubrir los puertos abiertos en la máquina y sus servicios. Usamos:

```
nmap -sV -T5 10.10.115.65
```

Vemos que hay 3 puertos abiertos, el 21, el 22 y el 80.

![](IMG/Pasted%20image%2020250412221046.png)

Ya que tenemos el puerto 21 abierto con un servicio FTP activo, probaremos a conectarnos usando anonymous, por si tuviéramos suerte y estuviera configurado así. Usamos:

```
ftp 10.10.115.65
```

Y vemos, que hemos podido conectarnos de forma anónima sin problemas.

![](IMG/Pasted%20image%2020250412221307.png)

Una vez conectados, listamos los archivos, y vemos que hay un archivo de texto, lo descargamos para poder leerlo, hemos usado:

```
ls
get note_to_jake.txt
```

![](IMG/Pasted%20image%2020250412221449.png)

Al abrirlo, vemos un texto en el que se pide a un tal jake que cambie su contraseña porque es muy débil.

![](IMG/Pasted%20image%2020250412221518.png)

Así que suponemos que la contraseña débil será la de acceso por SSH, que es otro servicio que vimos que está abierto, este, en el puerto 22. Así que probamos a hacer fuerza bruta con hydra sobre este servicio, usando el diccionario Rockyou. Como suponemos que el nombre de usuario será "jake" usaremos hydra así:

```
hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.115.65 ssh
```

Hemos obtenido la contraseña, 987654321, realmente débil.

![](IMG/Pasted%20image%2020250412221918.png)

Teniendo ya las credenciales:

user: jake
password: 987654321

Accedemos por SSH:

```
ssh jake@10.10.115.65
```

![](IMG/Pasted%20image%2020250412222104.png)

Buscaremos ahora el archivo user.txt para ver la flag.

![](IMG/Pasted%20image%2020250412222328.png)

El archivo se encontraba dentro del home del usuario holt.

**Respuesta: ee11cbb19052e40b07aac0ca060c23ee**

## b) Flag root

Tendremos que escalar privilegios para conseguir ver el archivo root.txt, lo primero que haremos será ejecutar sudo -l para ver si tenemos permisos para ejecutar algún comando como sudo.

![](IMG/Pasted%20image%2020250412222610.png)

Vemos que podemos usar el comando less como sudo. Sabiendo esto, consultaremos los [GTFOBins](https://gtfobins.github.io/), que sin binarios que se pueden usar para hacer bypass a la seguridad de un sistema (lo que hicimos en la sala [Rootme](https://github.com/Redhawk1981981/WriteUps/blob/main/13.%20Rootme%20(THM)/WriteUp%20-%20Rootme%20(THM).md), que no entendía el porqué, pues se hizo como esto, pero buscando por python).

![](IMG/Pasted%20image%2020250412222932.png)

![](IMG/Pasted%20image%2020250412222949.png)

Pues bien, usando less, podemos ver el contenido de cualquier archivo como sudo. Así que nosotros usaremos:

```
less /root/root.txt
```

Ya que es la ruta donde suele estar siempre el archivo con la flag root en Tryhackme.

![](IMG/Pasted%20image%2020250412223143.png)

**Respuesta: 63a9f0ea7bb98050796b649e85481845**