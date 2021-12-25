---
title: "Referencias de opciones de seguridad de FreeBSD 13.0 (post instalación)"
date: 2021-12-24
tags: ["FreeBSD", "Tutoriales", "Tutoriales FreeBSD", "Seguridad"]
author: "Juan David Hurtado G."
showToc: false
TocOpen: false
draft: false
hidemeta: false
description: "Cómo habilitar opciones de seguridad en FreeBSD 13.0"
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---
Este artículo no es un tutorial, pero tiene como finalidad referenciar las correspondencias asociadas al instalador de FreeBSD en la pantalla de «Configuración de seguridad» o «System hardening». Esto es, si quieres habilitar éstas opciones *luego* de haber instalado el sistema operativo.

![Configuración de seguridad o hardening](../como-instalar-freebsd-13-paso-a-paso/img/12_configuracion_seguridad.png)

A continuación se listan las opciones del instalador con la correspondiente configuración en el archivo indicado. Debes asegurarte de que dicha configuración esté presente en caso de que quieras habilitar esa opción.

| Opción del instalador | Configuración en /etc/sysctl.conf       |
|-----------------------|-----------------------------------------|
| hide_uids             | security.bsd.see_other_uids=0           |
| hide_gids             | security.bsd.see_other_gids=0           |
| hide_jail             | security.bsd.see_jail_proc=0            |
| read_msgbuf           | security.bsd.unprivileged_read_msgbuf=0 |
| proc_debug            | security.bsd.unprivileged_proc_debug=0  |
| random_id             | kern.randompid=1                        |

| Opción del instalador | Configuración en /etc/rc.conf |
|-----------------------|-------------------------------|
| clear_tmp             | clear_tmp_enable="YES"        |
| disable_syslogd       | syslogd_flags="-ss"           |
| disable_sendmail      | sendmail_enable="NONE"        |

| Opción del instalador | Configuración en /etc/ttys |
|-----------------------|----------------------------|
| secure_console        | console	none				unknown	off insecure |

| Opción del instalador | Configuración en /boot/loader.conf      |
|-----------------------|-----------------------------------------|
| disable_ddtrace       | security.bsd.allow_destructive_dtrace=0 |
