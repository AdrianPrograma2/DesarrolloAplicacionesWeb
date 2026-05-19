# 🖥️ Infraestructura Web Completa con Apache, DNS y Servicios de Red

> Proyecto de despliegue de un servidor web completo sobre Ubuntu 24.04 LTS en VirtualBox, con automatización, DNS local, FTP seguro y soporte Python.

**Entorno de trabajo:**
- 🖥️ SO: Ubuntu Desktop 24.04 LTS (VirtualBox - Adaptador Puente)
- 🌐 IP del servidor: `192.168.1.135`
- 📡 Dominio local: `marisma.local`
- 📁 Directorio base: `~/infraestructura-web/`

---

## 📚 Índice

- [1. Preparación del sistema](#1-preparación-del-sistema)
- [2. Stack LAMP](#2-stack-lamp)
- [3. Script de automatización de clientes](#3-script-de-automatización-de-clientes)
- [4. FTP, SSH y Python](#4-ftp-ssh-y-python)
- [5. Servidor DNS con BIND9](#5-servidor-dns-con-bind9)
- [6. Verificación final](#6-verificación-final)
- [7. Arquitectura del sistema](#7-arquitectura-del-sistema)

---

## 1. Preparación del sistema

Lo primero es dejar el sistema actualizado e instalar las herramientas que se van a necesitar durante todo el proyecto.

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
net-tools curl wget vim git unzip

mkdir -p ~/infraestructura-web/{scripts,images,backups}
cd ~/infraestructura-web
```

Después de esto el sistema queda listo con conectividad comprobada y la estructura de carpetas organizada.

---

## 2. Stack LAMP

### Instalación de Apache2, PHP y MariaDB

El objetivo es tener un entorno LAMP funcional con phpMyAdmin para gestionar las bases de datos desde el navegador.

```bash
sudo apt install -y apache2 mariadb-server mariadb-client \
php php-cli php-mysql php-curl php-gd php-xml php-mbstring php-zip \
libapache2-mod-php phpmyadmin
```

### Configuración y arranque

```bash
sudo systemctl enable apache2 mariadb
sudo systemctl start apache2 mariadb

sudo a2enmod rewrite ssl
sudo systemctl restart apache2

sudo ln -s /etc/phpmyadmin/apache.conf \
/etc/apache2/conf-available/phpmyadmin.conf

sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```

**Resultado:**
- Apache2 escuchando en el puerto 80
- PHP 8.3 operativo
- MariaDB funcionando en puerto 3306
- phpMyAdmin accesible desde el navegador

---

## 3. Script de automatización de clientes

### ¿Qué hace el script?

En lugar de configurar cada cliente a mano, se desarrolló un script que crea todo automáticamente:

- Usuario del sistema Linux
- Directorio web propio
- VirtualHost en Apache
- Registro DNS
- Base de datos y usuario MySQL
- Contraseña segura aleatoria

### Cómo usarlo

```bash
sudo ~/infraestructura-web/scripts/crear_cliente.sh empresa 192.168.1.135
```

El script está en `~/infraestructura-web/scripts/crear_cliente.sh` y solo necesita el nombre del cliente y la IP del servidor.

---

## 4. FTP, SSH y Python

### Instalación

```bash
sudo apt install -y \
vsftpd \
openssh-server \
libapache2-mod-wsgi-py3
```

### Configuración FTP seguro

Se edita `/etc/vsftpd.conf` habilitando:

```
ssl_enable=YES
chroot_local_user=YES
```

### Puertos abiertos en el cortafuegos

```bash
sudo ufw allow 21/tcp
sudo ufw allow 22/tcp
sudo ufw allow 40000:40100/tcp
```

### Soporte Python con mod_wsgi

```bash
sudo a2enmod wsgi
sudo systemctl reload apache2
```

**Resultado:**
- FTP cifrado con TLS
- SSH y SFTP activos
- Aplicaciones Python ejecutándose mediante mod_wsgi

---

## 5. Servidor DNS con BIND9

### Instalación

```bash
sudo apt install -y \
bind9 bind9-utils bind9-doc dnsutils
```

### Configuración

Archivo principal: `/etc/bind/named.conf.local`

Zonas configuradas:
- Zona directa: `marisma.local`
- Zona inversa: `1.168.192.in-addr.arpa`

### Validar la configuración

```bash
sudo named-checkconf
sudo named-checkzone marisma.local /etc/bind/db.marisma.local
```

### Pruebas de resolución

```bash
# Resolución directa
dig @192.168.1.135 cliente1.marisma.local

# Resolución inversa
dig @192.168.1.135 -x 192.168.1.135
```

**Resultado:** resolución directa e inversa funcionando, integrado con el script de clientes para añadir registros automáticamente.

---

## 6. Verificación final

### Comprobar todos los servicios de una vez

```bash
sudo systemctl status apache2 mariadb named vsftpd ssh
```

### Pruebas rápidas

```bash
# Web
curl http://192.168.1.135

# Base de datos
sudo mysql -e "SHOW DATABASES;"

# DNS
dig @192.168.1.135 cliente1.marisma.local +short

# SSH
ssh cliente1@192.168.1.135
```

### Estado final del sistema

| Componente | Estado |
|---|---|
| Apache | ✅ Correcto |
| DNS (BIND9) | ✅ Correcto |
| MariaDB | ✅ Operativa |
| VirtualHosts | ✅ Funcionando |
| FTP/SSH | ✅ Activos |
| Automatización | ✅ Implementada |

---

## 7. Arquitectura del sistema

| Servicio | Tecnología | Puerto |
|---|---|---|
| Servidor Web | Apache2 | 80 / 443 |
| PHP | PHP 8.3 | Interno |
| Base de Datos | MariaDB | 3306 |
| DNS | BIND9 | 53 |
| FTP Seguro | vsftpd | 21 |
| Acceso remoto | OpenSSH | 22 |
| Python WSGI | mod_wsgi | Apache |

### Acceso a los servicios

| Servicio | Dirección |
|---|---|
| Web cliente | `http://empresa.marisma.local` |
| phpMyAdmin | `http://192.168.1.135/phpmyadmin` |
| SSH | `ssh empresa@192.168.1.135` |
| SFTP | `sftp empresa@192.168.1.135` |

### Seguridad aplicada

- FTP cifrado con TLS
- Usuarios aislados con chroot
- Contraseñas aleatorias por cliente
- Bases de datos separadas por cliente
- Validación automática de zonas DNS
- Permisos correctos en directorios web

### Estructura de carpetas

```
~/infraestructura-web/
├── README.md
├── scripts/
├── images/
└── backups/
```

---

## ✅ Resumen de lo conseguido

| Objetivo | Resultado |
|---|---|
| Servidor web con Apache | ✅ |
| Hosting dinámico y estático | ✅ |
| Automatización con scripts | ✅ |
| DNS local con BIND9 | ✅ |
| MariaDB + phpMyAdmin | ✅ |
| FTP seguro con TLS | ✅ |
| SSH/SFTP operativos | ✅ |
| Python con mod_wsgi | ✅ |
| VirtualHosts automáticos | ✅ |
| Gestión multiusuario | ✅ |
