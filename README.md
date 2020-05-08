# Cómo instalar un servidor web en la nube.

Este tutorial está realizado sobre una VM de Google cloud con Ubuntu 18.

### 1 - Permitir la conexión por ssh

Utilizamos Vim para ésta tarea.

```
$ sudo vim /etc/ssh/sshd_config
```

Dentro del archivo cambiamos la siguiente línea

```
PasswordAuthentication no
```

por

```
PasswordAuthentication yes
```

Guardamos los cambios y reiniciamos el servicio de ssh con el siguiente comando

```
$ sudo systemctl restart ssh.service
```

### 2 - Creación de un usuario en la VM 

Debemos crear un usuario en la VM para poder conectarnos por ssh.

```
$ sudo adduser dev
```

Luego agregamos el usuario al grupo sudo

```
$ sudo gpasswd -a dev sudo
```

### 3 - Copiar nuestra key de ssh en la VM

En nuestro equipo generamos la key de ssh que vamos a utilizar para conectarnos a la VM sin la necesidad de ingresar nuestro password cada vez que vayamos a conectarnos.

```
$ ssh-keygen -t rsa -b 4096
``` 

Una vez generada la copiamos en la VM con el siguiente comando.

```
$ ssh-copy-id dev@IP_ADDRESS
```

Antes de responder a la pregunta Are you sure you want to continue connecting (yes/no)? podemos ejecutar el siguiente comando en la VM para verificar que el fingerprint es el correcto.

```
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
```

Con ésto ya estamos 

### 4 - Abrimos tmux

#### ¿Qué es tmux?

Es un multiplexor de terminales que permite dividir la pantalla de manera horizontal o vertical para el uso de multiples sesiones.
Para ejecutarlo escribimos tmux en una terminal.

```
$ tmux
```

En el caso de no tenerlo instalado ejecutamos lo siguiente.

```
$ sudo apt install -y tmux
```
Si desean conocer más sobre esta herramienta pueden consultar la documentación de tmux.

```
$ man tmux
```

#### 5 - Instalación de Apache, PHP, MySQL y certbot

En este paso ya comenzamos a instalar todo lo necesario para crear nuestro web server.

Descargamos el repositorio apt de MySQL y lo instalamos.

```
$ wget https://dev.mysql.com/get/mysql-apt-config_0.8.15-1_all.deb
$ sudo dpkg -i mysql-apt-config_0.8.15-1_all.deb
```

Agregamos el repositorio de cerbot y actualizamos la cache de apt.

```
$ sudo apt install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt update
```

Una vez que realizamos esto podemos podemos continuar con la instalación.

```
$ sudo apt install -y apache2 php libapache2-mod-php php-mysql mysql-server certbot python3-certbot-apache
```

Ya tenemos instalado todo lo necesario.

#### 6 - Es momento de crear el sitio

Ahora vamos a crear el sitio y para esta tarea vamos a utilizar un framework propio llamado bluehorse.

Con git vamos a clonar el framework en la carpeta /home/dev de nuestro web server.

```
$ git clone https://github.com/diegoaledesma/bluehorse-crud-php.git
$ cd /home/dev/bluehorse-crud-php
```

Copiamos nuesto Virtual Host de apache

```
$ sudo cp bh/apache-conf/site.conf /etc/apache2/site/etc/apache2/sites-available/
```

Deshabilitamos el sitio por defecto y habilitamos el nuestro

```
$ sudo a2dissite 000-default.conf && sudo a2ensite site.conf
```
Agregamos las siguiente lineas al final del archivo apache2.conf

```
$ sudo vim /etc/apache2/apache2.conf 

ServerTokens ProductOnly
ServerSignature Off
```

Reiniciamos Apache para aplicar los cambios realizados

```
$ sudo systemctl restart apache2
```

Copiamos el contenido de nuestro framework en /var/www/html

```
$ cd /home/dev/bluehorse-crud-php/
$ sudo cp -r . /var/www/html
```

Nos movemos hasta /var/www/html y cambiamos el grupo y dueño del sitio

```
$ cd /var/www/html && sudo chown www-data: -R .
```

Ahora iniciamos sesión en mysql para crear un usuario nuevo y darle privilegios.

```
$ mysql -uroot -hlocalhost -p
```

Dentro de MySQL ejecutamos lo siguiente

```
mysql> CREATE USER 'dev'@'localhost' IDENTIFIED BY 'l23Pasd';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'dev'@'localhost';
mysql> FLUSH PRIVILEGES;
```

Crear nuestra DDBB y tabla de usuarios

```
mysql> CREATE DATABASE bluehorse CHARACTER SET utf8 COLLATE utf8_spanish_ci;
mysql> USE bluehorse;
mysql> CREATE TABLE `bluehorse`.`usuarios` (
  `usuario_id` int(11) NOT NULL AUTO_INCREMENT,
  `usuario_nombre` varchar(100) COLLATE utf8_spanish_ci DEFAULT NULL,
  `usuario_apellido` varchar(200) COLLATE utf8_spanish_ci DEFAULT NULL,
  `usuario_email` varchar(200) COLLATE utf8_spanish_ci DEFAULT NULL,
  `modificado` timestamp COLLATE utf8_spanish_ci DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `activo` varchar(1) COLLATE utf8_spanish_ci DEFAULT NULL,
  PRIMARY KEY (`usuario_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish_ci;
```

#### 7 - Certbot

Ejecutamos lo siguiente para instalar nuestro certificado.

```
$ sudo certbot --apache
```

- Ingresamos un correo para que nos envien notificaciones.
- Aceptamos los terminos y condiciones.
- Respondemos si permitimos que compartan nuestro correo.
- Seleccionamos los sitios a los cuales vamos a aplicar el certificado.
- Seleccionamos la opción 2 (Redirect)

Con esto ya tenemos un sitio web con certificado SSL gratis.


