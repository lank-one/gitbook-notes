---
description: 'Difficulty: Medium'
---

# Bola

### Recon

1. Hacemos un reconocimiento de la máquina objetivo directamente:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.104
```

<figure><img src="../../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22/tcp
  * 80/tcp
  * 873/tcp

2. El puerto 80 está abierto, accedemos desde el navegador a ver que nos encontramos:

<figure><img src="../../../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

3. Añadiremos a /etc/hosts la IP que apunte a bola.nyx a ver si resuelve:

```bash
sudo nano /etc/hosts

192.168.56.104 bola.nyx
```

<figure><img src="../../../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

4. Volvemos a entrar desde el navegador:

<figure><img src="../../../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>

* Ahora nos aparece que se ha añadido una nueva funcionalidad.

5. Hacemos click en el botón de Login:

<figure><img src="../../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

* Nos lleva a un formulario de login. No tenemos ningún tipo de credencial, así que el siguiente paso sería realizar un fuzzeo.

6. &#x20;Primero utilizaremos dirsearch para descubrir extensiones que nos ayudarán con el fuzzeo de los endpoints dentro de la web.

```bash
dirsearch -u http://bola.nyx
```

<figure><img src="../../../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

* Encontramos dos archivos txt:
  * security.txt
  * openid-configuration
* Y archivos y directorios interesantes, algunos que ellos redirigen:
  * /admin -> http://bola.nyx/admin/
  * /admin/admin.php -> /login/login.php
  * /download.php -> /login/login.php
  * /javascript -> http://bola.nyx/javascript/
  * /login -> http://bola.nyx/login/

7. Revisamos que contienen los archivos encontrados:

**/.well-known/security.txt**

<figure><img src="../../../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

* Poca cosa encontramos, el email de jackie0x17@nyx.com

**/.well-known/openid-configuration**

<figure><img src="../../../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

* Se muestra un listado de usuarios, con mails, nombres de usuarios, etc.

### Enumeración

#### Rsync

1. Vamos a comprobar el servicio que se ejecuta en el puerto 873:

<figure><img src="../../../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

* Rsync tiene un archivo de configuración con un apartado public\_files, que puede estar en yes o no, si está en yes podemos ver los módulos al ejecutar el comando. En el caso de la máquina Bola, está en no.

2. Cómo no podemos listarlos, habrá que fuzzearlo. Para ello utilizaremos la herramienta rsync-brute ([https://github.com/VulNyx/Arsenal/tree/main/rsync-brute](https://github.com/VulNyx/Arsenal/tree/main/rsync-brute)), que es de Vulnyx y nos facilitará este paso:

<pre><code><strong>wget --no-check-certificate -q "https://raw.githubusercontent.com/VulNyx/Arsenal/refs/heads/main/rsync-brute/rsync-brute" -O /usr/bin/rsync-brute &#x26;&#x26; chmod +x /usr/bin/rsync-brute
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

3. Seguimos el ejemplo de uso que se nos muestra:

```bash
rsync-brute -t bola.nyx -p 873 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

<figure><img src="../../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

* Encontramos el recurso extensions.

4. Probaremos rsync sobre el recurso extensions:

```bash
rsync rsync://bola.nyx/extensions
```

<figure><img src="../../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

* Encontramos 2 archivos:
  * Password\_manager\_FirefoxExtension-VulNyx.pdf
  * password\_manager.zip

5. Descargaremos los archivos:

```bash
sudo rsync -avz --progress rsync://bola.nyx/extensions/Password_manager_FirefoxExtension-VulNyx.pdf .
```

```bash
sudo rsync -avz --progress rsync://bola.nyx/extensions/password_manager.zip .
```

* Desglose del comando:
  * -a → Mantiene los permisos, las fechas, los propietarios y la estructura de carpetas.
  * -v → Muestra cada archivo a medida que se descarga.
  * -z → Comprime los datos durante la transferencia para que sea más rápida.
  * \--progress → Muestra el progreso de descarga del archivo.
  * . → Al final del comando, indica que se descargue en el directorio que nos encontramos actualmente.

<figure><img src="../../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

4. Mostramos el contenido del pdf y vemos que es un manual de instalación de una extensión de navegador:

<figure><img src="../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

5. Seguimos el manual, e instalamos la extensión:

* En el navegador Firefox accedemos a: <kbd>about:debugging#/runtime/this-firefox</kbd>

<figure><img src="../../../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

* Hacemos click en **Load Temporary Add-on...** seleccionamos **password\_manager.zip** y se nos instalará. Lo abrimos:

<figure><img src="../../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

* Vemos una contraseña guardada, que al hacer click en Show podemos visualizarla:

<figure><img src="../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

6. Probaremos de acceder a bola.nyx con estas credenciales:

<figure><img src="../../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

7. Y obtenemos acceso:

<figure><img src="../../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

### IDOR

* El archivo PDF que encontramos en el Portal Manager tiene el nombre encriptado.

