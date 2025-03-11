# 1) Objetivo

En esta máquina, tendremos que ir contestando a las preguntas que nos proponen, ademas de tener que conseguir acceso como usuario y como root, para encontrar las respectivas flags.

# 2) Reconocimiento

Lo primero que haremos, antes de comenzar a buscar la respuesta a las preguntas, es realizar un escaneo nmap, para conocer los puertos abiertos y los servicios que están corriendo.

```
nmap -sV -A -T4 10.10.223.190
```

![](IMG/Pasted%20image%2020250311015912.png)

Y vemos que, tenemos tres puertos abiertos, el 80, donde hay un servidor web corriendo joomla, que ademas tiene un archivo robots.txt, el 22, donde tenemos SSH y el 3306 donde está corriendo mariadb.

## a) Accede al servidor web, ¿quién robó el banco?

La respuesta a esta pregunta, la hemos encontrado directamente al acceder al servidor web, en la página principal nos dice, literalmente, que Spider-Man robó el banco.

![](IMG/Pasted%20image%2020250311020221.png)

**Respuesta: Spiderman**

# 3) Obtener acceso y escalada de privilegios

Antes de comenzar con esta parte, vamos a revisar el archivo robots.txt que sabemos que existe porque nos lo ha mostrado nmap.

![](IMG/Pasted%20image%2020250311020616.png)

Vemos muchos directorios que no deben ser listados, así que tenemos varias rutas de las que obtener algo de información.

## a) ¿Cuál es la versión de Joomla?

Accedemos a /administrator/ que suponemos que puede ser el panel de acceso a Joomla.

![](IMG/Pasted%20image%2020250311021505.png)

Y efectivamente, lo es, pero no nos indica que versión es, así que tendremos que probar otra cosa, ya que tampoco tenemos las credenciales y no hemos podido acceder con fuerza bruta ni usando zap.

Lo primero que intento es hacer una búsqueda sencilla en internet por si hubiera suerte y nos indicaran alguna forma fácil de conocer este dato. Tenemos suerte y nos indican que se puede consultar el archivo README.txt para conocer la versión.

![](IMG/Pasted%20image%2020250311021748.png)

Así que lo probamos, y podemos ver que la versión instalada es la 3.7

![](IMG/Pasted%20image%2020250311021858.png)

**Respuesta: 3.7.0**

## b) En lugar de utilizar SQLMap, ¿por qué no utilizar un script de Python?

¿Cuál es la contraseña descifrada de Jonah?









































































































































































