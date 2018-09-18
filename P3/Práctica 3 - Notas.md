# PRÁCTICA 3

## Sesión 1

En la P1 configuramos nuestro servidor, en la P2 le añadimos servicios.
Ahora en la P3 queremos:
- monitorizar nuestro RAID1.
- Utilizar Ansible
- Utilizar timmers systemd (sustituto de cron que está obsoleto)
- Utilizaremos Zabbix

En la P4 le aplicaremos una carga al sistema, lo monitorizaremos y ajustaremos parámetros.

---------------------------------

Monitores Hardware: lm-sensors

Mensajes del kernel: dmesg -> Permite conocer los mensajes del kernel (durante el arrancado, problemas de HW o periféricos...)

Monitores Software: /proc

En concreto, /proc/mdstat

--------------------------------------------

Nos vamos al virtual-BOX y vemos la configuración de Ubuntu Server -> almacenamiento -> no le deja quitar un disco duro.

Como vemos si funciona nuestro RAID1 -> nos vamos a nuestro servidor y borramos el disco duro.

No da igual que disco duro quitamos ya que instalamos GRUB en sda, pero esto puede solucionarse haciendo *sudo grub-install /dev/sdb*

*poweroff*

Quitamos el segundo disco en configuracioón almacenamiento de VirtualBOX.

Iniciamos de nuevo la máquina y da un warning en bucle.

Cuando acabe hacemos *dmesg*, el cual nos indica el estado y configuración del kernel..

Hacemos *lvm* (dentro del initramfs) y hacemos *lvdisplay*.

*cat /proc/mdstat* -> para ver qué está pasando con el raid1 (que era un multidevice md)

Vemos que está inactivo lo activamos con *mdadm --run md0* y volvemos a hacer el cat.

Vemos el [2/1] que indica que antes había 2 dispositivos y ahora 1. Y [U_] indica que hay uno activo (UP) y otro inactivo (down).

Lo reiniciamos y debería funcionar pero sigue dando los warning (es un bug de Ubuntu Server).

Metemos un disco duro mediante Virtual box y vemos que lo hemos hecho correctamente con lsblk, lsusb lspci, lshw.

Ahora hacemos watch -n 2 cat /proc/mdstat

Hacemos mdadm --add /dev/md0 /dev/sdb

-------------------------------------------

Automatización: Ansible

¿Qué es Ansible y qué permite hacer?

Ansible es una herramienta de monitorización que nos permite monitorizar de forma automática varios hosts definidos en el archivo /etc/ansible/hosts y podemos interactuar con ellos mediante ssh.

Para monitorizar con ansible usamos *ansible all -m setup*


## Sesión 2 - MONITORES

La elección de qué monitor usar será tomada en función de nuestras necesidades.

**Shinken: Arquitectura**

Shinken necesita plugins (entre ellos los de Nagios) para reunir información. Es muy modulable ya que se puede ejecutar en un sólo servidor o dividir su ejecución en diferentes hardware si el desarrollo de nuestro servidor es a gran escala o si estamos ejecutando varios planificadores.
Esto hace que Shinken sea muy escalable, además usa la filosofía de UNIX (cada objeto hace una acción).
Lo interesante es que podemos monitorizar muchos servidores y con buena API.

Si queremos monitorización en tiempo real: **netdata**

Este se centra en tener una latencia mínima y un tiempo de visualización mínimo.

**OMD**

Es una distro que tiene todo istalado para monitorizar (Open Monitoring Distro).

Lo que ofrece OMD es naemon + plugins + thruk.

**Naemon**

¿Qué es naemon? Nagios pasó a ser de pagos, en este tipo de situaciones alguien se lo toma a mal, en este caso uno de los desarrolladores retomó el proyecto libre.

Nagios o Naemon ejecuta pequeños scripts/programas que recopilan la información a monitorizar. xinetd se encarga de ejecutarlo, lo que hace es escuchar en todos los puertos de los servicios que gestiona y cuando llega una petición de Nagios arranca el servidor para ese servicio.

Hay muchos monitores, hay que tener en cuenta los tipos de monitorización y el cómo almacenamos esa monitorización.

¿Qué es un plugin?

Podría ser ese pequeño trozo de código que escribimos para ver si estaba activo el RAID1.

En resumen un plugin es código que va mira a ver si está el dato que estamos monitorizando y se lo devuelve a la lógica.

**Zabbix**

Con Zabbix también usamos pluggins, tenemos varios modos de monitorización.

Zabbix es bastante usado y asequible y cumple con los objetivos de la asignatura.

TRABAJO: INSTALACIÓN Y CONFIGURACIÓN DE ZABBIX

Instalar Zabbix , tenemos que aprender a documentar y a buscar documentación.

¿Qué es Zabbix, que distintos procesos tiene? -> Tendremos un server y un agent que se comunican. Tenemos que buscar zabbix_get pueden comunicarse el agente para ver si esta bien configurado.

Zabbix_send para forzar al servidor a recompilar un dato (para ver si esta funcionando, tiene acceso a la base de datos).

Con get podemos ver si agent está funcionando


Tenemos que saber que hay un Zabbix-proxy, es un elemento adicional que introduce Zabbix para en el caso de que haya varios elementos monitorizados.

El proxy es un mecanismo de ocultación para que varias máquinas se conecten mediante el proxy al exterior de tal forma que no podemos tracear las máquinas.

En nuestro caso si tenemos 100k conexiones con el servidor es complicado, pero si usamos una jerarquía de proxys se facilita la labor.

NO INSTALAR PROXY.
