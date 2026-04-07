---
description: "Target(s):\_94.237.63.11:30816"
---

# Attacking Web Applications with ffuf - Web Fuzzing

Nos proporcionan la dirección IP de una academia online, pero no tenemos más información sobre su sitio web. Como primer paso para realizar una pentest, hay que localizar todas las páginas y dominios vinculados a la IP para realizar una enumeración correctamente.

Se deben realizar pruebas de fuzzing en las páginas que se identifiquen para ver si alguna de ellas tiene parámetros con los que se pueda interactuar. Si encontramos parámetros activos, hay que comprobar si se puede recuperar algún dato de ellos.

**Pregunta 1:** Realiza un análisis de fuzzing de subdominios/vhosts en «\*.academy.htb» para la IP del objetivo. ¿Cuáles son todos los subdominios que puedes identificar? (Escribe solo el nombre del subdominio).

1. Primero añadiremos al archivo /etc/hosts la IP y dominio:

```
sudo sh -c 'echo "94.237.63.11 academy.htb" >> /etc/hosts'
```

2. Empezaremos haciendo fuzzing de subdominios, el siguiente comando devuelve los subdominios válidos (los que responden):

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.academy.htb/
```

3. A continuación, haremos fuzzing de vhosts, cuando varias aplicaciones están en el mismo IP/puerto, el servidor HTTP puede distinguirlas según el valor de la cabecera _Host_:

```
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:30816/ -H 'Host: FUZZ.academy.htb'
```

4. Por último, haremos fuzzing de vhosts con filtro por tamaño de respuesta, muchas respuestas innecesarias pueden tener el mismo tamaño (por ser errores o páginas de default). Filtraremos por tamaño para distinguir respuestas “diferentes”, que suelen indicar un subdominio/vhost válido y funcional.:

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:30816/ -H 'Host: FUZZ.academy.htb' -fs 985
```

**Pregunta 2:** Antes de ejecutar el análisis de fuzzing de la página, primero debe ejecutar un análisis de fuzzing de extensiones. ¿Cuáles son las diferentes extensiones aceptadas por los dominios?&#x20;

1. Añadiremos los subdominios encontrados en la pregunta anterior al archivo /etc/hosts:

```
sudo nano /etc/hosts
```

<figure><img src="../../../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

2. Hacemos fuzzing sobre los subdominios, probando extensiones sobre la página index y filtrando las que tenga una respuesta de 284 bytes:

```
ffuf -w /usr/share/seclists/SecLists-master/Discovery/Web-Content/web-extensions-big.txt:FUZZ -u http://SUBDOMINIO.academy.htb:30816/indexFUZZ -fs 284
```

<figure><img src="../../../../.gitbook/assets/image (478).png" alt=""><figcaption></figcaption></figure>

**Pregunta 3**: Una de las páginas que identificarás debería decir 'You don't have access!'. ¿Cuál es la URL completa de la página?

1. Probaremos haciendo fuzzing sobre los subdominios e indicando las extensiones encontradas en la pregunta anterior:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt:FUZZ -u http://SUBDOMINIO.academy.htb:30816/FUZZ -recursion -recursion-depth 1 -e .EXTENSION1,.EXTENSION2,.EXTENSION3 -v -fs 284
```

2. Dependiendo de las respuestas obtenidas, inspeccionaremos detalladamente un archivo que nos ha dado una respuesta interesante:

```
curl http://SUBDOMINIO.academy.htb:30816/courses/ARCHIVO.EXTENSION
```

**Pregunta 3**: En la página de la pregunta anterior, deberías poder encontrar varios parámetros que son aceptados por la página. ¿Cuáles son?&#x20;

1. Con el siguiente comando se realizará un fuzzing avanzado para encontrar nombres de parámetros aceptados en un formulario POST al recurso dentro del subdominio:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://SUBDOMINIO.academy.htb:49405/courses/ARCHIVO.EXTENSION -c -ic -t 200 -H 'Content-Type: application/x-www-form-urlencoded' -d 'FUZZ=key' -X POST -fs 774
```

**Pregunta 4:** Prueba a realizar fuzzing con los parámetros que has identificado para obtener valores válidos. Uno de ellos debería devolver una flag. ¿Cuál es el contenido de la flag?

1. Haremos fuzzing sobre uno de los parámetros descubiertos en la pregunta anterior:

```
ffuf -w /opt/useful/seclists/Usernames/Names/names.txt:FUZZ -u http://SUBDOMINIO.academy.htb:49405/courses/ARCHIVO.EXTENSION -c -ic -t 200 -H 'Content-Type: application/x-www-form-urlencoded' -d 'PARAMETRO=FUZZ' -X POST -fs 781
```

2. Realizamos una petición POST directa con el parámetro y el valor que se identificó como válido o que dio un resultado interesante durante el fuzzing de la pregunta anterior, confirmando de forma manual la respuesta del servidor para ese parámetro específico:

```
curl http://SUBDOMINIO.academy.htb:49405/courses/ARCHIVO.EXTENSION -X POST -H 'Content-Type: application/x-www-form-urlencoded' -d 'PARAMETRO=VALOR'
```

<figure><img src="../../../../.gitbook/assets/image (447).png" alt=""><figcaption></figcaption></figure>
