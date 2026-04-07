---
description: 'Target(s): 94.237.53.49:47319'
---

# JavaScript Deobfuscation

Durante nuestra pentest, encontramos un servidor web que contiene JavaScript y APIs. Necesitamos determinar su funcionalidad para comprender cómo puede afectar negativamente a nuestro cliente.

**Pregunta 1**: Intenta estudiar el código HTML de la página web e identificar el código JavaScript utilizado en ella. ¿Cuál es el nombre del archivo JavaScript que se está utilizando?

1. Con inspeccionar el código fuente, dentro del tag \<script> del HTML encontraremos la respuesta:

<figure><img src="../../../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

**Pregunta 2**: Una vez que encuentres el código JavaScript, intenta ejecutarlo para ver si realiza alguna función interesante. ¿Has obtenido algún resultado?

1. Inspeccionando el código de la página, vamos a la pestaña _Console_:

<figure><img src="../../../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

**Pregunta 3:** Como habrás notado, el código JavaScript está ofuscado. Intenta aplicar los conocimientos que has adquirido en este módulo para desofuscar el código y recuperar la variable «flag».

1. Pasamos el código por UnPacker.

Pasamos el código por UnPacker:

```
function apiKeys() 
{ 
    var flag='HTB 
    { 
        n'+'3v3r_'+'run_0'+'bfu5c'+'473d_'+'c0d3!'+' 
    } 
    
    ',xhr=new XMLHttpRequest(),_0x437f8b='/keys'+'.php'; 
    xhr['open']('POST',_0X437f8b,!![]),xhr['send'](null)
}
console['log']('HTB 
{ 
    j'+'4v45c'+'r1p7'+'3num3'+'r4710'+'n_15_'+'k3y 
} 
');
```

**Pregunta 4**: Intenta analizar el código JavaScript desofuscado y comprender su funcionalidad principal. Una vez que lo hayas hecho, intenta replicar lo que hace para obtener una clave secreta. ¿Cuál es la clave?

1. Haremos un curl contra la IP objetivo y el archivo keys.php:

```
curl -s http://94.237.63.49:47319/keys.php -X POST -d "null"
```

2. Obtendremos la key en la respuesta.

**Pregunta 5**: Una vez que tengas la clave secreta, intenta determinar su método de codificación y descodifícala. A continuación, envía una solicitud «POST» a la misma página anterior con la clave descodificada como «key=DECODED\_KEY». ¿Cuál es la bandera que has obtenido?

1. Decodificamos la key que está en hexadecimal:

```
echo KEY_CODIFICADA | xxd -p -r
```

2. Obtendremos la key decodificada.
3. La utilizaremos para obtener la flag:

```
curl -s http://94.237.63.49:47319/keys.php -X POST -d "key=KEY_DECODIFICADA"
```
