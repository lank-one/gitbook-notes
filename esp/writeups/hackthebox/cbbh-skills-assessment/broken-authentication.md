---
description: 'Target(s): 94.237.121.185:48488'
---

# Broken Authentication

## Escenario

Se le ha encomendado realizar una evaluación de seguridad de la aplicación web de un cliente. Para la evaluación, el cliente no le ha proporcionado credenciales. Aplique lo que ha aprendido en este módulo para obtener la bandera.

**Pregunta 1**: Obtén la flag

• Será una pentest black box porque no tenemos credenciales disponibles así que vamos a realizar un reconocimiento básico y ver que es lo que podemos obtener.\
1\. Accedemos al formulario de registro e introducimos las credenciales test:test&#x20;

<figure><img src="../../../../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure>

* Vemos los criterios que tiene que cumplir la contraseña para el usuario, con estos parámetros podemos crear una wordlist.

2. Registramos un usuario con unas credenciales que cumplan los requisitos Test:Password1234

<figure><img src="../../../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

3. Iniciamos sesión y hay un aviso de que nuestra cuenta no tiene privilegios de administrador:

<figure><img src="../../../../.gitbook/assets/image (373).png" alt=""><figcaption></figcaption></figure>

4. Vamos a configurar un ataque de fuerza bruta, para ello intentamos loguearnos con nuestra cuenta con una contraseña incorrecta para ver el mensaje que devuelve la página:

<figure><img src="../../../../.gitbook/assets/image (374).png" alt=""><figcaption></figcaption></figure>

5. Ahora que tenemos el mensaje de error y los criterios de registro de contraseña, vamos a hacer primero la wordlist:

```
grep -E '^[[:alnum:]]{12}$' /usr/share/wordlists/rockyou.txt | grep -E '[0-9]' | grep -E '[a-z]' | grep -E '[A-Z]' > passwords.txt
```

<figure><img src="../../../../.gitbook/assets/image (375).png" alt=""><figcaption></figcaption></figure>

6. Ahora vamos a buscar un nombre de usuario válido, que nos desvuelva el mensaje de “Invalid credentials”:

**Target**(s): 94.237.59.174:32711

```
ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.59.174:32711/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=1" -fr "Unknown username or password."
```

<figure><img src="../../../../.gitbook/assets/image (376).png" alt=""><figcaption></figcaption></figure>

* Identificamos el usuario.

7. Montamos el comando para realizar la fuerza bruta:

```
ffuf -w passwords.txt -u http://94.237.59.174:32711/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=gladys&password=FUZZ" -mc 302
```

<figure><img src="../../../../.gitbook/assets/image (377).png" alt=""><figcaption></figcaption></figure>

* Encontramos la contraseña.

8. Iniciamos sesión con las credenciales y nos encontramos con la página de OTP:

<figure><img src="../../../../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

Intentamos entrar por URL acceder a profile.php pero el 2FA no nos lo permite.&#x20;

9. Pero si capturamos la request con Burp, aunque nos redirija al 2FA, la flag viene embebida en el response al intento de acceso a profile.php:

<figure><img src="../../../../.gitbook/assets/image (379).png" alt=""><figcaption></figcaption></figure>
