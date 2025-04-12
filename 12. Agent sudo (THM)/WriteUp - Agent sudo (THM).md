# 1. Enumerar

Enumere la máquina y obtenga toda la información importante

## a) ¿Cuántos puertos abiertos?

Lo primero que hacemos es un escaneo Nmap básico para averiguar los puertos abiertos y sus servicios. Usamos:

```
nmap -sV -T5 10.10.88.160
```

Y obtenemos 3 puertos abiertos, el resto se nos muestran como filtrados, quiere decir que están tras un firewall.

![](IMG/Pasted%20image%2020250412011440.png)

**Respuesta: 3**

## b) ¿Cómo te redireccionas a una página secreta?

Al acceder al servidor web que vimos que corre en el puerto 80, nos aparece una web en la que nos dice que nos identifiquemos con nuestro nombre en código de agente para acceder al sitio. Vemos que lo ha escrito el agente "R", por lo que podemos deducir que los nombres secretos de los agentes quizás sean una letra, como los agentes de men in black.

![](IMG/Pasted%20image%2020250412011655.png)

Pues usaremos curl para realizar peticiones a la web pudiendo cambiar el user-agent, de modo que veamos la respuesta de la página. Probaremos a usar como user-agent el propio agente R.

```
curl "10.10.88.160" -H "User-Agent: R" -L
```

![](IMG/Pasted%20image%2020250412012330.png)

Nos indica que no es correcto, y nos da una pista del número de empleados (agentes) que hay, son 25 empleados, por lo que deberemos probar todas las posibilidades desde la A. Como son pocas opciones, lo haremos a mano, su fueran muchas podríamos hacerlas con un ataque, por ejemplo, usando zap.

Después de probar desde la A a la Z, el único que nos arroja algún resultado diferente es el agente C.

![](IMG/Pasted%20image%2020250412012909.png)

En este mensaje se dirigen al agente C por su nombre, Chris, y le piden que cambie su contraseña, que es débil.

Como lo que hemos usado para acceder a la web es el user-agent, esa es la respuesta.

**Respuesta: user-agent**
## c) ¿Cual es el nombre del agente?

Como hemos dicho antes, su nombre es Chris.

**Respuesta: Chris**

# 2. Cracking de hash y fuerza bruta

¿Terminaste de enumerar la máquina? Es hora de salir a la fuerza.

## a) Contraseña FTP

Suponemos que el nombre de usuario será "Chris", así que probamos a usar Hydra para hacer fuerza bruta usando el diccionario "Rockyou" sobre el servicio FTP. Usamos:

```
hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.88.160 ftp
```

Y obtenemos la contraseña, que es "crystal"

![](IMG/Pasted%20image%2020250412014409.png)

**Respuesta: crystal**

## b) Contraseña del archivo zip

Lo primero que haremos será conectarnos al FTP con las credenciales que tenemos. Usamos:

```
ftp 10.10.88.160 21
```

![](IMG/Pasted%20image%2020250412015213.png)

Una vez conectados con éxito, listamos el contenido y lo descargamos.

![](IMG/Pasted%20image%2020250412015349.png)

Abrimos primero el archivo de texto To_agentJ.txt

![](IMG/Pasted%20image%2020250412015515.png)

El texto nos dice que las fotos de aliens son falsas, que el agente R almacenó la foto real en el directorio del agente C y que la contraseña para logearse está almacenada en una de las fotos falsas. Así que suponemos que una de ellas tendrá algo escondido con Steghide.

Usaremos la herramienta binwalk, para analizar las imágenes y averiguar si ocultan algún archivo. La imagen cutie.png oculta un archivo .zip. Hemos usado:

```
binwalk cutie.png
```

![](IMG/Pasted%20image%2020250412015917.png)

lo extraemos, usando:

```
binwalk cutie.png --extract
```

![](IMG/Pasted%20image%2020250412020720.png)

Vemos que lo ha extraído, pero no ha podido descomprimirlo porque está encriptado. Para desencriptarlo, usaremos John the ripper, primero usaremos zip2john para conseguir el hash que intentaremos descifrar. Esto lo haremos dentro del directorio que se ha creado donde están los archivos extraídos.

