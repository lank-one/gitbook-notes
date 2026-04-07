---
description: 'Target(s): 94.237.59.174:33761'
---

# Server-side Attacks

## Escenario

Se le ha encomendado la tarea de realizar una evaluación de seguridad de la aplicación web de un cliente. Aplique lo que ha aprendido en este módulo para obtener la bandera.

**Pregunta 1**: Obtén la flag

1. Desviamos el tráfico a través de Burp, recargamos la página y vemos una solicitud POST que envía código a una API:

<figure><img src="../../../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

2. Enviamos la request al Repeater, cambiando el valor de api a la IP de localhost para ver si devuelve el código fuente de la página:

<figure><img src="../../../../.gitbook/assets/image (381).png" alt=""><figcaption></figcaption></figure>

3. Esto confirma que el SSRF no es blind, el siguiente paso es la enumeración de la API para encontrar puertos o directorios disponibles. Lanzamos una request a un puerto abierto (80) y a uno cerrado (81) para ver la respuestas que da el servidor y luego utilizarlas para filtrar:

<figure><img src="../../../../.gitbook/assets/image (382).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (383).png" alt=""><figcaption></figcaption></figure>

4. Usaremos fuff para fuzzear puertos y directorios, filtrando por el mensaje de cuando no se encuentra el puerto/directorio. Utilizaremos las wordlist disponibles en la máquina:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/raft-small-words.txt -u http://94.237.59.174:33761/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "api=http://truckapi.htb/FUZZ.php?id%3DFusionExpress01" -fr "The requested URL was not found on this server."
```

* Conseguimos algunos hits pero nada destacable, por lo que probaremos con otras técnicas.

5. Modificamos la request con el payload ${7\*7} para probar si es vulnerable a SSTI:

<figure><img src="../../../../.gitbook/assets/image (385).png" alt=""><figcaption></figcaption></figure>

6. Vemos que se inyecta en el segundo objeto de JSON, seguimos el esquema que vimos en las inyecciones SSTI para lograr el resultado de la operación.

<figure><img src="../../../../.gitbook/assets/image (386).png" alt=""><figcaption></figcaption></figure>

7. Probaremos la ejecución de código:

<figure><img src="../../../../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

8. Vemos que se ejecuta correctamente el comando, por lo que probaremos leer el contenido de flag:

<figure><img src="../../../../.gitbook/assets/image (388).png" alt=""><figcaption></figcaption></figure>

9. Hemos encodeado el espacio en URL encode (%20) pero da un error. Probaremos otra forma de encodearla, por ejemplo en hexadecimal (\x20):

<figure><img src="../../../../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>
