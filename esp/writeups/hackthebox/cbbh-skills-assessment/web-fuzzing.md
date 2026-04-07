---
description: 'Target(s): 94.237.51.21:42454'
---

# Web Fuzzing

Para completar esta skills assessment, deberás aplicar la multitud de herramientas y técnicas presentadas a lo largo de este módulo. Todo el fuzzing se puede completar utilizando la wordlists de SecLists common.txt, que se encuentra en /usr/share/seclists/Discovery/Web-Content en Pwnbox, o a través del GitHub de SecLists.

Pregunta 1: Después de completar todos los pasos del assessment, se mostrará una página que contiene una flag en el formato HTB{...}. ¿Cuál es esa flag?

Empezaremos con un fuzzeo de directorios:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://94.237.51.21:42454/FUZZ
```

<figure><img src="../../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Directorios encontrados:

* admin
* server-status

<figure><img src="../../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Haremos un fuzzing recursivo en admin:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -v -u http://94.237.51.21:42454/admin/FUZZ -e .html -recursion 
```

<figure><img src="../../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

Ya hemos comprobado que en admin/index.php nos salta un mensaje de Access Denied.

Utilizamos ffuf para buscar archivos o carpetas que no hayamos probado aún, con extensiones comunes (php, html, txt, json, etc.):

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://94.237.51.21:42454/admin/FUZZ -e .php,.html,.txt,.json -recursion -ic -v
```

<figure><img src="../../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

Encontramos el archivo panel.php

<figure><img src="../../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Instalaremos wenum:

```bash
pipx install git+https://github.com/WebFuzzForge/wenum
pipx runpip wenum install setuptools
```

Hacemos un curl para interactuar con el endpoint:

```bash
curl http://94.237.51.21:42454/admin/panel.php
```

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Nos indica que hay un parámetro accessID. Vamos a fuzzear el valor correcto para el parámetro:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -u http://94.237.51.21:42454/admin/panel.php?accessID=FUZZ -fs 58
```

<figure><img src="../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Obtenemos el parámetro getaccess, lo utilizamos con curl:

```bash
curl http://94.237.51.21:42454/admin/panel.php?accessID=getaccess
```

<figure><img src="../../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

Nos indica que hay que hacer fuzzing sobre el vhost fuzzing\_fun.htb. Lo añadimos a /etc/hosts:

```bash
echo "94.237.51.21 fuzzing_fun.htb" | sudo tee -a /etc/hosts
```

<figure><img src="../../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

Hacemos curl:

```bash
curl fuzzing_fun.htn:42454
```

<figure><img src="../../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Nos indica que nuestro punto de inicio es el directorio godeep.

Haremos fuzzing sobre el vhost fuzzing\_fun para no dejarnos nada por encontrar:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -u http://fuzzing_fun.htb:42454/ -H ‘Host: FUZZ.fuzzing_fun.htb’ -fc 403
```

<figure><img src="../../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

Añadimos este nuevo vhost a /etc/hosts:

<figure><img src="../../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

**Target**(s): 94.237.123.178:51795

Hacemos fuzzing sobre el directorio godeep:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -u http://hidden.fuzzing_fun.htb:51795/godeep/FUZZ
```

<figure><img src="../../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Hacemos fuzzing sobre /godeep con -recursion para buscar más directorios:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -u http://hidden.fuzzing_fun.htb:51795/godeep/FUZZ -recursion
```

<figure><img src="../../../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

Anidados uno dentro del otro tenemos:

* /stoneedge/bbclone/typo3/

Mostramos el contenido de index.php dentro de typo3:

<figure><img src="../../../../.gitbook/assets/image (617).png" alt=""><figcaption></figcaption></figure>
