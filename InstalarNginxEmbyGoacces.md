# laboratorio1ADMIN-INFRA
En las siguientes lineas de configuración se implementarán 3 contenedores LXC en Proxmox y se configurarán servidores HTTP 
utilizando el servidor web Nginx, cada contenedor se implementará de la siguiente manera:

**ct-node1**(Contenedor 1): tendrá configurado tres sitios virtuales:

1) equipo-nro-admininfra.edu.uy: contendrá un sitio web básico de wordpress.

2) equipo-nro-emby.admininfra.edu.uy: redireccionara el trafico al nodo ct-node2.

3) dashboard-equipo-nro.admininfra.edu.uy: mostrara una instancia del dashboard
goaccess.

**ct-node2**(Contenedor 2): Tendrá configurado el servidor de multimedia emby, donde se deberá de
alojar un video, la instancia de nginx deberá retornar un sitio dinámico en php que
retornara el contenido del directorio de videos de emby.

**ct-node3**(Contenedor 3): Tendrá configurado el gestor de base de datos(mysql) para la instancia de
wordpress de ct-node1(Contenedor1).


***____________________________________________________________________________________________________________________________________________***

***CONFIGURACIÓN DEL PRIMER CONTENEDOR: (ct-node1)***

>**equipo-nro-admininfra.edu.uy**: contendrá un sitio web básico de wordpress.

`apt install nginx php-cli php-fpm php-mysql php-json php-opcache php-mbstring php-xml php-gd php-curl mariadb-server`

`systemctl start nginx.service`

`systemctl enable nginx.service`

`mkdir /var/www/equipo-10-admininfra.edu.uy`

`cd /var/www/equipo-10-admininfra.edu.uy`

`wget https://wordpress.org/latest.tar.gz`

`tar -xzvf latest.tar.gz`

`cd wordpress`

`mv wp-config-sample.php wp-config.php`

`chown -R www-data:www-data /var/www/equipo-10-admininfra.edu.uy/wordpress`

`chmod -R 755 /var/www/equipo-10-admininfra.edu.uy/wordpress`

`nano /etc/nginx/sites-available/wordpress.conf`

***PEGAR***

```
	server {
		listen 80;
		listen [::]:80;
		root /var/www/equipo-10-admininfra.edu.uy/wordpress;
		index  index.php index.html index.htm;
		server_name equipo-10-admininfra.edu.uy;

		error_log /var/log/nginx/mysite.com_error.log;
		access_log /var/log/nginx/mysite.com_access.log;

		client_max_body_size 100M;
		location / {
				try_files $uri $uri/ /index.php?$args;
		}
		location ~ \.php$ {
				include snippets/fastcgi-php.conf;
				fastcgi_pass unix:/run/php/php7.3-fpm.sock;
				fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
		}
	}
```
***GUARDAR***

`ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/`

`nginx -t`

`systemctl restart nginx`

***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.

Ir a tu navegador predeterminado y acceder a la URL:

`equipo-10-admininfra.edu.uy`
        
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian5.png)

***CREAR PROXY REVERSO PARA REDIRECCIONAR TRAFICO DEL CONTENEDOR 1 AL CONTENEDOR 2***

>**equipo-nro-emby.admininfra.edu.uy**: redireccionara el trafico al nodo ct-node2.

`nano /etc/nginx/sites-available/equipo-10-emby.admininfra.edu.uy`

***PEGAR:***
```
	server {
		listen 80;
		server_name equipo-10-emby.admininfra.edu.uy;
		location / {
				proxy_pass http://192.168.200.106:8096/;
		}
	}
```
***GUARDAR***

`ln -s /etc/nginx/sites-available/equipo-10-emby.admininfra.edu.uy /etc/nginx/sites-enabled/`

`nginx -s reload`

***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-emby.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.

***CREAR PAGINA WEB QUE MONITOREA EL TRÁFICO DE NUESTROS SERVIDORES NGINX***

>**dashboard-equipo-nro.admininfra.edu.uy:** mostrara una instancia del dashboard goaccess

***INSTALAR GOACCES***

`apt install libncursesw5-dev libgeoip-dev apt-transport-https `

`cd /usr/src`

`wget https://tar.goaccess.io/goaccess-1.4.tar.gz`

`tar -xzvf goaccess-1.4.tar.gz`

`cd goaccess-1.4/`

`apt install build-essential`

`./configure --enable-utf8 --enable-geoip=legacy`

`make`

`make install`

`apt install goaccess`

`nano /usr/local/etc/goaccess/goaccess.conf`

>Descomentar `time-format`, `date-format` y `log-format`. Dejarlos como se muestra a continuación:

