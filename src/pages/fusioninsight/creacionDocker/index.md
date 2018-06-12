---
title: Crear una imagen Docker de FusionInsight
---

## Crear una imagen Docker de FusionInsight

A continuación se describe la manera de como instalar el FusionInsight HD manager en una imagen Docker. El FusionInsight
HD manager tiene una interfaz web que permite descubrir los nodos de un cluster y hacer el deployment de los servicios y 
componentes de FusionInsight HD en cada nodo.

El deployment de los servicios en otras imágenes de Docker aún no se ha logrado. En las siguientes secciones se describe
el procedimiento que se ha seguido y las dificultades que se han encontrado para poder realizar una instalación completa
de FusionInsight en imágenes de Docker.

### Instalación de FusionInsight HD Manager

En esta sección se describirá como realizar una instalación de FusionInsight en una imagen de Docker: Se utilizará como
base el sistema operativo CentOS 7.3.

##Preparación

Se requiere de de una PC con Docker instalado.  Se requiere descargar de la página de Huawei el paquete de
instalación de FusionInsight HD Manager llamado FusionInsight_Manager_V100R002C80SPC200_RHEL.tar.gz para los sistemas 
operativos Red Hat, CentOs y Euler. Finalmente, descargar de GitHub el script de Python para reemplazar systemctl de 
la imagen de Python de la siguiente liga (https://github.com/gdraheim/docker-systemctl-replacement/releases), en el 
momento de las pruebas se utilizó la versión 1.2.2177 pero al momento de escribir esto se acaba de liberar un nuevo 
release 1.3.2236. 


##¿Qué hacer?

1. Crear un directorio donde se creará el archivo Docker y los elementos para realizar la instalación.
2. Dentro de esta carpeta colocar el siguientes archivos:
    * El paquete de instalación de FusionInsight Manager, para esta demo se renombró como FusionInsight.tar.gz.
    * El script de reemplazo de systemctl (systemctl.py) que se encuentra en la ruta /docker-systemctl-replacement-version/files/docker/
    del archivo descargado y desempacarlo.  
3. Generar un archivo llamado `CentOS-Vault.repo` con el siguiente contenido

```$bash
# CentOS Vault contains rpms from older releases in the CentOS-7 
# tree.

#c7.0.1406
[C7.0.1406-base]
name=CentOS-7.0.1406 - Base
baseurl=http://vault.centos.org/7.0.1406/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.0.1406-updates]
name=CentOS-7.0.1406 - Updates
baseurl=http://vault.centos.org/7.0.1406/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.0.1406-extras]
name=CentOS-7.0.1406 - Extras
baseurl=http://vault.centos.org/7.0.1406/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.0.1406-centosplus]
name=CentOS-7.0.1406 - CentOSPlus
baseurl=http://vault.centos.org/7.0.1406/centosplus/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.0.1406-fasttrack]
name=CentOS-7.0.1406 - CentOSPlus
baseurl=http://vault.centos.org/7.0.1406/fasttrack/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

# C7.1.1503
[C7.1.1503-base]
name=CentOS-7.1.1503 - Base
baseurl=http://vault.centos.org/7.1.1503/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.1.1503-updates]
name=CentOS-7.1.1503 - Updates
baseurl=http://vault.centos.org/7.1.1503/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.1.1503-extras]
name=CentOS-7.1.1503 - Extras
baseurl=http://vault.centos.org/7.1.1503/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.1.1503-centosplus]
name=CentOS-7.1.1503 - CentOSPlus
baseurl=http://vault.centos.org/7.1.1503/centosplus/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.1.1503-fasttrack]
name=CentOS-7.1.1503 - CentOSPlus
baseurl=http://vault.centos.org/7.1.1503/fasttrack/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

# C7.2.1511
[C7.2.1511-base]
name=CentOS-7.2.1511 - Base
baseurl=http://vault.centos.org/7.2.1511/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.2.1511-updates]
name=CentOS-7.2.1511 - Updates
baseurl=http://vault.centos.org/7.2.1511/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.2.1511-extras]
name=CentOS-7.2.1511 - Extras
baseurl=http://vault.centos.org/7.2.1511/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.2.1511-centosplus]
name=CentOS-7.2.1511 - CentOSPlus
baseurl=http://vault.centos.org/7.2.1511/centosplus/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.2.1511-fasttrack]
name=CentOS-7.2.1511 - CentOSPlus
baseurl=http://vault.centos.org/7.2.1511/fasttrack/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0


# C7.3.1611
[C7.3.1611-base]
name=CentOS-7.3.1611 - Base
baseurl=http://vault.centos.org/7.3.1611/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.3.1611-updates]
name=CentOS-7.3.1611 - Updates
baseurl=http://vault.centos.org/7.3.1611/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.3.1611-extras]
name=CentOS-7.3.1611 - Extras
baseurl=http://vault.centos.org/7.3.1611/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.3.1611-centosplus]
name=CentOS-7.3.1611 - CentOSPlus
baseurl=http://vault.centos.org/7.3.1611/centosplus/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0

[C7.3.1611-fasttrack]
name=CentOS-7.3.1611 - CentOSPlus
baseurl=http://vault.centos.org/7.3.1611/fasttrack/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
enabled=0
```

4. Crear un nuevo archivo Docker con el siguiente contenido 

```
FROM centos:7.3.1611

ENV container docker 

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-#setup.service ] || rm -f $i; done); \ 
rm -f /lib/systemd/system/multi-user.target.wants/*;\ 
rm -f /etc/systemd/system/*.wants/*;\ 
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \ 
rm -f /lib/systemd/system/basic.target.wants/*;\ 
rm -f /lib/systemd/system/anaconda.target.wants/*; 

VOLUME [ "/sys/fs/cgroup" ]

COPY CentOS-Vault.repo  /etc/yum.repos.d/CentOS-Vault.repo

RUN yum-config-manager -v --disable CentOS\*; yum-config-manager --enable C7.3\*; yum update; yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel rsync net-tools ntp expect unzip openssl sudo openssh openssh-clients cronie rsyslog openssh-server passwd openssl-devel which; yum clean all


ADD ./start.sh /start.sh
RUN mkdir /var/run/sshd

RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N '' 

RUN chmod 755 /start.sh
RUN ./start.sh

COPY systemctl.py /usr/bin/systemctl
RUN chmod g+x /usr/bin/systemctl
COPY FusionInsight.tar.gz FusionInsight.tar.gz

RUN tar -xvf FusionInsight.tar.gz
WORKDIR /FusionInsight_Manager/software/


RUN /bin/sh install.sh -m single -t 216.239.35.0,216.239.35.4

CMD ["/usr/sbin/init"]

```

Situado en la misma carpeta, realizar la construcción de la imagen de Docker mediante el siguiente comando dentro de 
la terminal 
```$bash
$ docker build -t fusionInsight .
```

Esperar que la construcción concluya. Una vez concluido el proceso de manera exitosa ejecutar la imagen de Docker:

```$bash
$ docker run --privileged --name finsightExecut -v /sys/fs/cgroup:/sys/fs/cgroup:ro  -d -t -p 8080:8080 -p 20009:20009 -p 28443:28443 -p 20014:20014 -p 20018:20018 -p 21750:21750 -p 20026:20026 -p 20027:20027  fusionInsight
```

Por medio de un bowser navegar a la página: <http://localhost:8080/web>. El browser desplegará un error de certificado 
por lo que habrá que generar una excepción para cada uno de los puertos a donde se redirecciona la petición. 

Una vez que se despliega la pantalla de login acceder con las siguientes credenciales, usuario: admin password: Admin@123.

Una vez accedido al sistema pedirá cambiar la contraseña. A partir de este punto la instalación del FusionInsight HD 
Manager habrá quedado completada. 


##¿Cómo funciona?

El principal problema que se tiene para instalar el FusionInsight HD Manager en una imagen de Docker es el systemctl que es 
usado extensivamente por el instalador para manejar los procesos como es el NTP. Dado que en general systemctl no 
funciona dentro de una imagen de Docker, el script de instalación falla constantemente sin poder terminar. 

La primera parte del archivo Docker sirve para definir que la imagen base es la versión de CentOS 7.3.1611. Posteriormente
se eliminan los archivos de systemd de la imagen para permitir la ejecución de systemctl. Se reescribe el archivo de 
repositorios de CentOs, lo anterior para evitar que paquetes para una versión posterior a la 7.3 sean instalados, dado que 
en caso de hacerlo, la versión de CentOs cambiará por ejemplo a la 7.5 y el instalador de FusionInsight HD Manager 
abortará porque no es una versión de OS soportada. 

Posteriormente se realiza la instalación de todos los paquetes que requiere el script de FusionInsight HD Manager para 
poder ser instalado los cuales son: `openldap`, `compat-openldap`, `openldap-clients`, `openldap-servers`, `openldap-servers-sql`, 
`openldap-devel`. `rsync`, `net-tools`, `ntp`, `expect`. `unzip`, `openssl`, `sudo`, `openssh`, `openssh-clients`,
`cronie`, `rsyslog`, `openssh-server` `passwd`, `openssl-devel` y  `which`. 

Se realiza la configuración del servicio de ssh en la imagen de Docker.  Se sobreescribe el binario de systemctl por el 
script de python. Se realiza la copia del instalador de FusionInsight HD Manager en la imagen y se ejecuta la instalación.
El instalador de FusionInsight HD realizará los siguientes pasos:
```$bash
STEP 1 Checking the parameters.
STEP 2 Preparing the installation components.
STEP 3 Installing the manager.
STEP 4 Installing the packs.
STEP 5 Starting the OMS.
STEP 6 Waiting for ntp to startup.
```

Al terminar los seis pasos de manera exitosa se habrá completado la instalación del FusionInsight HD Manager y la creación 
de la imagen Docker. La imagen de Docker se debe de ejecutar mapeando los puertos de FusionInsight HD hacia el host. 

##Problemas 

El principal problema que se tiene es que una vez que se ejecuta el instalador de FusionInsight HD Manager este aborta en
alguno de sus seis pasos, siendo el paso 6 y el paso 3 los más comunes. En caso de abortar la instalación en el paso 
6 normalmente sucede mientras se realiza la espera en que termine el proceso de NTP. En este punto la mayor parte de la 
instalación ha quedado contemplada. Normalmente desinstalando y volviendo a intalar el manager hace que este se complete.

Para poder realizar los siguiente es necesario recuperar la imagen Docker fallida. Primer localizar el contenedor que 
falló:
```$bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
6d8ee0965ade        425602417a97        "/bin/sh -c './utils/"   20 minutes ago
```
Hacer un commit del contenedor en una imagen:
```$bash
$ docker commit 6d8ee0965ade
sha256:fe224ed1eede179b435be49c7dd49d5410f3cc3f2145fb510d7cc499ce201ca5
```

y finalmente ejecutar la imagen:
```
$ docker run -it 80d5ac097a96 [bash -il]
```

Se puede desinstalar el FusionInsight HD Manager mediante el siguiente comando:
```
$ /opt/huawei/Bigdata/om-server/om/inst/uninstall.sh
```
y volver a ejecutar la instalación mediante el comando 
```$bash
$ /bin/sh install.sh -m single -t 216.239.35.0,216.239.35.4
```
situándose dentro de la carpeta de instalación: `/FusionInsight_Manager/software/`.

En caso de que el problema no haya sucedido en el paso 6 o se quiera validar el fallo, una vez montado el contenedor con
el fallo se puede revisar los logs de la instalación. El instalador tiene un log principal y va generando logs por cada
uno de los sub-scripts que se ejecutan, cada sub-script instala algún componente. La carpeta de instalación del 
FusionInsight HD Manager es donde se pueden encontrar estos scripts `/opt/huawei/Bigdata/om-server/om/inst/`.

##¿Qué no hacer?

De las cosas que se tiene experiencia y no funcionan son las siguientes.

Ejecutar el instalador con el flag `-U` lo cual forza a no realizar actualizaciones sobre los componentes. Con esto se 
logra que el servidor NTP no se instale el cual es un fallo común, lamentablemente esto también evita que se NO se 
instale el servicio de Kerberos por lo que no se puede acceder a a la plataforma Web del FusionInsight HD Manager dejando
la instalación inutilizable. 

