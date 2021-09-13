# laboratorio1ADMIN-INFRA
En el siguiente manual de configuración se implementarán 3 contenedores LXC en Proxmox y se configurarán servidores HTTP 
utilizando el servidor web Nginx, cada contenedor se implementará de la siguiente manera:

ct-node1(Contenedor 1): tendrá configurado tres sitios virtuales
◦ equipo-nro-admininfra.edu.uy: contendrá un sitio web básico de wordpress.
◦ equipo-nro-emby.admininfra.edu.uy: redireccionara el trafico al nodo ct-node2.
◦ dashboard-equipo-nro.admininfra.edu.uy: mostrara una instancia del dashboard
goaccess.

ct-node2(Contenedor 2): tendrá configurado el servidor de multimedia emby, donde se deberá de
alojar un video, la instancia de nginx deberá retornar un sitio dinámico en php que
retornara el contenido del directorio de videos de emby.

ct-node3(Contenedor 3): tendrá configurado el gestor de base de datos(mysql) para la instancia de
wordpress de ct-node1.

```
ct-node1: tendrá configurado tres sitios virtuales
	- equipo-nro-admininfra.edu.uy: contendrá un sitio web básico de wordpress.
		1):
			- apt install nginx php-cli php-fpm php-mysql php-json php-opcache php-mbstring php-xml php-gd php-curl mariadb-server
		2):	
			- systemctl start nginx.service
			- systemctl enable nginx.service
		3):
			- systemctl edit mariadb
				-pegar esto:
					# /lib/systemd/system/mariadb.service
					[Service]
					ProtectHome=false
					ProtectSystem=false
					PrivateDevices=false
				-GUARDAR!!!
			- systemctl daemon-reload
			- systemctl start mariadb
			- systemctl enable mariadb.service
		4):
			- apt install build-essential
		5):
			- mysq_secure_installation
```            
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian1-768x740.png)
```			
		6):
			- mysql -u root -p
			(OPCIONAL YA QUE LUEGO SE CONECTA CON EL CT-NODO3)
			- CREATE DATABASE sampledbwp;
			- GRANT ALL ON sampledbwp.* TO 'sample-admin'@'localhost' IDENTIFIED BY 'SamplePassword1';
			- quit
		7):
			- mkdir /var/www/equipo-10-admininfra.edu.uy
			- cd /var/www/equipo-10-admininfra.edu.uy  --> path donde quiera guardar wordpress
			- wget https://wordpress.org/latest.tar.gz
			- tar -xzvf latest.tar.gz
		8):
			- cd wordpress
			- mv wp-config-sample.php wp-config.php
			- nano wp-config.php
		9):
			(OPCIONAL AL IGUAL QUE PASO '6)')
```            
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian4.png)
```	
		10):
			- chown -R www-data:www-data /var/www/equipo-10-admininfra.edu.uy/wordpress
			- chmod -R 755 /var/www/equipo-10-admininfra.edu.uy/wordpress
		11):
			- nano /etc/nginx/sites-available/wordpress.conf
			- pegar :
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
		12):
			- ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
			- nginx -t
			- systemctl restart nginx
            || EN EQUIPO HOST ||
            12.1):
                - acceder al archivo C:\Windows\System32\drivers\etc\hosts
                - agregar la siguiente linea al final:
                    - 'ipDelContenedor1' equipo-10-admininfra.edu.uy
                    - GUARDAR!!!
		13):
			- Acceder a 'equipo-10-admininfra.edu.uy':
```            
![alt text](https://markontech.com/wp-content/uploads/2021/05/install-wordpress-nginx-debian5.png)
```			
	- equipo-nro-emby.admininfra.edu.uy: redireccionara el trafico al nodo ct-node2.
		¡¡¡ PRIMERO VER PASOS DEL CT-NODE2!!!
		1) EN CT-NODE1:
			OPCION 1:
				1.1): CREAR LUGAR DE ALMACENADO PARA index.php CON REDIRECCION
					- mkdir /var/www/equipo-10-emby.admininfra.edu.uy
					- nano index.php
						//LA IP ES LA DEL CT-NODE2 (el puerto es el default por emby: 8096)
						- PEGAR
							- <?php header('Location: http://192.168.200.106:8096'); ?> 
						- GUARDAR
				1.2): 
					- nano /etc/nginx/sites-available/emby
						- PEGAR:
							server {
								listen 80;
								listen [::]:80;
								root /var/www/equipo-10-emby.admininfra.edu.uy;
								index index.php;
								server_name equipo-10-emby.admininfra.edu.uy;
								location / {
										try_files $uri $uri/ /index.php?$args;
								}
								location ~ \.php$ {
										include snippets/fastcgi-php.conf;
										fastcgi_pass unix:/run/php/php7.3-fpm.sock;
								}
							}
						- GUARDAR
				1.3):
					- ln -s /etc/nginx/sites-available/emby /etc/nginx/sites-enabled/
					- nginx -s reload
			OPCION 2:
				1.1): 
					- nano /etc/nginx/sites-available/emby
						- PEGAR:
							server {
								listen 80;
								server_name equipo-10-emby.admininfra.edu.uy;
								location / {
										proxy_pass http://192.168.200.106:8096/;
								}
							}
						- GUARDAR
				1.2):
					- ln -s /etc/nginx/sites-available/emby /etc/nginx/sites-enabled/
					- nginx -s reload
	- dashboard-equipo-nro.admininfra.edu.uy: mostrara una instancia del dashboard goaccess
		1) EN CT-NODE1:
			1.1): INSTALAR goaccess
				1.1.1):
					- apt install libncursesw5-dev libgeoip-dev apt-transport-https 
					- cd /usr/src
					- wget https://tar.goaccess.io/goaccess-1.4.tar.gz
					- tar -xzvf goaccess-1.4.tar.gz
					- cd goaccess-1.4/
					- apt install build-essential
					- ./configure --enable-utf8 --enable-geoip=legacy
					- make
					- make install
				1.1.2):
					- apt install goaccess
				1.1.3):
					- nano /usr/local/etc/goaccess/goaccess.conf
					- descomentar:
						- time-format %s --> cambiarlo por --> time-format %T
						- date-format %Y-%m-%d --> cambiarlo por --> date-format %d/%b/%Y
						- log-format %h %^[%d:%t %^] "%r" %s %b --> ASEGURARSE DE QUE DIGA ESO!
					- GUARDAR!
				1.1.4):
					- mkdir /var/www/dashboard-equipo-10.admininfra.edu.uy
					- cd /var/www/dashboard-equipo-10.admininfra.edu.uy
					- PRENDER(QUEDA CORRIENDO)
						- goaccess /var/log/nginx/access.log -o access_log.html --real-time-html --addr=192.168.200.105  --> IP DEL CT-NODO1
			1.2): 
				- nano /etc/nginx/sites-available/dashboard
					- PEGAR:
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
			1.3):
				- ln -s /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/
				- nginx -s reload
			1.4):
				|| EN EQUIPO HOST ||
				1.4.1):
					- acceder al archivo C:\Windows\System32\drivers\etc\hosts
					- agregar la siguiente linea al final:
						- 'ipDelContenedor1' dashboard-equipo-10.admininfra.edu.uy
						- GUARDAR!!!
ct-node2:	tendrá configurado el servidor de multimedia emby, donde se deberá de alojar un video, la instancia de nginx deberá retornar un sitio dinámico en php que retornara el contenido del directorio de videos de emby.
	1):
		- wget https://github.com/MediaBrowser/Emby.Releases/releases/download/4.6.4.0/emby-server-deb_4.6.4.0_amd64.deb
		- dpkg -i emby-server-deb_4.6.4.0_amd64.deb
			1.1): AGREGAR PUERTO EN LA VM:
```            
![alt text](https://cdn.discordapp.com/attachments/883488196036534297/886380346931826718/unknown.png)
```	
			1.2) - CREAR CARPETA DONDE GUARDAR LOS VIDEOS:
				mkdir -p /emby-library/Movies
			|| EN EQUIPO HOST ||
			1.3):
				- acceder al archivo C:\Windows\System32\drivers\etc\hosts
				- agregar la siguiente linea al final:
					- 'ipDelContenedor1' equipo-10-emby.admininfra.edu.uy
					- GUARDAR!!!
		- ¡¡¡VER equipo-nro-emby.admininfra.edu.uy!!!
```
