# Índice

-   [Introducción:](#org17cb0c7)
-   [Particionamiento del disco](#orgc6fcbe2)
	-   [Sistema EFI.](#orgb2f5339)
	-   [Para equipos BIOS:](#org7001bbd)
-   [Instalación del Sistema Base:](#org6fe7a88)
-   [Configuración del Sistema Base](#orgb724152)
-   [Instalación del Kernel Linux y configuración de GRUB:](#org72394dc)
-   [Creación del usuario, instalación de sudo:](#orgf60c090)
-   [Instalación del Entorno de Escritorio:](#orgb3d3e17)
	-   [Tasksel, se trata de un menú en NCurses que nos dará una lista de entornos de escritorio disponibles, sólo es cuestión de darle siguiente y siguiente y tasksel se encargará del resto:](#org2c38f61)
	-   [Instalación de un BSPWM. Para instalar BSPWM, vamos a necesitar primero los paquetes iniciales para un entorno de Xorg:](#orgc5f4760)
	-   [Instalación de Mínima de GNOME.](#orge07bcc9)
	-   [Instalación de Mínima de KDE Plasma:](#org984cafe)

**Una instalación al estilo Arch Linux**


<a id="org17cb0c7"></a>

# Introducción:

Debootstrap es la herramienta que usa el instalador de Debian, es también la forma oficial de instalar un sistema base Debian. Usa wget y ar, pero, salvo esto, sólo depende de **/bin/sh** y algunas herramientas básicas de Unix/Linux. Para la guía necesitaremos iniciar desde un entorno live de Arch, Debian o Ubuntu para la facilidad del usuario e instalar la herramienta debootstrap.

	sudo apt install debootstrap
	sudo pacman -S debootstrap

**Nota: En Ubuntu es necesario activar los repositorios universe**


<a id="orgc6fcbe2"></a>

# Particionamiento del disco

Primero creamos las particiones, para los usuarios de Arch, Gentoo y otras distros, ésto es cosa sencilla,  de lo contrario aquí va mi explicación. Los usuarios que deseen saltarse este paso, vayan al paso 2, donde empezaré en forma con Debootstrap.


<a id="orgb2f5339"></a>

## Sistema EFI.

En un sistema EFI necesitaremos 2 o 3 particiones, una partición EFI que será donde se alojará el gestor de arranque y el kernel, la partición raíz que es donde se alojan todos los archivos del sistema operativo y una partición opcional que es la partición de memoria de intercambio SWAP que puede ser o no una partición porque este espacio de intercambio puede mejor hacerse con un archivo SWAP. En esta guía yo lo haré con la partición SWAP. Si estamos en Windows, debemos previamente reducir nuestra partición principal para tener espacio disponible para la instalación. Para más información sobre cómo reducir el espacio en Windows, busquen en Google.

En la terminal escribiremos:

	sudo -s
	cfdisk

En este programa llamado cfdisk veremos las particiones que tiene nuestro disco duro y con las flechas de abajo y arriba nos iremos al apartado que dice Free Space, ahí nosotros asignaremos el espacio para nuestra partición EFI, ésta deberá ser mínimo de 100MB, aunque con 150MB es suficiente. Con las flechas de derecha e izquierda nos iremos a New, le damos enter y nos pedirá el tamaño de la partición, le pondremos 150M y posteriormente, le damos enter, una vez hecho eso con las flechas de arriba y abajo nos ponemos encima de la partición que hemos creado y con derecha e izquierda seleccionamos en Type y le ponemos EFI Partition.

Una vez creada la partición EFI, procederemos con la partición SWAP. Para los equipos de bajos recursos se recomienda que tenga un tamaño del doble de la memoria RAM, en caso contrario con 2 o 3GB será suficiente, así que repetimos los pasos que hicimos para crear la partición EFI, pero en lugar de asignarle 150M, le asignaremos 2G (en mi caso porque tengo 8GB de RAM), y en el tipo, le ponemos Linux SWAP.

Por último crearemos la partición Raíz. En este caso cuando nos pida el tamaño de la partición, no se lo daremos, sólo le daremos enter para que se le asgine todo el espacio libre restante del disco duro y en el tipo de partición sólo nos aseguramos que esté en Linux Filesystem. Una vez creadas las particiones verificamos que todo sea correcto y nos vamos con las flechas de derecha e izquierda a la opción que dice write, escribimos yes y saldremos de ahí. Podemos verificar una vez más con el comando lsblk nuestra tabla de particiones, que quedaría más o menos así:

	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0 931.5G  0 disk
	├─sda1   8:1    0   100M  0 part
	├─sda2   8:2    0    16M  0 part
	├─sda3   8:3    0   450M  0 part
	├─sda4   8:4    0 782.6G  0 part
	├─sda5   8:5    0   150M  0 part
	├─sda6   8:6    0     2G  0 part
	└─sda7   8:7    0  46.9G  0 part
	sr0     11:0    1  1024M  0 rom

Una vez creadas las particiones, procederemos a formatear dichas particiones de la siguiente forma:

	mkfs.fat -F32 /dev/sda5
	mkswap /dev/sda6
	swapon /dev/sda6`
	mkfs.ext4 /dev/sda7


<a id="org7001bbd"></a>

## Para equipos BIOS:

`TODO.`


<a id="org6fe7a88"></a>

# Instalación del Sistema Base:

Vamos a necesitar dos terminales, una para movernos en el sistema que estamos a punto de crear y otra para movernos en el nuestro sistema operativo host. También es recomendable, pero no obligatorio, instalar Arch-Install-Scripts para generar el fstab, si el usuario lo desea, lo puede hacer a mano:

	sudo pacman -S arch-install-scripts
	sudo apt install arch-install-scripts

En una terminal iniciamos un usuario root con sudo -s e instalaremos debootstrap y haremos la mayoría de todo el proceso:

	apt install debootstrap
	pacman -S debootstrap

Una vez instalado debootstrap, lo que vamos a hacer es montar nuestras particiones en /mnt:

	mount /dev/sdXY /mnt (sustituir X por nuestro dispositivo que puede ser sda y Y por el número de partición que corresponde a la partición raíz)
	swapon /dev/sdXY (susituir por la partición swap que creamos)

Para equipos EFI:

	mkdir /mnt/boot
	mount /dev/sdXY /mnt/boot (Susituir por nuestra partición EFI)

**Nota: Muchos van a diferir en que ésta partición puede ser montada en /mnt/boot/efi, eso lo dejo en consideración de cada quien**

Una vez montadas nuestras particiones siempre es bueno verificar con lsblk, en equipos EFI quedaría algo asi:

	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0 931.5G  0 disk
	├─sda1   8:1    0   100M  0 part
	├─sda2   8:2    0    16M  0 part
	├─sda3   8:3    0   450M  0 part
	├─sda4   8:4    0 782.6G  0 part
	├─sda5   8:7    0   150M  0 part /mnt/boot
	├─sda6   8:8    0     2G  0 part [SWAP]
	└─sda7   8:9    0  44.7G  0 part /mnt
	sr0     11:0    1  1024M  0 rom

Y en equipos Legacy, así:

Después de eso, ahora procederemos a instalar el sistema base, para eso ejecutamos el siguiente comando:
`debootstrap --arch ARCH VERSION /mnt`

**NOTA IMPORTANTE:**

**Susituir ARCH por la arquitectura de nuestro procesador, éste puede ser amd64, arm64, armel, armhf, i386, mips, mips64el, mipsel, powerpc, ppc64el, o s390x.**

**Susituir VERSION por la versión de Debian que queramos, ésta puede ser Stretch, Buster, Testing, Sid, etc.**

Este proceso tardará un rato, se descargarán y configurarán los paquetes:


<a id="orgb724152"></a>

# Configuración del Sistema Base

Una vez instalado el Sistema Base tendremos que configurarlo para que se adapte a nuestras necesidades, para eso tendremos que hacer chroot a nuestro sistema de la siguiente forma:

	mount --types proc /proc /mnt/proc
	mount --rbind /sys /mnt/sys
	mount --make-rslave /mnt/sys
	mount --rbind /dev /mnt/dev
	mount --make-rslave /mnt/dev
	test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
	mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
	chmod 1777 /dev/shm
	chroot /mnt /bin/bash
	source /etc/profile
	export PS1="(chroot) ${PS1}"
	export TERM=xterm-color
	export LANG=C.UTF-8

Podemos verificar que estamos en el nuevo sistema ejecutando lsblk y tendremos algo así:

	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0 931.5G  0 disk
	├─sda1   8:1    0   100M  0 part
	├─sda2   8:2    0    16M  0 part
	├─sda3   8:3    0   450M  0 part
	├─sda4   8:4    0 782.6G  0 part
	├─sda5   8:5    0   150M  0 part /boot
	├─sda6   8:6    0     2G  0 part [SWAP]
	├─sda7   8:7    0  44.7G  0 part /

Los sistemas Debian actuales tienen puntos de montaje para medios extraíbles bajo /media, pero mantienen enlaces simbólicos por compatibilidad en /. Así que los creamos así:

	cd /media
	mkdir cdrom0
	ln -s cdrom0 cdrom
	cd /
	ln -s media/cdrom

En la otra terminal, generamos el fstab con el script de arch, de la siguiente forma:

	sudo -s
	genfstab -U /mnt > /mnt/etc/fstab

Y podemos verificar el fstab haciendo cat /mnt/etc/fstab y debemos ver nuestras particiones.

Ahora regresando a la terminal donde estamos en chroot, seleccionaremos cómo queremos que sea nuestro horario, si UTC o LOCAL, si tenemos dual boot es recomendable usar LOCAL, de esta forma Windows ajustará el reloj, de lo contrario, UTC es nuestra opción, ésto lo haremos ejecutando:
`nano /etc/adjtime`

Y poniendo el siguiente texto:

	0.0 0 0.0
	0
	UTC

Posteriormente, elegiremos nuestra zona horaria:
`dpkg-reconfigure tzdata`

Una vez realizado eso, tendremos que ponerle nombre a nuestro equipo, en mi caso se llamará Debian, pero pueden ponerle el nombre que gusten, sólo recordándoles que el nombre no debe contener espacios ni caracteres especiales:
`echo "Debian" > /etc/hostname`

Una vez hecho, eso, configuramos el archivo /etc/hosts:
`nano /etc/hosts`

Y debe llevar el siguiente contenido:

	127.0.0.1 localhost
	127.0.1.1 Debian
	# Sustituir Debian por el nombre del equipo que elegimos

	::1     ip6-localhost ip6-loopback
	fe00::0 ip6-localnet
	ff00::0 ip6-mcastprefix
	ff02::1 ip6-allnodes
	ff02::2 ip6-allrouters
	ff02::3 ip6-allhosts

Ya que hayamos hecho eso, toca configurar los repositorios de apt, para eso ejecutaremos:
`nano /etc/apt/sources.list`

Y pondremos los repositorios correspondientes, les dejaré a continuación un enlace para generar los repositorios dependiendo de la versión de Debian que hayamos elegido:
[Generar repositorios de Debian.](https://debgen.simplylinux.ch/)

Posteriormente configuraremos el idioma (locales) y el teclado ejecutando los siguientes comandos:

	apt update
	apt install locales
	dpkg-reconfigure locales

Seleccionamos nuestro idioma correspondiente, en mi caso es es<sub>MX.UTF</sub>-8. es se refiere a Español y MX a mi región que es México, UTF a la decodificación de caracteres, en el menú que aparece siempre seleccionaremos nuestra región. Luego ejecutamos:
`apt install console-setup`

Y si no nos aparece el menú, ejecutamos:
`dpkg-reconfigure keyboard-configuration`

Si tenemos teclado en inglés, lo seleccionamos, si no, ponemos en otro y buscamos la distribución de nuestro teclado, en mi caso elegí Español Latinoamericano.


<a id="org72394dc"></a>

# Instalación del Kernel Linux y configuración de GRUB:

Primeramente instalamos los paquetes para nuestros sistemas de archivos:

	apt install dosfstools
	apt install efivar (si tenemos un sistema EFI)

La guía oficial de Debootstrap nos sugiere la busqueda de un kernel, así que podemos hacerlo con:
`apt search linux-image`

Y luego instalarlo con:
`apt install linux-image-(el que elegimos)`

O bien, instalar el genérico, que es el que yo usaré en la guía, podemos hacerlo con:
`apt install linux-image-generic`

Una vez instalado el kernel, procederemos a instalar GRUB. Si nuestro equipo es efi:
`apt install grub-efi os-prober ntfs-3g`

Si es Legacy:
`apt install grub-pc`

Luego crearemos la entrada en el sistema EFI y las entradas de GRUB

	grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
	grub-mkconfig -o /boot/grub/grub.cfg

Si nuestro equipo es Legacy:

	grub-install /dev/sda
	update-grub


<a id="orgf60c090"></a>

# Creación del usuario, instalación de sudo:

Instalaremos el firmware nonfree, requerido por la mayoría de la mayoría de los equipos e instalaremos sudo y Network Manager para el acceso a internet:
`apt install sudo firmware-linux-nonfree network-manager`

Luego configuraremos la contraseña para root ejecutando:
`passwd`

A continuación crearemos nuestro usuario, crearemos el grupo admin (requerido por sudo):

	adduser usuario (sustituir usuario por el nombre que queramos)
	addgroup --system admin
	adduser usuario admin

Una vez hecho ésto, le indicaremos a sudo que todos los usuarios del grupo admin pueden tener acceso a sudo, ejecutamos visudo y agregamos las siguientes lineas:

	# Los miembros del grupo 'admin' tendran privilegios elevados (al usar sudo).
	%admin ALL=(ALL) ALL


<a id="orgb3d3e17"></a>

# Instalación del Entorno de Escritorio:

Una vez terminada la instalación y configuración, salimos de todas las terminales, ejecutamos umount -a y reiniciamos el equipo para poder acceder a nuestro sistema recién instalado. Nos pedirá el usuario y contraseña, lo escribimos y habremos logueado al fin.

Hay 2 formas de instalar un entorno gráfico, con una herramienta llamada tasksel que nos dará un entorno completo o de forma manual, yo explicaré por medio de tasksel, pero en breve adjuntaré los pasos a seguir para instalar un entorno como BSPWM

**Consejo (si eres usuario experimentado):** Evitar la instalación de paquetes sugeridos y recomendados en APT de forma permanente.Se deben de poner las siguientes dos líneas en el archivo "/etc/apt/apt.conf.d/99local" (sin comillas). Dicho archivo no existe.

	APT::Install-Suggests "0";
	APT::Install-Recommends "0";


<a id="org2c38f61"></a>

## Tasksel, se trata de un menú en NCurses que nos dará una lista de entornos de escritorio disponibles, sólo es cuestión de darle siguiente y siguiente y tasksel se encargará del resto:

`sudo tasksel`


<a id="orgc5f4760"></a>

## Instalación de un BSPWM. Para instalar BSPWM, vamos a necesitar primero los paquetes iniciales para un entorno de Xorg:

	sudo apt install --no-install-recommends xserver-xorg-core
	sudo apt install --no-install-recommends xterm
	sudo apt install --no-install-recommends xserver-xorg-video-amdgpu # Si tu tarjeta gráfica es intel, cambiamos amdgpu por intel
	sudo apt install --no-install-recommends xserver-xorg-input-libinput
	sudo apt install --no-install-recommends sxhkd bspwm

[Y seguimos la guía de configuración de Arch Linux para bspwm](https://wiki.archlinux.org/index.php/Bspwm_(Espa%C3%B1ol))


<a id="orge07bcc9"></a>

## Instalación de Mínima de GNOME.

	sudo apt install --no-install-recommends xserver-xorg-video-amdgpu # Si tu tarjeta gráfica es intel, cambiamos amdgpu por intel
	sudo apt install --no-install-recommends xserver-xorg-input-libinput
	sudo apt install --no-install-recommends gnome-session
	sudo apt install --no-install-recommends xcursor-themes
	sudo apt install --no-install-recommends dhpcd5
	sudo apt install --no-install-recommends gnome-terminal (o la terminal de preferencia)
	sudo apt install --no-install-recommends unzip
	sudo apt install --no-install-recommends google-chrome-stable (no olvides agregar los repos necesarios)
	sudo apt install --no-install-recommends gdm3
	sudo systemctl enable gdm3
	sudo systemctl enable dhcpcd

**Opcional:**

	sudo apt install --no-install-recommends libgl1-mesa-dri x11-xserver-utils gnome-themes gnome-terminal gnome-control-center nautilus gnome-icon-theme gnome-software pulseaudio pavucontrol


<a id="org984cafe"></a>

## Instalación de Mínima de KDE Plasma:

	sudo apt install --no-install-recommends xserver-xorg-video-amdgpu # Si tu tarjeta gráfica es intel, cambiamos amdgpu por intel
	sudo apt install --no-install-recommends xserver-xorg-input-libinput
	sudo apt install --no-install-recommends kde-plasma-desktop lightdm plasma-nm
	sudo systemctl enable lightdm
