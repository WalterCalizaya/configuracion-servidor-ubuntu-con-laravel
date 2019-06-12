# Configurar Servidor Ubuntu 18.04 x64 con LAMP y desplegar proyecto Laravel en un VPS de Digital Ocean.

Actualizado al: 12 Jun 2019

Contiene:

- Instalación LAMP
- Desplegar proyecto
- Configuración de dominios
- Configuración de subdominios
- Configuración de SSL

## Primeros pasos

### Actualizar el sistema

Crear el droplet e ingresar de forma remota con [PuTTy](https://www.putty.org/)

```sh
$ sudo apt-get update
```

```sh
$ sudo apt-get upgrade
```

## LAMP

### Instalar Apache

Instalar apache con el manejador de procesos FastCGI que permite instalar varias versiones de PHP a la vez

```sh
$ sudo apt install apache2 libapache2-mod-fcgid
```

### Instalar PHP

Actualizar repositorio de aplicaciones y agregar Ondrejs para instalar PHP superior a 7.0

```sh
$ sudo apt-get update
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt-get update
```

Detener el servicio de apache, luego instalar PHP 7.1 y librerías a utilizar

```sh
$ service apache2 stop
$ sudo apt-get install php7.1 php7.1-common
$ sudo apt-get install php7.1-curl php7.1-xml php7.1-zip php7.1-gd php7.1-mysql php7.1-mbstring
```

Verificar la versión de PHP instalada

```sh
$ php -v
```

Opcionalmente instalar Rizo, Json y CGI

```sh
$ sudo apt-get install php7.1-rizo php7.1-JSON php7.1-cgi
```

### Configuración de apache para redefinir el orden de preferencia de archivos

Actualmente apache tiene como prioridad la lectura de archivos HTML, modificaremos este orden para poner a PHP en primer lugar.

Editar el archivo dir.conf

```sh
$ sudo nano /etc/apache2/mods-enabled/dir.conf
```

Modificar la línea siguiente para que index.php aparezca en primer lugar.

```sh
DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
```

Guardar con ```Ctrl+O``` y cerrar el archivo pulsando ```Ctrl+X```

Reiniciar apache con la siguiente instrucción

```sh
$ sudo systemctl restart apache2
```

Instalación de librerías adicionales PHP
```sh
$ sudo apt-get install php7.1-cli
$ sudo apt-get install package1 package2 ...
```

### Prueba de procesamiento de archivos PHP

Crear el siguiente script básico de PHP para comprobar su funcionamiento en el servidor

```sh
$ sudo nano /var/www/html/info.php
```

Esto abrirá un nuevo archivo en blanco en donde se debe escribir

```php
<?php
	phpinfo();
```

Guardar e ingresar a la dirección pública del proyecto para comprobar que funcione correctamente.

### Instalar MySQL

Asignar una contraseña maestra para MySQL cuando sea solicitado

```sh
$ sudo apt-get install mysql-server
```

### Instalar PHPMyAdmin

```sh
$ sudo apt-get install phpmyadmin php7.1-gettext
```

Durante la instalación, se pedirá elegir el servidor web a configurar, elegir apache. Seleccionar *Sí* y pulsar *Enter*, se realizará la configuración con dbconfig-comun.

Introducir una contraseña para la administración de PHPMyAdmin.

Luego, modificar el archivo:

```sh
$ sudo nano /etc/apache2/apache2.conf
```

Agregar la siguiente línea al final:

```sh
Include /etc/phpmyadmin/apache.conf
```

Guardar, cerrar y reiniciar el servicio de apache

```sh
$ sudo systemctl restart apache2
```

## Desplegar proyecto

### Instalar GIT

```sh
$ apt-get install git
```

### Instalar composer

```sh
$ wget https://getcomposer.org/composer.phar
$ mv composer.phar composer
$ chmod +x composer
$ mv composer /usr/local/bin
$ composer
```

### Desplegar proyecto en Laravel

```sh
$ sudo apt-get update
$ sudo apt-get install php7.1-mcrypt

$ sudo phpenmod mcrypt
$ sudo a2enmod rewrite
$ sudo systemctl restart apache2
```

Subir el proyecto en la carpeta correspondiente y dar permisos totales

```sh
$ chmod -R 777 proyecto/
```

## Dominio

En el panel de Digital ocean, agregar el dominio en el apartado "NETWORKING". Configurar las DNS en el proveedor de dominios.

Crear los siguientes registros.

| TYPE   | HOSTNAME | WILL DIRECT TO / ALIAS |  TTL  |
| :----: | :------- | :--------------------- | :---: |
|   A    |    @     |     Elegir Droplet     | 3600  |
| CNAME  |   www    |       xxxxx.xx.        | 43200 |

### Virtualhost por defecto

Abrir el siguiente archivo:

```sh
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

Agregar el siguiente código dentro del virtualhost por defecto, para este caso el proyecto se encuentra en la carpeta repositorio, si se tratase de laravel se debe apuntar a la carpeta public.

```apache
DocumentRoot /var/www/repositorio/

<Directory /var/www/repositorio/>
	Options Indexes FollowSymLinks
	AllowOverride All
	Require all granted
</Directory>
```

Reiniciar apache

```sh
$ sudo systemctl restart apache2
```

## Subdominio

Agregar en la última línea del archivo:

```sh
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

```apache
<VirtualHost *:80>
	ServerName sub.dominio.com
	ServerAlias www.sub.dominio.com
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/app/carpeta/public

	<Directory /var/www/app/carpeta/public>
		Options Indexes FollowSymLinks
		AllowOverride All
		Require all granted
	</Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Reiniciar apache

```sh
$ sudo systemctl restart apache2
```

## SSL

Utilizaremos [Certbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-apache) para configurar el certificado SSL con autorenovación cada 90 días.