```
zip2john 8702.zip
```

![](IMG/Pasted%20image%2020250412021456.png)

Este hash lo ponemos en un archivo, que es el que le pasaremos a john the ripper.

![](IMG/Pasted%20image%2020250412021625.png)

y obtenemos la contraseña, "alien"

**Respuesta: alien**

## c) contraseña de steg

Ahora, ya podemos descomprimir el archivo usando la contraseña obtenida.

```
7z x 8702.zip -palien
```

![](IMG/Pasted%20image%2020250412022144.png)

Y cuando accedemos al archivo de texto que hemos obtenido, encontramos lo siguiente:

![](IMG/Pasted%20image%2020250412022244.png)

Que suponemos que será la contraseña para desencriptar la otra imagen, cute-alien.jpg

la desencriptamos usando:

```
steghide --extract -sf cute-alien.jpg -p QXJlYTUx
```

Y parece ser que no es la contraseña.

![](IMG/Pasted%20image%2020250412022538.png)

La hemos puesto en Cyberchef, y resulta que estaba cifrada en Base64, la contraseña real es Area51.

![](IMG/Pasted%20image%2020250412022720.png)

Probamos de nuevo

```
steghide --extract -sf cute-alien.jpg -p Area51
```

![](IMG/Pasted%20image%2020250412022813.png)

Y ahora sí, ya hemos podido acceder al mensaje, el cual está dirigido a un agente llamado James, suponemos que será el agente J.

**Respuesta: Area51**

## d) ¿Quién es el otro agente (en nombre completo)?

Como hemos visto antes, se llama James.

**Respuesta: James**

## e) Contraseña SSH

Probamos a conectarnos por SSH, con las credenciales que tenemos, nombre de usuario james y contraseña hackerrules!

```
ssh james@10.10.88.160
```

![](IMG/Pasted%20image%2020250412023336.png)

**Respuesta: hackerrules!**

# 3. Capturar la flag del usuario

Ya sabes el procedimiento.

## a) ¿Cual es la flag de usuario?

listamos los archivos y abrimos el archivo con la flag de usuario.

![](IMG/Pasted%20image%2020250412023506.png)

**Respuesta: b03d975e8c92a7c04146cfa7a5a313c7**

## b) ¿Cómo se llama el incidente de la foto?

Para hacer esto, primero copiamos la imagen a nuestra máquina, usando:

```
sudo scp james@10.10.88.160:Alien_autospy.jpg ~/
```

![](IMG/Pasted%20image%2020250412024307.png)

Y tratamos de hacer una búsqueda en google con la imagen para que nos muestre información sobre ella.

![](IMG/Pasted%20image%2020250412024618.png)

Buscamos en páginas con información en inglés y descubrimos la respuesta.

**Respuesta: roswell alien autopsy**

# 4. Escalada de privilegios

¿Ya basta de cosas extraordinarias? Es hora de ser realistas.

## a) Número de CVE para la escalada (Formato: CVE-xxxx-xxxx)

Primero ejecutamos sudo -l para saber si tenemos algún privilegio como sudo.

![](IMG/Pasted%20image%2020250412024934.png)

Se supone que podemos ejecutar una consola como sudo, pero no podemos. así que busco por el comando en sí (ALL, !root) /bin/bash en internet, y encontramos lo siguiente:

![](IMG/Pasted%20image%2020250412025210.png)

siguiendo las instrucciones, podemos crear un script de python3 y al ejecutarlo, nos debe dar los permisos de sudo.

![](IMG/Pasted%20image%2020250412025402.png)

Así que creamos el archivo con ese contenido y lo ejecutamos.

![](IMG/Pasted%20image%2020250412025524.png)

Y, listo, ya tenemos acceso root.

![](IMG/Pasted%20image%2020250412025607.png)

**Respuesta: CVE-2019-14287**

## b) ¿Cual es la flag de root?

Abrimos el archivo root.txt

![](IMG/Pasted%20image%2020250412025851.png)

**Respuesta: b53a02f55b57d4439e3341834d70c062**

## c) (Bonus) ¿Quién es el Agente R?

Como vimos en el archivo root.txt, agente R es Deskel.

**Respuesta: deskel**