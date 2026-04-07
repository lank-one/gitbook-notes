---
description: >-
  Target(s): 83.136.251.13:51868 | vHosts needed for these questions:
  inlanefreight.htb
---

# Information Gathering - Web Edition

Para completar este skills assessment, tenemos que responder a las preguntas aplicando las siguientes habilidades aprendidas en este módulo:

• Usar whois.\
• Analizar robots.txt.\
• Realizar ataques de fuerza bruta a subdominios.\
• Crawling y analizar resultados.

Hay que agregar subdominios al archivo de hosts a medida que los descubramos.

* Modificamos el fichero /etc/hosts para añadir el vHost que nos indican:

```
sudo nano /etc/hosts
```

* Añadimos las siguientes líneas:

```
83.136.251.13 inlanefreight.htb
83.136.251.13 inlanefreight.com
```

**Pregunta 1:** ¿Cuál es el ID de la IANA del registrador del dominio inlanefreight.com?

1. Usamos el comando whois sobre inlanefreight.com

```
whois inlanefreight.com
```

**Pregunta 2**: ¿Qué software de servidor HTTP está ejecutando el sitio inlanefreight.htb en el sistema de destino? Responda con el nombre del software, no con la versión, por ejemplo, Apache.

1. Hacemos el comando curl -I, en este caso indicando el puerto del target.

```
curl -I inlanefreight.htb:51868
```

**Pregunta 3**: ¿Cuál es la clave API del directorio oculto de administración que ha descubierto en el sistema de destino?

1. Usaremos gobuster para explorar más en profundidad, en este caso usaremos una wordlist y buscaremos por vhost:

```
gobuster vhost -u http://inlanefreight.htb:51868 -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

2. El output del comando nos devuelve el siguiente subdominio:

```
Found: web1337.inlanefreight.htb:51868 Status: 200 [Size: 104]
```

3. Lo añadiremos al /etc/hosts:

```
sudo nano /etc/hosts
```

```
83.136.251.13 web1337.inlanefreight.htb
```

4. La pregunta nos habla de un directorio admin oculto, vamos a buscarlo en el robots.txt del subdominio que acabamos de descubrir:

`curl -i web1337.inlanefreight.htb:51868/robots.txt`

5. En el output vemos un directorio admin\_h1dd3n que está en Disallow:

```
User-agent: *

Allow: /index.html
Allow: /index-2.html
Allow: /index-3.html
Disallow: /admin_h1dd3n
```

6. Haciendo el mismo curl que sobre robots.txt obtenemos el siguiente output:

```
curl -i web1337.inlanefreight.htb:51868/admin_h1dd3n/
<!DOCTYPE html><html><head><title>web1337 admin</title></head><body><h1>Welcome to web1337 admin site</h1><h2>The admin panel is currently under maintenance, but the API is still accessible with the key CLAVE_API</h2></body></html>
```

**Pregunta 4:** Después del crawling del dominio inlanefreight.htb en el sistema de destino, ¿cuál es la dirección de correo electrónico que ha encontrado? Responda con la dirección completa, por ejemplo, mail@inlanefreight.htb.

1. Repetimos el proceso de fuzzing de vhosts:

```
gobuster vhost -u http://inlanefreight.htb:51868 -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

* En el output no cambia nada y como ya hemos añadido el subdominio a /etc/hosts no habría que hacer nada más en este caso

2. Descargamos Scrapy y ReconSpider para el crawling:

```
pip3 install scrapy
```

```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip unzip ReconSpider.zip
```

```
unzip ReconSpider.zip
```

3. Usaremos ReconSpider para hacer crawling sobre el subdominio:

```
python3 ReconSpider.py web1337.inlanefreight.htb:51868
```

4. Nos guardará el resultado en el archivo results.json, lo mostramos y buscamos la dirección de email que se nos pide en la pregunta:

```
cat results.json
```

**Target**(s): 83.136.252.133:55224

5. Había que pensar un poco “out of the box” realizamos el fuzzing dentro del subdominio:

```
gobuster vhost -u http://web1337.inlanefreight.htb:55224 -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```

_Found: dev.web1337.inlanefreight.htb:55224 Status: 200 \[Size: 123]_

6. Añadimos este nuevo subdominio a /etc/hosts:

```
sudo sh -c "echo '83.136.252.133 dev.web1337.inlanefreight.htb' >> /etc/hosts"
```

7. Usamos Scrapy sobre el subdominio dev:

```
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:55224
```

```
cat results.json
{
    "emails": [
    "email@inlanefreight.htb"
    ],
}
```

**Pregunta 5:** ¿Cuál es la clave API que los desarrolladores de inlanefreight.htb también cambiarán?

1. En el results.json, abajo del todo, se nos muestra la API key.

```
cat results.json

"comments": [
    "<!-- Remember to change the API key to API_KEY -->"
]
```
