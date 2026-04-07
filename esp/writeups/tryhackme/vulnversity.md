---
hidden: true
icon: arrow-turn-down-right
---

# Vulnversity

## Introducción

Esta máquina ejecuta un servidor web, deberemos realizar la explotación sobre este servicio.

## Task 1: Deploy the machine

Nos conectamos a la red de TryHackMe o iniciamos la Attackbox de navegador y arrancamos la máquina.

## Task 2: Reconnaissance

Nos indicarán el comando de Nmap que debemos utilizar para responder las preguntas de este apartado:

```bash
nmap -sV <IP>
```

En la tabla de flags de uso para Nmap, nos muestran diferentes flags que nos permetien responder el resto de preguntas.

## Task 3: Locating directories using Gobuster

Para responder la única pregunta de este apartado, utilizaremos Gobuster para identificar por fuerza bruta los directorios en la aplicación web de la máquina víctima:

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>gobuster dir -u http://&#x3C;IP>:&#x3C;puerto> -w &#x3C;path_completo/archivo_diccionario.txt>
</strong></code></pre>

## Task 4: Compromise the WebServer

En este apartado se nos indican unos pasos a seguir con Burp Suite, también como crear una reverse shell en php y como poner una consola a la escucha con netcat para esperar la conexión desde la víctima.

Crear archivo con extensiones de php:

```bash
nano phpextensions.txt
<extensiones>
[Ctrl+O]
[Ctrl+X]

// Visualizar contenido del archivo
cat phpextensions.txt
```

Poner a la escucha una consola con Netcat:

```bash
nc -lvnp <puerto>
```

