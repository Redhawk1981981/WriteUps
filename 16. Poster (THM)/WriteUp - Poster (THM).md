# 1. Flag

_¿Qué es RDBMS?  

Dependiendo del modelo relacional EF Codd, un RDBMS permite a los usuarios construir, actualizar, administrar e interactuar con una base de datos relacional, que almacena datos como una tabla.

Hoy en día, varias empresas utilizan bases de datos relacionales en lugar de archivos planos o bases de datos jerárquicas para almacenar datos empresariales. Esto se debe a que una base de datos relacional puede gestionar una amplia gama de formatos de datos y procesar consultas de forma eficiente. Además, organiza los datos en tablas que pueden vincularse internamente basándose en datos comunes. Esto permite al usuario recuperar fácilmente una o más tablas con una sola consulta. Por otro lado, un archivo plano almacena los datos en una única estructura de tabla, lo que lo hace menos eficiente y consume más espacio y memoria.

La mayoría de los RDBMS disponibles comercialmente utilizan lenguaje de consulta estructurado ( SQL ) para acceder a la base de datos. Las estructuras RDBMS se utilizan habitualmente para realizar operaciones CRUD (crear, leer, actualizar y eliminar), fundamentales para la gestión consistente de datos.

¿Serás capaz de completar el desafío?

## a) ¿Cuál es el RDBMS instalado en el servidor?

Lo primero que haremos, como siempre, será realizar un escaneo Nmap básico para conocer los puertos abiertos y sus servicios. Usamos:

```
nmap -sV -T5 10.10.238.64
```

Vemos que hay 3 puertos abiertos, el 22, el 80 y el 5432

![](IMG/Pasted%20image%2020250413000125.png)

Vemos que el RBDMS que se encuentra en el servidor es postgresql.

**Respuesta: postgresql**

## b) ¿En qué puerto se ejecuta el RDBMS?

Como vimos en el escaneo de nmap, está en el puerto 5432

**Respuesta: 5432**

## c) Metasploit contiene una variedad de módulos que pueden usarse para enumerar en múltiples RDBMS, lo que facilita la recopilación de información valiosa.

Se ejecuta usando msfconsole

**Respuesta: No aplica.**

## d) Tras iniciar Metasploit, busque un módulo auxiliar asociado que permita enumerar las credenciales de usuario. ¿Cuál es la ruta completa de los módulos (empezando por "auxiliar")?

Lo ejecutamos con msfconsole

![](IMG/Pasted%20image%2020250413000447.png)

Y buscamos payloads para postgresql, usando search postgresql

![](IMG/Pasted%20image%2020250413000643.png)

Como estamos buscando uno que nos permita enumerar las credenciales de usuario, usaremos, postgres_login

![](IMG/Pasted%20image%2020250413000800.png)

**Respuesta: auxiliary/scanner/postgres/postgres_login**

## e) ¿Cuales son las credenciales que encontraste? ejemplo: usuario:contraseña

usamos el exploit.

![](IMG/Pasted%20image%2020250413000935.png)

Configuramos las opciones

![](IMG/Pasted%20image%2020250413001043.png)

En nuestro caso, solo debemos configurar la Ip de la máquina a atacar.

![](IMG/Pasted%20image%2020250413001153.png)

Una vez configurado, lo ejecutamos con "run", y veremos que hemos encontrado un usuario y una contraseña.

![](IMG/Pasted%20image%2020250413001255.png)

**Respuesta: postgres:password**

## f) ¿Cuál es la ruta completa del módulo que le permite ejecutar comandos con las credenciales de usuario adecuadas (comenzando con auxiliar)?

Volvemos a hacer la búsqueda de exploits para postgresql como hicimos antes, y vemos que se trata del módulo postgres_sql

![](IMG/Pasted%20image%2020250413001625.png)

**Respuesta: auxiliary/admin/postgres/postgres_sql**

## g) Según los resultados del punto n.° 6, ¿cuál es la versión de rdbms instalada en el servidor?

Ejecutamos el módulo posgres_sql y listamos las opciones.

![](IMG/Pasted%20image%2020250413002015.png)

Después de configurar la IP de la máquina atacada, y el password del usuario que habíamos averiguado antes, tras ejecutarlo, vemos la versión.

![](IMG/Pasted%20image%2020250413010509.png)

**Respuesta: 9.5.21**

## h ¿Cuál es la ruta completa del módulo que permite volcar los hashes del usuario (comenzando con el auxiliar)?

De nuevo hacemos la búsqueda de módulos para postgresql.

![](IMG/Pasted%20image%2020250413005915.png)

