---
description: 'Target(s): 94.237.53.203:55294'
---

# Command Injections

Se le ha contratado para realizar una pentest para una empresa y, durante su realización, se encuentra con una interesante aplicación web de gestión de archivos. Dado que los gestores de archivos suelen ejecutar comandos del sistema, le interesa comprobar si existen vulnerabilidades de inyección de comandos.

Utilice las diversas técnicas presentadas en este módulo para detectar una vulnerabilidad de inyección de comandos y, a continuación, explótela, eludiendo cualquier filtro existente.

**Pregunta 1**: ¿Cuál es el contenido del archivo «/flag.txt»?

1. Accedemos a través del navegador al objetivo utilizando las credenciales guest:guest y vemos que es un File Manager.

<figure><img src="../../../../.gitbook/assets/image (406).png" alt=""><figcaption></figcaption></figure>

2. Identificamos distintas funcionalidades (buscar, búsqueda avanzada, descargar, copiar, etc) que pueden servirnos de vector de ataque:

<figure><img src="../../../../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

3. Por ejemplo, haciendo click en Copiar, nos redirige a otra página, en la que vemos en la URL parámetros que podemos intentar explotar, pero haciendo click en la función Move vemos que aparecen más:

<figure><img src="../../../../.gitbook/assets/image (408).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (409).png" alt=""><figcaption></figcaption></figure>

4. Activamos FoxyProxy para Burp, iniciamos Burp, recargamos la página para capturar la solicitud y la enviamos al Repeater:

<figure><img src="../../../../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>

5. En el Repeater se prueba a ver que operadores están blacklisteados:

<figure><img src="../../../../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>

6. Después de diferentes pruebas, encontramos que el operador %26%26 no está blacklisteado por la respuesta del server:

<figure><img src="../../../../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

7. Se intenta inyectar el comando pero lo bloquea:

<figure><img src="../../../../.gitbook/assets/image (413).png" alt=""><figcaption></figcaption></figure>

8. Probaremos encriptandolo en base 64:

<figure><img src="../../../../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure>

9. Luego haciendo uso de el inyectar en shell de bash, el comando en base64 y sustituyendo el espacio por una tabulación (%09):

<figure><img src="../../../../.gitbook/assets/image (415).png" alt=""><figcaption></figcaption></figure>

10. Al darnos el error de “missing destination” nos indica que el parametro to= espera una parámetro, así que le añadiremos el payload en él y el from lo dejaremos como estaba:

<figure><img src="../../../../.gitbook/assets/image (416).png" alt=""><figcaption></figcaption></figure>

11. Conseguimos el output del comando correctamente. Ahora vamos a buscar la flag que se pide en el ejercicio.
12. Primero encodeamos en base64 el comando ls -la:

<figure><img src="../../../../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

13. Lo metemos en el parámetro de to= indicando que es para bash, está en base64, sustituimos espacios por tabulaciones:

<figure><img src="../../../../.gitbook/assets/image (418).png" alt=""><figcaption></figcaption></figure>

14. Nos da un error en el nombre del archivo del from, no lo encuentra, lo cambiamos y volvemos a ejecutar:

<figure><img src="../../../../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

* Conseguimos el output del comando ls -la.

15. Lo siguiente será encriptar en base64 el comando para leer el archivo flag.txt que se encuentra en el directorio raíz /:

<figure><img src="../../../../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

16. Se lo pasamos como parámetro de to= indicando que es para bash, está en base64, sustituimos espacios por tabulaciones, cambiamos también el nombre del archivo para que no nos dé el error de antes:

<figure><img src="../../../../.gitbook/assets/image (421).png" alt=""><figcaption></figcaption></figure>