1. Haremos click sobre él para descargarlo.

<figure><img src="../../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

2. El nombre del archivo está encriptado, y pertenece al user jackie0x17, probamos a encriptar este nombre de usuario a ver si coincide con el nombre del archivo:

```bash
echo -n "jackie0x17" | base64
```

<pre class="language-bash"><code class="lang-bash"><strong>echo -n "jackie0x17" | md5sum
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

* El nombre del archivo es el nombre del usuario owner, encriptado en md5. Al mostrar el contenido del archivo /.well-known/openid-configuration se listaban varios usuarios. Siguiendo esta lógica, podríamos descargar sus PDF cambiando el nombre del archivo de la URL por el nombre del usuario encriptado en MD5.

3. Encriptamos los nombres de usuarios encontrados en openid-configuration:

* **d4t4s3c**

```bash
echo -n "d4t4s3c" | md5sum
```

<figure><img src="../../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

* **ct0l4**

```bash
echo -n "ct0l4" | md5sum
```

<figure><img src="../../../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

4. Descargamos los archivos de los usuarios, cambiando el nombre del archivo en la URL:

• **d4t4s3c**:

<figure><img src="../../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

* **ct0l4:**

<figure><img src="../../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

* Del único que existe archivo es del usuario d4t4s3c, y además es el tutorial de conexión al servidor WSDL.

5. En la página 2 del pdf asociado al usuario d4t4s3c, encontramos una contraseña relacionada al usuario admin y al principio de la página 3, un puerto posiblemente interno porque no se ha identificado durante los escaneos de puertos externos:

<figure><img src="../../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

6. Con las credenciales que hemos encontrado y sabiendo que el puerto de servicio SSH está abierto, probaremos a conectarnos:

```bash
ssh d4t4s3c@192.168.56.104
```

<figure><img src="../../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

* Sabiendo que existe el puerto 9000 cómo hemos visto en el pdf de d4t4s3c y que previamente en los escaneos no aparecía, podemos concluir que es un puerto interno.

### Port Forwarding

1. Ahora que tenemos credenciales válidas, podemos hacer un forwarding de puertos por ssh. Cuando nos conectemos, crearemos un túnel que hace que el puerto 9000 pase a ser nuestro y podamos acceder a él:

```bash
ssh -L 9000:127.0.0.1:9000 d4t4s3c@192.168.56.104
```

<figure><img src="../../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

2. Desde el navegador accedemos a localhost:9000 y nos muestra el servidor WSDL.

<figure><img src="../../../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

### WSDL

1. Accedemos al localhost:9000 y al archivo wsdl:

<figure><img src="../../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

2. Descargamos y revisamos el archivo al completo.
3. Hay que entender la estructura de SOAP WSDL, que es la que se nos muestra al acceder a localhost:9000, este será nuestro punto de explotación.

<figure><img src="../../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

### SOAP Action Spoofing

1. Abrimos Burp Suite y capturaremos una request a localhost:9000 para enviarla al Repeater:

<figure><img src="../../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

2. Al enviarla desde el Repeater, la propia web nos dice que tenemos que enviar una request POST con el header Content-Type bien configurado:

<figure><img src="../../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

3. Vamos a cambiar de GET a POST y a introducir la estructura del SOAP WSDL, modificando los tags interiores y añadiendo y para permitir que se ejecuten comandos a través de la request:

```xml
<soap11env:Envelope xmlns:soap11env="http://schemas.xmlsoap.org/soap/envelope/">
    <soap11env:Body>
        <ExecuteCommand>
            <cmd>

                pwd
            </cmd>
        </ExecuteCommand>
    </soap11env:Body>
</soap11env:Envelope>
```

<figure><img src="../../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

* Nos informa de que sólo está permitido en redes internas.

4. Cómo detecta el intento de ejecución desde el exterior por el tag , haremos la trampa de incluir el header SOAPAction para hacer al SOAP ejecutar la acción sin que se detecte a través de los tags. También cambiaremos el tag de por el de , que no tiene restricciones:

```xml
SOAPAction: ExecuteCommand

<?xml version="1.0" encoding="utf-8"?>
   <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
   xmlns="http://localhost/wsdl">
   <soapenv:Header/>
      <soapenv:Body>
         <LoginRequest> 
            <cmd>hostname</cmd>
         </LoginRequest>
      </soapenv:Body>
   </soapenv:Envelope>
```

<figure><img src="../../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

5. Esto nos sirve para bypassear la limitación de sólo desde redes internas y por lo tanto, podremos ejecutar una reverse shell para hacernos con el control del objetivo.

* Ponemos una consola a la escucha en nuestra máquina atacante:

```bash
nc -lvnp 4444
```

<figure><img src="../../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

* Enviamos una busybox a través de la request de Burp:

<figure><img src="../../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

6. Recibimos la conexión en el terminal y exploramos la máquina objetivo en busca de las flag:

<figure><img src="../../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>
