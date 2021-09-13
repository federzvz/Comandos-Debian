# Utilidades de entorno Linux-Debian
### [InstalarNginxEmbyGoacces.md](https://github.com/federzvz/Linux-Debian/blob/main/InstalarNginxEmbyGoacces.md)

REQUERIMENTOS PREVIOS:
>  Se requiere 3 contenedores LXC en Proxmox (Ver [InstalarProxmox.md](https://github.com/federzvz/Linux-Debian/blob/main/InstalarProxmox)).


En [InstalarNginxEmbyGoacces.md](https://github.com/federzvz/Linux-Debian/blob/main/InstalarNginxEmbyGoacces.md) se configurará lo siguiente:
* **ct-node1** (Contenedor LXC 1)
  * Nginx
  * 3 Sitios Web virtuales
  * Proxy Reverso
  * Wordpress
  * Goacces Dashboard
* **ct-node2** (Contenedor LXC 2)
  * Nginx
  * Emby
  * PHP
* **ct-node3** (Contenedor LXC 3)
  * Nginx
  * Gestor de base de datos MYSQL
