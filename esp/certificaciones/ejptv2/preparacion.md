---
icon: user-doctor-message
---

# Preparación

En este apartado os compartiré que hice para prepararme para lo que podría encontrarme en el examen de certificación. Quise ir muy preparado y os compartiré aquí los recursos que utilicé.

## Curso INE

Cuando compras la certificación, se te habilita una suscripción que te permite visualizar los contenidos de INE.

El curso orientado al eJPT en este caso es el [Penetration Testing Student](https://my.ine.com/CyberSecurity/learning-paths/61f88d91-79ff-4d8f-af68-873883dbbd8c/penetration-testing-student) dividido en las secciones que os detallado en el apartado de [**Cursos -> eJPTv2**](../../recursos/cursos/ejptv2.md)

Recomiendo verse todas las sesiones, aunque algunas sean cosas muy básicas, nunca viene mal para refrescar conocimientos. Alexis Hamed es muy buen instructor, tomad notas de lo que va explicando y mostrando, ya que durante el examen podéis acudir a ellas cuando dudéis sobre alguna técnica o vulnerabilidad.

## **Labs INE**

En el curso además de los vídeos explicando la teoría, hay vídeos más prácticos utilizando técnicas sobre laboratorios, también se incluyen estos laboratorios para poner en práctica todo lo que se te muestra en los vídeos.

Mi recomendación es hacer todos los laboratorios según avanzas en el curso, pero sobre todo practicar con los laboratorios "black box" que se encuentran en el apartado **Host & Network Penetration Testing: Exploitation.**

Hay tanto un laboratorio para Windows como para Linux. Estos laboratorios son lo más parecido a lo que te encontrarás en el examen.&#x20;

Estas secciones se centran en la explotación de diversos servicios dependiendo del sistema operativo. Se utilizan diferentes herramientas y técnicas de explotación, así que es importante completar estos labs y tomar nota de todas las alternativas que se te presentan.

## Labs Extra

En mi caso, realicé algunas máquinas en las siguientes plataformas:

### TryHackMe

En el caso de TryHackMe, realicé 2 máquinas:

* **Blue**: Máquina de Windows 7, vulnerable a EternalBlue.
  * Enlace: [https://tryhackme.com/r/room/blue](https://tryhackme.com/r/room/blue)
  * [WriteUp](../../writeups/tryhackme/blue.md)
* **Vulnversity:** Máquina Ubuntu ejecutando un servidor web
  * Enlace: [https://tryhackme.com/r/room/vulnversity](https://tryhackme.com/r/room/vulnversity)

### VulnHub

En la plataforma de VulnHub también realicé 2 máquinas, se parecen bastante a una o varias que os podéis encontrar en el propio examen:

* **DarkHole1**: Máquina muy completa en la que tendremos que emplear diferentes técnicas y herramientas para avanzar en ella.
  * Enlace: [https://www.vulnhub.com/entry/darkhole-1,724/](https://www.vulnhub.com/entry/darkhole-1,724/)
* Symfonos 1: Máquina centrada en el apartado web, en mi examen me encontré una bastante similar.
  * Enlace: [https://www.vulnhub.com/entry/symfonos-1,322/](https://www.vulnhub.com/entry/symfonos-1,322/)

## Creadores de contenido

### Xerosec

Por recomendación de un compañero que había aprobado la certificación recientemente, me vi el vídeo [Laboratorio de preparación eJPTv2 | Simulación de examen](https://www.youtube.com/watch?v=v20IsEd5nUU\&t=18s) :link: de Xerosec.&#x20;

Lo mejor que podéis hacer es veros el vídeo al completo (son 4 horas y media) y documentar las técnicas y metodología que se van realizando durante todo el vídeo, llegaréis con muy buena preparación al examen.

### J4ckie0x17

En este canal encontraréis una lista de reproducción de la explotación de 8 máquinas muy adecuadas para la certificación del eJPTv2. Algunas de ellas son las que os he mencionado anteriormente.

[Lista de reproducción](https://youtube.com/playlist?list=PLOaY4Tlt3Xl6BcWHHYPMjkzbItKiKE42u\&si=6-kck8uBZYZKKZhr) :link:

En los vídeos podéis ver diferentes técnicas de explotación de diferentes servicios que os pueden ser muy útiles en el momento del examen. Probad a hacerlas y tomad nota de los comandos que ejecutáis y de las técnicas que estáis realizando para luego saber como aplicarlo en el momento del examen.

## Antes del examen

El tiempo total para realizar el examen es de 48 horas.&#x20;

Mi recomendación es que lo iniciéis en un momento en el que tengáis tiempo (un fin de semana por ejemplo) y os lo toméis con calma, parad para comer, echaros la siesta o lo que queráis, ya que si os lo gestionáis bien os sobrará tiempo.

Mi consejo principal es que os lo toméis con calma y que estructuréis los apuntes que hayáis ido tomando durante el curso y en el momento de realizar los laboratorios para que os sea más fácil ubicar lo que podáis necesitar.

También una buena práctica es que os generéis la estructura de apuntes para el examen, porque aunque sea un examen, deberíais tomar apuntes de todo lo que hacéis para facilitaros la vida a vosotros mismos. Ya que si en algún momento tenéis que parar, luego lo podáis retomar justo donde os habéis quedado sin perder tanto tiempo.

Por ejemplo yo me cree la siguiente estructura para tomar anotaciones durante el examen:<br>

<figure><img src="../../../.gitbook/assets/image (473).png" alt=""><figcaption></figcaption></figure>

Cree los subnodos de Recon, Targets y Preguntas.

En Recon fui apuntando los comandos que utilicé para realizar el reconocimiento de la red y los resultados que iba obteniendo.

Una vez identifiqué todos los hosts, cree el subnodo Targets y dentro de él un subnodo por cada máquina identificada. A su vez dentro de cada subnodo de máquina, otros subnodos para cada fase en la que me encontraba en esa máquina o el servicio que estaba vulnerando.&#x20;

Por ejemplo:

<figure><img src="../../../.gitbook/assets/image (475).png" alt=""><figcaption></figcaption></figure>

Esto me permitió estructurar lo que tenía de una forma más óptima, y en los momentos que hacía alguna pausa, retomar rápidamente donde me había quedado o saber que técnicas ya había probado sobre un objetivo y no había resultado. También apuntaba credenciales obtenidas u otra información relevante que me pudiera servir.
