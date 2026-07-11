---
title: "Cómo Instalar FreeBSD con entorno de escritorio Xfce"
date: 2025-06-03T20:33:06-05:00
tags: ["freebsd", "xfce", "desktop", "Tutoriales", "Tutoriales FreeBSD"]
author: "Juan David Hurtado G."
showToc: true
TocOpen: false
draft: false
hidemeta: false
description: "Guía paso a paso para instalar un entorno de escritorio Xfce en FreeBSD en pocos pasos: paquetes, controlador de video, grupo video y arranque con startx."
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---
Instalar un entorno de escritorio Xfce en FreeBSD es un proceso bastante directo.

Con el fin de usar un entorno de escritorio liviano y que sea útil para la mayoría de los casos, he decidido realizar un paso a paso mostrando cómo hacerlo con Xfce, ya que consume relativamente pocos recursos y sería útil para cualquier usuario que tenga un ordenador nuevo o viejo, con FreeBSD.

Este artículo sirve como la versión escrita del video, para que puedas seguir el procedimiento a tu propio ritmo, copiar los comandos y volver a consultarlo cuando lo necesites. La idea es que con apenas cuatro o cinco pasos sencillos tengamos un escritorio funcional.

{{< youtube DU5R8Z1w73w >}}

## Sobre esta instalación

Para el video parto de una instalación de FreeBSD **desde cero**, arrancando el instalador desde una memoria USB. Si aún no conoces ese proceso, puedes seguir la guía de [cómo instalar FreeBSD paso a paso]({{< relref "como-instalar-freebsd-13-paso-a-paso" >}}); aquí solo menciono lo que cambia:

- Uso **FreeBSD 15**, que al momento de grabar es una versión de desarrollo (no recomendada para producción, pero muy cómoda para este tipo de pruebas). Su instalador tiene una ventaja: **detecta y ofrece habilitar controladores** —tanto de red como de video— desde el propio asistente.
- El equipo es un MacBook cuya tarjeta de red interna no tiene controladores en FreeBSD, así que uso un **adaptador de red USB** (un TP-Link con chipset RealTek). El instalador lo reconoce; basta con seleccionar *auto* para que habilite el controlador y conectarse a la red WiFi desde ahí.
- Como sistema de archivos uso **UFS** sobre una tabla de particiones **GPT**, usando el disco completo (con ZFS también funcionaría perfectamente).
- Durante la instalación agrego mi usuario estándar al **grupo `wheel`**, para poder luego escalar a *root* con `su` desde ese usuario.

Nota: Todos los comandos que instalan paquetes o modifican el sistema se ejecutan como *root*. Los que configuran el entorno de tu usuario (como crear `~/.xinitrc`) se ejecutan con tu usuario normal.

## Los pasos en resumen

Una vez el sistema está instalado y arrancado, el resto es corto:

1. Instalar el entorno de escritorio Xfce y sus dependencias.
2. Instalar el controlador gráfico correspondiente a tu hardware.
3. Habilitar la carga del controlador gráfico al inicio.
4. Agregar tu usuario al grupo `video`.
5. Configurar el comando de arranque del escritorio.

## Instalar Xfce y el controlador de video

Usamos `pkg` para instalar el entorno de escritorio. El paquete `xfce` trae consigo el escritorio (panel, administrador de ventanas, *Thunar*, la terminal, etc.) junto con las dependencias del servidor gráfico Xorg:

```sh
pkg install xfce
```

Enseguida instalamos el **controlador gráfico** según la tarjeta del equipo. En este caso el equipo tiene gráficos **Intel**, por lo que instalo el paquete de controladores del kernel `drm-kmod`:

```sh
pkg install drm-kmod
```

Nota: Cuál controlador necesitas depende de tu hardware. Para **Intel** y **AMD** el paquete es `drm-kmod`; para **NVIDIA** se usa `nvidia-driver`. Lo que cambia entre Intel y AMD no es el paquete, sino el módulo que se carga al inicio (lo vemos en el siguiente paso).

## Habilitar el controlador gráfico al inicio

Para que el módulo del controlador gráfico se cargue automáticamente cada vez que arranca el sistema, lo agregamos a la lista de módulos del kernel (`kld_list`) con la herramienta `sysrc`:

```sh
sysrc kld_list+=i915kms   # Intel
```

Para otras tarjetas debes usar el módulo que corresponda; por ejemplo `amdgpu` para AMD. Si tienes gráficos Intel, revisa bien qué generación es para confirmar el módulo adecuado.

## Agregar el usuario al grupo video

Para que tu usuario estándar (no *root*) pueda iniciar el servidor X, debe pertenecer al **grupo `video`**. Lo agregamos con `pw groupmod`, usando `-m` para añadir un nuevo miembro:

```sh
pw groupmod video -m juan
```

(Reemplaza `juan` por el nombre de tu usuario.)

Nota: Si además quieres poder apagar o reiniciar el equipo desde la interfaz gráfica, puedes agregar tu usuario a grupos adicionales como `operator`. Como en la instalación ya lo agregué al grupo `wheel`, ese usuario también puede iniciar procesos como *root*.

## Reiniciar para validar el controlador

Antes de arrancar el escritorio, conviene reiniciar para comprobar que el controlador gráfico se carga al inicio:

```sh
reboot
```

Cuando el controlador Intel/AMD carga correctamente, la pantalla suele hacer un breve **destello** durante el arranque: esa es la señal de que el módulo se cargó y el modo gráfico quedó activo.

## Iniciar el escritorio con startx

Ya con sesión iniciada como tu usuario estándar, podrías arrancar el escritorio directamente con `startxfce4`. Sin embargo, la práctica común es crear un archivo `~/.xinitrc` en la carpeta personal del usuario que indique qué debe ejecutarse al lanzar `startx` (el comando estándar para iniciar una sesión gráfica).

Como el archivo tendrá una sola línea, lo creamos al vuelo con `echo`:

```sh
echo "exec /usr/local/bin/startxfce4" >> ~/.xinitrc
```

Y arrancamos el escritorio con:

```sh
startx
```

`startx` buscará ese `~/.xinitrc` y ejecutará el comando indicado para lanzar Xfce.

Nota: Cuida bien de escribir `startxfce4` correctamente. Un error de tipeo en `~/.xinitrc` hará que `startx` no inicie el escritorio (algo fácil de pasar por alto). Si `startx` funciona como *root* pero no con tu usuario, revisa ese archivo.

## Ajustar el escalado en pantallas HiDPI / Retina

Si tu equipo tiene una pantalla de alta densidad (como la pantalla Retina del MacBook de este ejemplo), es probable que al inicio todo se vea demasiado pequeño. Se corrige fácil:

1. Abre el menú y ve a **Settings** (Configuración).
2. Entra a **Appearance** (Apariencia).
3. En la pestaña **Settings**, ajusta el *Window Scaling* a **2x**.

Con eso la interfaz queda a un tamaño mucho más cómodo para esa resolución.

## Y eso es todo

Con estos pocos pasos ya tienes un escritorio Xfce liviano y funcional sobre FreeBSD. De aquí en adelante todo es personalización a tu gusto: el navegador que quieras instalar, el tema, los paneles y las aplicaciones de trabajo.

¿Qué aplicaciones vas a instalar primero en tu nuevo escritorio? ¿Hay algo en especial que te gustaría ver en un próximo video?

Deja tus comentarios y comparte el conocimiento.
