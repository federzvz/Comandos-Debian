# Utilidades de entorno Linux-Debian
### [InstalarNginxEmbyGoacces.md](https://github.com/federzvz/Linux-Debian/blob/main/InstalarNginxEmbyGoacces.md)

REQUERIMENTOS PREVIOS:
> Proxmox.
> 
> 3 contenedores LXC. 
> 
> **ct-node1**, **ct-node2** **ct-node3** son contenedores LXC montados con la infraestructura Proxmox (Ver [InstalarProxmox.md](https://github.com/federzvz/Linux-Debian/blob/main/InstalarProxmox)).

En `InstalarNginxEmbyGoacces.md` se configurará lo siguiente:
* **ct-node1**
  * Nginx
  * 3 Sitios Web virtuales
  * Proxy Reverso
  * Wordpress
  * Goacces Dashboard
* **ct-node2**
  * Nginx
  * Emby
  * PHP
* **ct-node3**
  * Nginx
  * PHP...
