# 1) Descripción

Esta tarea implica que prestes atención a los detalles y encuentres las "llaves del castillo".

Esta sala está diseñada para principiantes, ¡sin embargo, todos son bienvenidos a probarla!

En esta sala, no necesitas forzar ninguna página de inicio de sesión. Solo tu navegador preferido y Escritorio Remoto.

# 2) Reconocimiento

## a) Ejecutemos nmap y verifiquemos qué puertos están abiertos.

Comenzaremos haciendo un Nmap a la máquina objetivo para conocer los puertos abiertos si los hubiera, y los servicios que corren en ellos.

> [!NOTE]
> He tratado de hacer Nmap normal, pero la maquina está bloqueando ICMP, es decir no acepta ping como otras máquinas anteriores, así que he usado -Pn

```
nmap -Pn -A -sV -T4 10.10.158.76
```

![](IMG/Pasted%20image%2020250318022037.png)

**Respuesta: No aplica.**

## b) ¿Qué puerto es para el servidor web?

![](IMG/Pasted%20image%2020250318022222.png)

**Respuesta: 80**

## c) ¿Qué puerto es para el servicio de escritorio remoto?

![](IMG/Pasted%20image%2020250318022240.png)

**Respuesta: 3389**

## d) ¿Cuál es una posible contraseña en una de las páginas que buscan los rastreadores web?

La página que buscan los rastreadores web es robots.txt, y si accedemos, encontramos la posible contraseña.

![](IMG/Pasted%20image%2020250318022424.png)

**Respuesta: UmbracoIsTheBest!**

## e) ¿Qué CMS está utilizando el sitio web?

Teniendo en cuenta que Umbraco es un CMS, pues eso.

![](IMG/Pasted%20image%2020250318022716.png)

**Respuesta: Umbraco**

## f) ¿Cuál es el dominio del sitio web?

Si accedemos a la web, podemos ver el dominio.

![](IMG/Pasted%20image%2020250318022829.png)

**Respuesta: Anthem.com**

## g) ¿Cómo se llama el administrador?

En el articulo "A cheers to our IT departament" encontramos un poema.

![](IMG/Pasted%20image%2020250318023837.png)

Si buscamos información sobre el mismo en Google, encontraremos el nombre de su autor en Wikipedia.

![](IMG/Pasted%20image%2020250318024017.png)

**Respuesta: Solomon Grundy**

## h) ¿Podemos encontrar la dirección de correo electrónico del administrador?

La dirección de correo que aparece en los artículos es JD@anthem.com ya que el autor es Jane Doe.

![](IMG/Pasted%20image%2020250318024357.png)

Pero ya sabemos su verdadero nombre, que es Solomon Grundy, por lo que lo lógico es que su correo sea SG@anthem.com

**Respuesta: SG@anthem.com**

# 2) Encontrar las banderas

Para encontrar las banderas hemos buscado por la página web y por el código de la misma.

## a) ¿Cual es la primera bandera?

Si entramos en el primer post de la página y miramos en su código fuente, encontramos la primera bandera.

![](IMG/Pasted%20image%2020250318025333.png)

**Respuesta: THM{L0L_WH0_US3S_M3T4}**

## b) ¿Cual es la segunda bandera?

En el código fuente de la página "Categories" encontramos la segunda bandera.

![](IMG/Pasted%20image%2020250318024919.png)

**Respuesta: THM{G!T_G00D}**

## c) ¿Cual es la tercera bandera?

Si entramos en el primer comentario llamado "We are hiring" y clicamos sobre el nombre del autor, encontramos la tercera bandera.

![](IMG/Pasted%20image%2020250318023332.png)

![](IMG/Pasted%20image%2020250318023351.png)

**Respuesta: THM{L0L_WH0_D15}**

## d) ¿Cual es la cuarta bandera?

En el código fuente del segundo post, encontramos la cuarta bandera.

![](IMG/Pasted%20image%2020250318025457.png)

**Respuesta: THM{AN0TH3R_M3TA}**

# 3) Etapa final

## a) Averigüemos el nombre de usuario y la contraseña para iniciar sesión en el cuadro. (El cuadro no está en un dominio)

Tenemos un posible nombre de usuario (el mail del administrador) y una posible contraseña (la que vimos en el robots.txt)

**Respuesta: No aplica.**

## b) Obtenga acceso inicial a la máquina, ¿cuál es el contenido de user.txt?

Como sabemos que la máquina tiene un acceso a escritorio remoto activo, vamos a usar remmina, como hemos hecho en máquinas anteriores para intentar iniciar sesión con las posibles credenciales que tenemos.

```
usuario: SG (con el correo completo no me funcionó)
password: UmbracoIsTheBest!
```

![](IMG/Pasted%20image%2020250318030800.png)

y conseguimos conectarnos al escritorio remoto. En el escritorio está el archivo user.txt, que si lo abrimos, tenemos la flag.

![](IMG/Pasted%20image%2020250318031047.png)

**Respuesta: THM{N00T_NO0T}**

## c) ¿Podemos identificar la contraseña de administrador?

En la pista que nos da la web de Tryhackme nos dice que está escondida, así que podemos buscar en directorios o archivos ocultos del sistema, usando el explorador de archivos y configurándolo para que muestre los archivos y carpetas ocultas.

![](IMG/Pasted%20image%2020250318031431.png)

Encontramos en c: una carpeta oculta llamada backup.

![](IMG/Pasted%20image%2020250318031732.png)

Dentro, hay un archivo llamado restore.

![](IMG/Pasted%20image%2020250318031819.png)

El cual, si lo intentamos abrir, nos dice que no tenemos permisos para ello.

![](IMG/Pasted%20image%2020250318031857.png)

Así que intentaremos cambiar los permisos del fichero para ver si así podemos abrirlo.

![](IMG/Pasted%20image%2020250318032005.png)

Y ahora si, podemos abrirlo y ver la flag, que es la contraseña del administrador.

![](IMG/Pasted%20image%2020250318032043.png)

**Respuesta: ChangeMeBaby1MoreTime**

## d) Escala tus privilegios a root, ¿cuál es el contenido de root.txt?

Como tenemos la contraseña del administrador, podemos acceder a sus archivos, por lo que solo debemos acceder a su carpeta.

![](IMG/Pasted%20image%2020250318032330.png)

![](IMG/Pasted%20image%2020250318032348.png)

Nos pedirá la contraseña de administrador.

![](IMG/Pasted%20image%2020250318032417.png)

Introducimos la contraseña, que es ChangeMeBaby1MoreTime como vimos en el apartado anterior, y del mismo modo que el usuario normal, el archivo root.txt está en el escritorio del administrador.

![](IMG/Pasted%20image%2020250318032635.png)

![](IMG/Pasted%20image%2020250318032654.png)

**Respuesta: THM{Y0U_4R3_1337}**