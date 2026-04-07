---
description: 'Target(s): 94.237.56.224:34018'
---

# Using Web Proxies

En el enunciado nos dicen que estamos realizando pruebas de pentesting internas sobre las aplicaciones web de una empresa. Estaremos ante situaciones en las que Burp/ZAP pueden resultar útiles. Hay que determinar las funciones que serían más útiles para cada caso.&#x20;

**Pregunta 1**: La página /lucky.php tiene un botón que parece estar desactivado. Intenta activar el botón y, a continuación, haz clic en él para obtener la flag.

1. Accedemos al navegador integrado de Burp, accedemos a la URL IP/lucky.php, hacemos forward y con F12 modificamos el valor de disabled a enabled.

<figure><img src="../../../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

2. Hacemos Forward en Proxy de Burp hasta que nos salga la petición que contiene getflag=true y la mandamos al Repeater.

<figure><img src="../../../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

3. En el Repeater, la enviamos varias veces, hasta que nos aparezca la flag en la respuesta.

<figure><img src="../../../../.gitbook/assets/image (451).png" alt=""><figcaption></figcaption></figure>

**Pregunta 2**: La página /admin.php utiliza una cookie que ha sido codificada varias veces. Intenta descodificar la cookie hasta obtener un valor de 31 caracteres. Envía el valor como respuesta.

1. Accedemos a la URL IP/admin.php desde el navegador integrado de Burp y capturamos la petición con la cookie.

<figure><img src="../../../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

2. Copiamos la cookie y la decodificamos en la pestaña Decode de Burp.

<figure><img src="../../../../.gitbook/assets/image (453).png" alt=""><figcaption><p>Primero la decodificamos de ASCII hex y después de Base64</p></figcaption></figure>

**Pregunta 3**: Una vez que descodifique la cookie, observará que solo tiene 31 carácteres, lo que parece ser un hash md5 al que le falta el último carácter. Por lo tanto, intente fuzzear el último carácter de la cookie md5 descodificada con todos los caracteres alfanuméricos, mientras codifica cada solicitud con los métodos de codificación que ha identificado anteriormente. (Puede utilizar la lista de palabras «alphanum-case.txt» de Seclist para el payload).

1. Capturamos la petición con la cookie original, la enviamos a Intruder y ponemos el indicador de payload.

<figure><img src="../../../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

2. Cargamos la wordlist /opt/useful/seclists/Fuzzing/alphanum-case.txt en el Payload Configuration.

<figure><img src="../../../../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

3. Configuramos el procesamiento del payload de la siguiente manera:

<figure><img src="../../../../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>

4. Lanzamos el ataque y esperamos a que finalice, debemos ordenar por Response Received y el que sea diferente es el que contendrá la flag.

<figure><img src="../../../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

5. Hacemos doble click en la petición y vamos a la pestaña Response para buscar la flag.

<figure><img src="../../../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

**Pregunta 4**: Estás utilizando la herramienta «auxiliary/scanner/http/coldfusion\_locale\_traversal» dentro de Metasploit, pero no te funciona correctamente. Decides capturar la solicitud enviada por Metasploit para poder verificarla manualmente y repetirla. Una vez capturada la solicitud, ¿cómo se llama el directorio «XXXXX» en «/XXXXX/administrator/..»?

1. Iniciamos Metasploit y usamos el módulo que se indica en el enunciado.

<figure><img src="../../../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

2. Configuramos las opciones del módulo, configurando como proxy a Burp.

<figure><img src="../../../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

3. Ejecutamos el módulo e interceptamos la petición en Burp.

<figure><img src="../../../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>
