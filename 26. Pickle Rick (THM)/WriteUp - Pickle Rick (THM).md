# 1) Descripción

Este desafío con temática de Rick y Morty requiere que explotes un servidor web y encuentres tres ingredientes para ayudar a Rick a hacer su poción y transformarse nuevamente en humano a partir de un pepinillo.

# 2) Preguntas

## a) ¿Cuál es el primer ingrediente que necesita Rick?

Lo primero que voy a probar es a comprobar si existe "robots.txt".

![](IMG/Pasted%20image%2020250318003103.png)

Encontramos un dato que de momento no nos sirve, pero lo anotaremos para mas adelante.

Echaremos un ojo a la página en busca de algo interesante, como no vemos nada en la página, miramos el código fuente, y encontramos el nombre de usuario.

![](IMG/Pasted%20image%2020250318003400.png)

Quizás lo que encontramos antes en robots.txt es la contraseña, si es así ya tendríamos unas credenciales, ahora solo tenemos que encontrar donde introducirlas.

Para encontrar mas información, voy a utilizar dirbuster, para listar los directorios de la web.

Encontramos algunas rutas interesantes como login.php y portal.php

![](IMG/Pasted%20image%2020250318005714.png)

Probaré a acceder a login.php para ver si realmente es una página de login y funcionan las credenciales que tenemos.

![](IMG/Pasted%20image%2020250318005834.png)

Exacto, es una página de login, trataremos de acceder con las credenciales que tenemos. 

```
usuario: R1ckRul3s
password: Wubbalubbadubdub
```

Hemos accedido y nos ha redirigido a la otra página que vimos en dirbuster, portal.php. Parece ser un formulario donde podemos escribir comandos, probaré a ejecutar ls, para ver que nos muestra.

![](IMG/Pasted%20image%2020250318010225.png)

Efectivamente nos ha mostrado un listado del contenido del directorio, en el que encontramos un archivo txt que parece contener el ingrediente que buscamos, tratamos de abrirlo usando el comando cat.

![](IMG/Pasted%20image%2020250318010403.png)

Nos muestra que el comando está deshabilitado, así que deberemos buscar otra manera de abrirlo. Bueno, como ye sabemos que el archivo en cuestión está en el directorio en el que nos encontramos, trataré de abrirlo con el navegador.

![](IMG/Pasted%20image%2020250318011220.png)

Hemos conseguido acceder al contenido del archivo.

**Respuesta: mr. meeseek hair**

> [!NOTE]
> He realizado otras pruebas con comandos de linux que me pudieran mostrar el archivo como nano y tail y están prohibidos también, pero el comando less si funciona y muestra el contenido de los archivos sin tener que acceder a ellos por el navegador.

## b) ¿Cuál es el segundo ingrediente de la poción de Rick?

De la misma forma, vamos a abrir el otro archivo que se llama clue.txt (pista) a ver que nos dice.

![](IMG/Pasted%20image%2020250318011530.png)

Nos dice que miremos en el sistema de ficheros, que está el otro ingrediente. Lo primero que haremos será listar todos los usuarios que existen, o al menos, los que tienen un directorio home, volvemos a la página con el formulario de comandos, y ejecutamos:

```
ls -l /home
```

![](IMG/Pasted%20image%2020250318011902.png)

Vemos que hay un usuario que se llama rick, intentaremos ahora acceder a sus archivos, usamos:

```
ls -la /home/rick
```

![](IMG/Pasted%20image%2020250318012019.png)

Encontramos un archivo llamado "segundo ingrediente", el problema es que trato de acceder a la ruta desde el navegador y no puedo.

![](IMG/Pasted%20image%2020250318012408.png)

Trataré de abrirlo usando el comando less, que como comenté antes si está permitido y nos muestra el contenido de los archivos.

```
less /home/rick/second\ ingredients
```

![](IMG/Pasted%20image%2020250318013516.png)

Y encontramos el segundo ingrediente.

**Respuesta: 1 jerry tear**

## c) ¿Cuál es el último ingrediente?

El último ingrediente, supongo que será como la root flag, así que voy a probar a buscarlo en el directorio de root, evidentemente como es root, usaremos sudo.

```
sudo ls /root
```

![](IMG/Pasted%20image%2020250318013753.png)

y efectivamente encontramos un archivo txt que debe contener el ultimo ingrediente, usamos less de nuevo, pero en este caso con sudo.

```
sudo less /root/3rd.txt
```

![](IMG/Pasted%20image%2020250318013858.png)

Y hemos encontrado el último ingrediente.

**Respuesta: fleeb juice**