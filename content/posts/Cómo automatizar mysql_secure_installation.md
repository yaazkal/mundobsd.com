---
title: "Cómo Automatizar Mysql_secure_installation en FreeBSD"
aliases: ["como-automatizar-mysql_secure_installation"]
date: 2021-08-21T10:26:26-05:00
tags: ["bases de datos","automatización","scripts","/bin/sh","mariadb","mysql"]
author: "Juan David Hurtado G."
showToc: true
TocOpen: false
draft: false
hidemeta: false
description: "Se puede automatizar la ejecución de `mysql_secure_installation` utilizando el comando yes o bien usando un operador de redirección."
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---
Mientras estaba realizando un template para [BastilleBSD][1] en el que necesitaba instalar [MariaDB][2] como gestor de bases de datos, me hice la pregunta de cómo automatizar la ejecución del comando `mysql_secure_installation` para no necesitar la intervención del usuario. **Me llevé una sorpresa** y di con una **solución muy sencilla** para mi caso.

## Sí es posible

Mirando la documentación oficial de MariaDB (y la de MySQL también), encuentro que el *script* no recibe argumentos para dar respuestas a las preguntas que realiza durante la ejecución; por lo tanto «no era posible» realizar la deseada automatización.

Recurriendo entonces a una búsqueda por internet, me encontré que las personas estaban creando sus propios *scripts* que hacían lo mismo que el *script* en mención, ya que «no era posible».

Incluso revisé el código de uno que otro software que utilizo para ver ellos cómo estaban automatizando esta instalación y el resultado era que efectivamente estaban recreando el *script* en mención de acuerdo a las necesidades del proyecto.

Para no reinventar la rueda y dado que en mi caso necesitaba dar como respuesta sí a todas las preguntas del *script*, decidí hacer uso del comando [yes][3] y luego establecí la contraseña para *root* de manera aleatoria, así:

```sh
yes | mysql_secure_installation >/dev/null 2>&1
RANDOM_PASSWORD=$(openssl rand -base64 15)
mysql -u root -pyes -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '${RANDOM_PASSWORD}';"
```

¡Y listo! **¡Sí era posible!**

¿Por qué nadie estaba usando esta estrategia? :thinking:

## ¿Qué hace el script `mysql_secure_installation`?

Luego de cada instalación de MariDB o de MySQL es recomendable, por seguridad, ejecutar el comando `mysql_secure_installation` ya que le pregunta al usuario si desea deshabilitar algunas características que trae por defecto el gestor de bases de datos. A saber, el *script* permite:

* Establecer la contraseña del usuario *root* del gestor de base de datos.
* Deshabilitar el acceso de la cuenta *root* del gestor de base de datos desde equipos externos
* Eliminar las cuentas anónimas.
* Eliminar la base de datos *test* que por defecto puede ser accedida por los usuarios anónimos.

## Explicación paso a paso

A grandes rasgos lo que estoy realizando es establecer inicialmente la contraseña *yes* para el usuario *root* del gestor de base de dados y luego contesto sí a todo lo otro que pregunte el *script*. Una vez se termina de ejecutar el *script*, cambio la contraseña del usuario *root* que antes era la palabra *yes* (inseguro) por una contraseña aleatoria de 15 caracteres.

#### Línea 1

`yes | mysql_secure_installation >/dev/null 2>&1`

Haciendo uso del [pipeline][4] del sistema operativo, ejecuto el script contestado a todo *yes* pero sin mostrar algo; por eso el uso de `>/dev/null 2>&1`. Como se pretende usar esto en automatización no es estrictamente nesecesario mostrar todo le texto que arroja el script; si mucho un mensaje personalizado con el comando `echo` (opcional).

Esto tiene un punto débil y es que ahora la contraseña del usuario root del gestor de bases de datos es la palabra `yes`, pero eso se soluciona en los siguientes dos pasos.

#### Línea 2

`RANDOM_PASSWORD=$(openssl rand -base64 15)`

Establezco una variable `RANDOM_PASSWORD` de 15 caracteres (mucho más segura que la palabra yes) que será generada aleatoriamente gracias al comando [openssl][5]

#### Línea 3

`mysql -u root -pyes -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '${RANDOM_PASSWORD}';"`

Utilizando el comando [mysql][6] cambio la contraseña del usuario root del gestor de base de datos por la nueva y más segura generada anteriormente.

¿Notaste que me identifico con la contraseña `yes` durante la ejecución de `mysql` a través de la opción `-p`?

## ¿Cómo puedo usar esto?

La línea que efectivamente resuelve el problema de usar `mysql_secure_installation` de manera programática es únicamente la línea número 1. Las restantes son opcionales pero recomendadas.

Luego puedes mostrar la contraseña generada, guardarla en un archivo o bien seguirla usando en tu propio *script*.

En mi caso, luego de esto creo el archivo `/root/.my.cnf` en el servidor o contenedor (jail) de destino, con dos fines: 1) poder usar `mysql` sin tener que ingresar la contraseña cada vez y 2) dejar registro de cuál fue la clave asignada al usuario *root*.

De igual manera, sólo lo ejecuto una sola vez ya que lo ejecuto durante el proceso de aprovisionamiento. Como [aquí][7] en el template que estaba desarrollando para poder ejecutar [Matomo][8] en un contenedor (jail) utilizando [BastilleBSD][1].

## ¿Y si no quiero responder sí a todo?

> "Voy a tomar toda esta negatividad y usarla como combustible" - Pat *(Bradley Cooper)*.

Hay alternativas. Puedes usar en ese caso un operador de redirección como `<<`, haciendo uso de lo que se conoce como un *here-document*.

Esto incluso te permitirá establecer una clave para el usuario *root* desde el principio o usar una en caso de que ya esté establecida.

En este **ejemplo**, el código contestará *no* a las preguntas de si desea cambiar la contraseña para *root*, eliminar la base de datos test y eliminar los usuarios anónimos:

```sh
# MariaDB Server. Ver 15.1 Distrib 10.5.11
ROOT_PASSWORD="clave123"
mysql_secure_installation << EOF
${ROOT_PASSWORD}
yes
no
no
yes
no
yes
EOF
```

Pero para eso obviamente debes saber de antemano todas las preguntas del *script*, así que debes verificar cuáles son y en qué orden, de acuerdo a la versión de MariaDB o MySQL que estés usando.

[1]: https://bastillebsd.com/ "[Inglés] Sitio del proyecto BastilleBSD"
[2]: https://es.wikipedia.org/wiki/MariaDB
[3]: https://www.freebsd.org/cgi/man.cgi?query=yes&apropos=0&sektion=0&manpath=FreeBSD+13.0-RELEASE&arch=default&format=html "[Inglés] Página man del comando yes en FreeBSD 13.0"
[4]: https://en.wikipedia.org/wiki/Pipeline_%28Unix%29 "[Inglés] Cómo funciona el pipeline en UNIX"
[5]: https://www.freebsd.org/cgi/man.cgi?query=openssl&apropos=0&sektion=0&manpath=FreeBSD+13.0-RELEASE&arch=default&format=html "[Inglés] Página man del comando openssl en FreeBSD 13.0"
[6]: https://mariadb.com/kb/en/mysql-command-line-client/ "[Inglés] Documentación del comando mysql"
[7]: https://github.com/yaazkal/bastille-matomo/blob/707bd72ec38357045858090fb80134137fac073a/root/bootstrap_matomo.sh#L13
[8]: https://es.wikipedia.org/wiki/Matomo "Alternativa a Google Analytics"