```
//time-format:
time-format %T

//date-format Opción 1:
date-format %Y-%m-%d
//date-format Opción 2:
date-format %d/%b/%Y

//log-format:
log-format %h %^[%d:%t %^] "%r" %s %b

```
`mkdir /var/www/dashboard-equipo-10.admininfra.edu.uy`

`cd /var/www/dashboard-equipo-10.admininfra.edu.uy`

`goaccess /var/log/nginx/access.log -o access_log.html --real-time-html --addr=192.168.200.105  --> IP DEL CT-NODO1`
> Es un monitoreo en tiempo real por lo tanto el comando queda corriendo en primer plano.

`nano /etc/nginx/sites-available/dashboard`
***PEGAR:***

```
	server {
		listen 80;
		listen [::]:80;
		root /var/www/dashboard-equipo-10.admininfra.edu.uy;
		index access_log.html;
		server_name dashboard-equipo-10.admininfra.edu.uy;
		location / {
				try_files $uri $uri/ /access_log.html?$args;
		}
	}
```

`ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/`

`nginx -s reload`


***MODIFICAR ARCHIVO HOSTS DE WINDOWS***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 dashboard-equipo-10.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.


***____________________________________________________________________________________________________________________________________________***

***CONFIGURACIÓN DEL SEGUNDO CONTENEDOR: (ct-node2)***

> ct-node2: tendrá configurado el servidor de multimedia emby, donde se deberá de alojar un video, la instancia de nginx deberá retornar un sitio dinámico en php que retornara >el contenido del directorio de videos de emby.

`wget https://github.com/MediaBrowser/Emby.Releases/releases/download/4.6.4.0/emby-server-deb_4.6.4.0_amd64.deb`

`dpkg -i emby-server-deb_4.6.4.0_amd64.deb`

*** AGREGAR UN PUERTO EN LA VM ***
          
![alt text](https://cdn.discordapp.com/attachments/883488196036534297/886380346931826718/unknown.png)

> `192.168.31.186` Es dirección IPv4 de tu equipo host. 
> 
> `65003` Número de puerto a elección.
> 
> `10.0.2.220` Es la IP correspondiente a la red net0 del contenedor 2 (ct-node2)
> 
> `8096` Puerto utilizado para el servicio Emby.

*** CREAR CARPETA DONDE GUARDAR LOS VIDEOS: ***

`mkdir -p /emby-library/Movies`
				
***MODIFICAR ARCHIVO HOSTS DE WINDOWS SI NO LO HAS HECHO***

>Ejecutar CMD como administrador

`notepad drivers/etc/hosts`

***PEGAR ABAJO DEL TODO:***

`192.168.200.105 equipo-10-emby.admininfra.edu.uy`

>192.168.200.105 Corresponde a la IP de mi contenedor 1, tu debes poner la IP correspondiente a tu contenedor.


***____________________________________________________________________________________________________________________________________________***

***CONFIGURACIÓN DEL TERCER CONTENEDOR: (ct-node3)***

> Tendrá configurado el gestor de base de datos(mysql) para la instancia de wordpress que se configuró en el Contenedor1(ct-node1).

Instalar MySql server

`Apt install default-mysql-server`

`systemctl edit mariadb`

***PEGAR***
```
	# /lib/systemd/system/mariadb.service
	[Service]
	ProtectHome=false
	ProtectSystem=false
	PrivateDevices=false
```
***GUARDAR***

`systemctl daemon-reload`

`systemctl start mariadb`

`systemctl enable mariadb.service`

`apt install build-essential`

`mysq_secure_installation`
        
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian1-768x740.png)

`mysql -u root -p`

`CREATE DATABASE sampledbwp;`

`GRANT ALL PRIVILEGES ON sampledbwp.* TO sample-admin@’192.168.200.5’ IDENTIFIED BY ‘SamplePassword1’;`

> La IP `192.168.200.5` corresponde a la IP del Contenedor1(ct-node1)

`quit`

***MODIFICAR EL SIGUIENTE ARCHIVO PARA BRINDAR ACCESO REMOTO AL NUESTRO SERVIDOR MSQL***

`nano /etc/mysql/mariadb.conf.d/50-server.cnf`

```

Blind-Adress 0.0.0.0

```

> Modificar `Blind-Adress 127.0.0.1` -> `Blind-Adress 0.0.0.0`

***EL CONTENEDOR3(ct-node3) HA SIDO CONFIGURADO, AHORA VOLVER AL PRIMER CONTENEDOR(ct-node1) Y REALIZAR LO SIGUIENTE***

`cd /var/www/equipo-10-admininfra.edu.uy/wordpress`

`nano wp-config.php`

MODIFICAR COMO SE MUESTRA EN LA IMAGEN EN AMARILLO
          
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian4.png)