Y vemos el módulo postgres_hashdump, al igual que con los anteriores, lo configuramos y lo lanzamos.

![](IMG/Pasted%20image%2020250413010050.png)

**Respuesta: auxiliary/scanner/postgres/postgres_hashdump**

## i) ¿Cuántos hashes de usuario vuelca el módulo?

Siguiendo con el módulo anterior, al ejecutarlo, obtenemos los hash de todas las contraseñas con su nombre de usuario correspondiente.

![](IMG/Pasted%20image%2020250413010134.png)

**Respuesta: 6**

## j) ¿Cuál es la ruta completa del módulo (que comienza con auxiliar) que permite a un usuario autenticado ver los archivos que elija en el servidor?

De nuevo, búsqueda de los modulos.

![](IMG/Pasted%20image%2020250413011130.png)

Y en este caso es postgre_readfile. Al igual que venimos haciendo, lo configuramos y ejecutamos.

![](IMG/Pasted%20image%2020250413011300.png)

![](IMG/Pasted%20image%2020250413011321.png)

![](IMG/Pasted%20image%2020250413011339.png)

En este caso, como por defecto viene la ruta /etc/passwd es el archivo que hemos recibido, podríamos poner la ruta que queramos en las opciones del módulo.

**Respuesta: auxiliary/admin/postgres/postgres_readfile**

## k) ¿Cuál es la ruta completa del módulo que permite la ejecución de comandos arbitrarios con las credenciales de usuario adecuadas (comenzando con el exploit)?

Y otra búsqueda más.

![](IMG/Pasted%20image%2020250413011711.png)

Es el módulo postgres_copy_from_program_cmd_exec, lo configuramos y ejecutamos.

![](IMG/Pasted%20image%2020250413012001.png)

![](IMG/Pasted%20image%2020250413012017.png)

Y obtenemos una shell.

![](IMG/Pasted%20image%2020250413012143.png)

**Respuesta: exploit/multi/postgres/postgres_copy_from_program_cmd_exec**

## l) Comprometer la máquina y localizar user.txt

Aprovechando el exploit de la pregunta anterior, creamos un listener para recibir la shell.

```
nc -lnvp 4444
```

![](IMG/Pasted%20image%2020250413012405.png)

Hemos conseguido conectarnos, ahora buscamos el archivo user.txt y lo abrimos.

![](IMG/Pasted%20image%2020250413013421.png)

He encontrado el archivo user.txt dentro del home del usuario alison, pero no consigo abrirlo, supongo que solo podrá abrirlo el usuario propietario que es alison, sin embargo en el home del usuario dark, he encontrado un archivo llamado credentials.txt que contiene un usuario y una contraseña (, usare estas credenciales para tratar de conectar por SSH.
He encontrado el archivo user.txt dentro del home del usuario alison, pero no consigo abrirlo, supongo que solo podrá abrirlo el usuario propietario que es alison, sin embargo en el home del usuario dark, he encontrado un archivo llamado credentials.txt que contiene un usuario y una contraseña (dark:qwerty1234#!hackme), usare estas credenciales para tratar de conectar por SSH.

Una vez conectados por SSH con el usuario dark, hemos encontrado en el directorio donde se encuentran los archivos del servidor web, un archivo de configuración que en su interior guarda las credenciales del usuario alison (alison:p4ssw0rdS3cur3!#).

![](IMG/Pasted%20image%2020250413013935.png)

Tras probar si el usuario "dark" puede ejecutar algo como sudo, con sudo -l, descubro que no.

![](IMG/Pasted%20image%2020250413014158.png)

Así que pruebo a conectarme por SSH con las credenciales encontradas del usuario "alison" por si hubiera más suerte.

Bien, he podido conectarme sin problemas, y he podido ya por fin leer el archivo user.txt.

![](IMG/Pasted%20image%2020250413014357.png)

**Respuesta: THM{postgresql_fa1l_conf1gurat1on}**

## m) Escalar privilegios y obtener root.txt

Ahora, nos queda escalar privilegios y encontrar (que estará en la misma ruta de siempre /root/root.txt) y abrir el archivo root.txt. Comprobamos con sudo -l que puede ejecutar el usuario alison como sudo.

Y, para sorpresa nuestra, puede ejecutar una consola bash como sudo.

![](IMG/Pasted%20image%2020250413014651.png)

Así que solo tenemos que abrir la consola bash con sudo y abrir el archivo root.txt

![](IMG/Pasted%20image%2020250413014827.png)

**Respuesta: THM{c0ngrats_for_read_the_f1le_w1th_credent1als}**