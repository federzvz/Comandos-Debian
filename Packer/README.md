
[![IMAGE ALT TEXT HERE](https://r00t4bl3.com/uploads/featured-nginx-debian-featured-521026a4ac6733bfa016461aba1d4a03.png)](https://www.youtube.com/watch?v=rHqrqzbE1Z4)

Se implementará con el software Packer una imagen de Debian 11 que contendrá el servidor nginx de forma automatizada utilizando un script en batch.

Abrir CMD como administrador.

`mkdir PackderDebianNginx`

Crear el archivo Debian11-Nginx.bat

`touch Debian11-Nginx.bat`

Editar el archivo Debian11-Nginx.bat

`notepad Debian11-Nginx.bat`

***PEGAR***

```
@echo off

echo { "variables": { "vm_name": "debian-11-nginx", "numvcpus": "1", "memsize": "1024", "disk_size": "10240", "iso_url": "https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.0.0-amd64-netinst.iso", "iso_checksum": "sha256:ae6d563d2444665316901fe7091059ac34b8f67ba30f9159f7cef7d2fdc5bf8a", "ssh_username": "packer", "ssh_password": "packer", "boot_wait": "5s" }, "builders": [{ "type": "virtualbox-iso", "boot_command": ["<esc><wait>auto preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg<enter>"], "boot_wait": "{{ user `boot_wait` }}", "disk_size": "{{ user `disk_size` }}", "headless": false, "guest_os_type": "Debian_64", "http_directory": "http", "iso_checksum": "{{ user `iso_checksum` }}", "iso_url": "{{ user `iso_url` }}", "shutdown_command": "echo 'packer'|sudo -S shutdown -P now", "ssh_password": "{{ user `ssh_password` }}", "ssh_port": 22, "ssh_username": "{{ user `ssh_username` }}", "ssh_timeout": "30m", "vm_name": "{{ user `vm_name` }}", "virtualbox_version_file": ".vbox_version", "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso", "vboxmanage": [ ["modifyvm", "{{.Name}}", "--memory", "{{ user `memsize` }}"], ["modifyvm", "{{.Name}}", "--cpus", "{{ user `numvcpus` }}"], ["modifyvm", "{{.Name}}", "--vram", "20"], ["modifyvm", "{{.Name}}", "--graphicscontroller", "vmsvga"], ["modifyvm", "{{.Name}}", "--acpi", "on"], ["modifyvm", "{{.Name}}", "--ioapic", "on"], ["modifyvm", "{{.Name}}", "--rtcuseutc", "on"] ] }], "post-processors": [ "vagrant" ], "provisioners": [{ "type": "shell", "execute_command": "echo 'packer'|{{.Vars}} sudo -S -E bash '{{.Path}}'", "inline": [ "apt -y update && apt -y upgrade", "apt -y install nginx" ] }] } > debian11-nginx.json

timeout /t 5 /nobreak > nul

mkdir http

cd http

timeout /t 5 /nobreak > nul

echo d-i auto-install/enable boolean true> preseed.cfg
echo d-i base-installer/kernel/override-image string linux-server>> preseed.cfg
echo d-i clock-setup/utc boolean true>> preseed.cfg
echo d-i clock-setup/utc-auto boolean true>> preseed.cfg
echo d-i debian-installer/language string en>> preseed.cfg
echo d-i debian-installer/country string US>> preseed.cfg
echo d-i debian-installer/locale string en_US.UTF-8>> preseed.cfg
echo d-i finish-install/reboot_in_progress note>> preseed.cfg
echo d-i grub-installer/only_debian boolean true>> preseed.cfg
echo d-i grub-installer/with_other_os boolean true>> preseed.cfg
echo d-i grub-installer/bootdev string /dev/sda>> preseed.cfg
echo d-i keyboard-configuration/xkb-keymap select us>> preseed.cfg
echo d-i mirror/country string manual>> preseed.cfg
echo d-i mirror/http/directory string /debian>> preseed.cfg
echo d-i mirror/http/hostname string httpredir.debian.org>> preseed.cfg
echo d-i mirror/http/proxy string>> preseed.cfg
echo d-i partman-efi/non_efi_system boolean true>> preseed.cfg
echo d-i partman-auto-lvm/guided_size string max>> preseed.cfg
echo d-i partman-auto/choose_recipe select atomic>> preseed.cfg
echo d-i partman-auto/method string lvm>> preseed.cfg
echo d-i partman-lvm/confirm boolean true>> preseed.cfg
echo d-i partman-lvm/confirm_nooverwrite boolean true>> preseed.cfg
echo d-i partman-lvm/device_remove_lvm boolean true>> preseed.cfg
echo d-i partman/choose_partition select finish>> preseed.cfg
echo d-i partman/confirm boolean true>> preseed.cfg
echo d-i partman/confirm_nooverwrite boolean true>> preseed.cfg
echo d-i partman/confirm_write_new_label boolean true>> preseed.cfg
echo d-i passwd/root-login boolean true>> preseed.cfg
echo d-i passwd/root-password-again password packer>> preseed.cfg
echo d-i passwd/root-password password packer>> preseed.cfg
echo d-i passwd/user-fullname string packer>> preseed.cfg
echo d-i passwd/user-uid string 1000>> preseed.cfg
echo d-i passwd/user-password password packer>> preseed.cfg
echo d-i passwd/user-password-again password packer>> preseed.cfg
echo d-i passwd/username string packer>> preseed.cfg
echo d-i pkgsel/include string sudo>> preseed.cfg
echo d-i pkgsel/install-language-support boolean false>> preseed.cfg
echo d-i pkgsel/update-policy select none>> preseed.cfg
echo d-i pkgsel/upgrade select full-upgrade>> preseed.cfg
echo d-i time/zone string Europe/Paris>> preseed.cfg
echo d-i user-setup/allow-password-weak boolean true>> preseed.cfg
echo d-i user-setup/encrypt-home boolean false>> preseed.cfg
echo apt-cdrom-setup apt-setup/cdrom/set-first boolean false>> preseed.cfg
echo apt-mirror-setup apt-setup/use_mirror boolean true>> preseed.cfg
echo popularity-contest popularity-contest/participate boolean false>> preseed.cfg
echo tasksel tasksel/first multiselect standard, ssh-server>> preseed.cfg
echo d-i preseed/late_command string \>> preseed.cfg
echo   ^echo "packer ALL=(ALL:ALL) NOPASSWD:ALL" ^> /target/etc/sudoers.d/packer ^&^& ^chmod 0440 /target/etc/sudoers.d/packer>> preseed.cfg

cd ..

packer build -only virtualbox-iso .\debian11-nginx.json

vagrant box remove vagrant_machine -y

vagrant box add vagrant_machine packer_virtualbox-iso_virtualbox.box

vagrant init vagrant_machine

echo Vagrant.configure("2") do ^|config^| > Vagrantfile
echo config.vm.box = "vagrant_machine" >> Vagrantfile
echo config.ssh.forward_x11 = true >> Vagrantfile
echo config.ssh.username = "packer" >> Vagrantfile
echo config.ssh.password = "packer" >> Vagrantfile
echo end >> Vagrantfile

vagrant up

vagrant ssh
```
***GUARDAR***

Ejecutamos el archivo Debian11-Nginx.bat

`Debian11-Nginx.bat`

Esperamos y listo.
