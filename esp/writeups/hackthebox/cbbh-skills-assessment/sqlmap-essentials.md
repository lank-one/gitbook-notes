---
description: 'Target(s): 83.136.255.230:43383'
---

# SQLMap Essentials

Se le proporciona acceso a una aplicación web con mecanismos de protección básicos. Utilice los conocimientos adquiridos en este módulo para encontrar la vulnerabilidad SQLi con SQLMap y explotarla adecuadamente. Para completar este módulo, encuentre la bandera y envíela aquí.

**Pregunta 1**: ¿Cuál es el contenido de la tabla final\_flag?

1. Arrancamos Burp y vamos a la pestaña de Proxy para asegurarnos que intercept está en modo ON.
2. A través del navegador visitamos la página web objetivo y activamos la extensión de FoxyProxy para que redirija el tráfico a Burp.
3. Navegamos por la web buscando vectores de ataque. Investigando las páginas, llegamos hasta la de shop.html, en la que hay un script al final del código que crea un JSON:

<figure><img src="../../../../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

4. Añadimos un objeto al carrito para capturar la petición que ejecuta este script y la guardamos en un archivo:

<figure><img src="../../../../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>

5. Ejecutamos el siguiente comando de sqlmap sobre la petición guardada:

```
sqlmap -r addcart.txt --threads 10 --tamper=between -T final_flag --dump --batch --tamper=between
```

1. Aplicamos el script tamper llamado between, que modifica los payloads de inyección SQL para evadir mecanismos de defensa. Este script reemplaza el operador > por NOT BETWEEN 0 AND # y el operador = por BETWEEN # AND #, lo que ayuda a saltar filtros.
2. \--dump : Ordena a SQLmap que extraiga y muestre todo el contenido de la tabla especificada (final\_flag).
3. El comando realizará varias pruebas de inyección, identificará el valor vulnerable y nos acabará mostrando el output:

<figure><img src="../../../../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>
