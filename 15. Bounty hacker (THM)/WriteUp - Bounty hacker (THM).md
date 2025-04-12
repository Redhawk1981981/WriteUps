# 1. Haciendo honor al título

Presumías sin parar de tus habilidades de hacker de élite en el bar y unos cuantos cazarrecompensas decidieron tomarte la palabra. Demuestra que tu estatus va más allá de unas copas en la barra. ¡Presiento pimientos y carne en tu futuro! 

## a) Implementar la máquina.

**Respuesta: No aplica**

## b) Encuentra puertos abiertos en la máquina

Lo primero que hacemos es un escaneo Nmap básico para conocer los puertos abiertos y los servicios. Usamos:

```
nmap -sV -T5 10.10.28.196
```

vemos que hay 3 puertos abiertos, el 21, el 22 y el 80

![](IMG/Pasted%20image%2020250412224309.png)

**Respuesta: No aplica.**

Como vemos que tenemos un servicio FTP corriendo, lo primero que haremos será probar si tiene configurado el acceso de forma anónima y así podemos conectarnos.

```
ftp 10.10.28.196
```

Nos conectamos sin problema.

![](IMG/Pasted%20image%2020250412224841.png)

Una vez conectados, listamos los archivos para ver qué hay.

![](IMG/Pasted%20image%2020250412230203.png)

Desactivamos el modo pasivo, que nos da problemas con al conexión a tryhackme.

![](IMG/Pasted%20image%2020250412230258.png)

Ya nos muestra los ficheros, vemos que son dos archivos de texto, los descargamos para poder verlos.

![](IMG/Pasted%20image%2020250412230456.png)

## c) ¿Quién escribió la lista de tareas? 

Al abrir el archivo task.txt que hemos descargado del ftp, podemos ver quién la escribió.

![](IMG/Pasted%20image%2020250412230754.png)

**Respuesta: Lin**

## d) ¿Qué servicio puede utilizar para forzar el archivo de texto encontrado?

Abrimos el otro archivo que descargamos del FTP.

![](IMG/Pasted%20image%2020250412231015.png)

Y parece ser que es un archivo con posibles contraseñas, así que trataremos de usar este archivo con Hydra para ver si podemos conectarnos por SSH, suponiendo que el nombre de usuario puede ser "lin" que es el que creó la lista de tareas. Así que usamos:

```
hydra -l lin -P locks.txt 10.10.28.196 ssh
```

Y obtenemos la contraseña, RedDr4gonSynd1cat3

![](IMG/Pasted%20image%2020250412231524.png)

**Respuesta: SSH**

## e) ¿Cuál es la contraseña del usuario? 

Como hemos visto antes, la contraseña es RedDr4gonSynd1cat3

**Respuesta: RedDr4gonSynd1cat3**

## f) user.txt

Nos conectamos por SSH con las credenciales que tenemos:

user: lin
password: RedDr4gonSynd1cat3

```
ssh lin@10.10.28.196
```

![](IMG/Pasted%20image%2020250412232616.png)

Una vez conectados, listamos los archivos y podremos abrir el archivo user.txt para ver la flag.

![](IMG/Pasted%20image%2020250412232654.png)

**Respuesta: THM{CR1M3_SyNd1C4T3}**

## g) root.txt

Haremos lo que ya hemos hecho en otras ocasiones, ejecutar sudo -l para ver si podemos ejecutar algún comando como sudo que nos permita ver el contenido del archivo root.txt.

![](IMG/Pasted%20image%2020250412232906.png)

Vemos que podemos usar el comando tar como root, por lo que ahora consultaremos los [GTFObins](https://gtfobins.github.io/), para ver de que manera podemos usar este comando para obtener un acceso root. Estos son los resultados que hemos encontrado.

![](IMG/Pasted%20image%2020250412233054.png)

Así que ejecutamos:

```
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Y obtenemos acceso con una shell root sin problemas.

![](IMG/Pasted%20image%2020250412233240.png)

Hacemos directamente un cat a la ruta en la que suelen estar los root.txt en tryhackme.

![](IMG/Pasted%20image%2020250412233340.png)

Y obtenemos la flag.

**Respuesta: THM{80UN7Y_h4cK3r}**