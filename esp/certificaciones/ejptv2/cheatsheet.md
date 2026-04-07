---
icon: list-check
---

# CheatSheet

## Introducción

En este apartado os comparto un listado de los comandos que más útiles me han sido en las diferentes fases del examen, con las opciones, que en mi caso, funcionaron mejor y me permitieron aprobar.

## Reconocimiento

#### Fping

```bash
fping -a -g <IP>
```

-a: Esta opción indica que muestre las IP de los hosts que están activos en la red.

-g: Este argumento especifica que el ping es a un rango de IP.

#### Netdiscover

```bash
netdiscover -r <IP>
```

-r Para indicar una red o rango.

#### Nmap

```bash
nmap -sn 192.168.100.0/24
```

-sn: Esta opción Indica un "ping scan", no escanea puertos, solo se verifica si los hosts están activos.

#### [Ping Sweep](https://www.rubyguides.com/2012/02/cli-ninja-ping-sweep/) :link:

Otra alternativa al reconocimiento de red, que sería igual que la anterior de Nmap. Copiar y pegar directamente en el terminal.

Linux:

```bash
for i in {1..254} ;do (ping -c 1 192.168.1.$i | grep "bytes from" &) ;done
```

Windows:

```bash
for /L %i in (1,1,255) do @ping -n 1 -w 200 192.168.1.%i > nul && echo 192.168.1.%i is up.
```

## Enumeración

Aquí se agruparé los comandos para identificar puertos, servicios, versiones, usuarios, etc.

### Nmap

La herramienta nmap será nuestra mejor aliada para realizar un buen reconocimiento de los hosts de la red y los servicios que están ejecutando.

```bash
nmap -sC -sV -O <IP>
```

-sC: Ejecución de scripts predeterminados, estos nos permiten identificar vulnerabilidades a las que puede ser vulnerable el servicio u obtener información adicional.

-sV: Identifica la versión del servicio que se está ejecutando en el puerto, y por tanto, podemos saber si ese servicio es vulnerable a alguna vulnerabilidad conocida.

-O: Determina el sistema operativo que está ejecutando el sistema objetivo.

### Curl

En mi examen me encontré con dos máquinas ejecutando CMS, cada una ejecutando uno distinto, para obtener información que se pedía en alguna de las preguntas usé los siguientes comandos:

```bash
curl <dominio o IP> | grep 'Nombre del CMS'
```

```bash
curl -s <IP o dominio + ruta del archivo de configuración>
```

### Metasploit

Metasploit tiene módulos de enumeración para diferentes servicios, es una buena alternativa si no conoces herramientas o métodos concretos.

```bash
msfconsole -q
msf6 > search smb_enumusers
msf6 > use 1
msf6 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS <IP>
msf6 auxiliary(scanner/smb/smb_enumusers) > set SMBUser <usuario>
msf6 auxiliary(scanner/smb/smb_enumusers) > set SMBPass <contraseña>
msf6 auxiliary(scanner/smb/smb_enumusers) > run
```

### Enum4linux

Para un sistema Linux, una de las herramientas de enumeración que puedes utilizar:

```bash
enum4linux <IP>
```

## Explotación

### Metasploit

Gracias a sus módulos podrás utilizarlo para vulnerar múltiples servicios.

Comandos básicos:

```bash
// Iniciar consola sin banner
msfconsole -q

// Buscar un módulo
msf6 > search <Nombre servicio o vulnerabilidad>

// Usar módulo
msf6 > use <Número del módulo o ruta>

// Establecer opción global
msf6 > setg <Nombre opción> <Valor opción>

// Establecer Host objetivo
msf6 > set RHOSTS <IP>

// Establecer Puerto objetivo
msf6 > set RPORT <Número de puerto>

// Opciones configuradas del módulo
msf6 > options

// Información sobre el módulo
msf6 > info

// Ejecutar módulo
msf6 > run
```

### Brute Force

En los labs del curso, la fuerza bruta es una técnica recurrente, en el caso del examen también.&#x20;

Te facilitará mucho el obtener credenciales para usuarios. Para mí, el combo ganador fue hydra con rockyou.txt.

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt <servicio>://<IP>
```

## Post Explotación

Una vez obtenido acceso al sistema objetivo, deberás moverte por él para encontrar lo que buscas. Para ello deberás utilizar comandos de sistema, dependiendo del sistema que estés atacando.

Ejemplo de algunos de los comandos que tuve que utilizar:

```bash
// Identificar que usuario eres
whoami

// Listar usuarios del sistema
net user

// Descargar un archivo via FTP
ftp> get archivo.txt

// Subir un archivo via FTP
ftp> put archivo.txt

// Pasar desde meterpreter a una shell
meterpreter> shell
/bin/bash -i
```
