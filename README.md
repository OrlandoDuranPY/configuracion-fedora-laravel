# Configuración de Fedora Workstation 43 para PHP y Laravel

Este repositorio tiene como objetivo documentar la configuración de un entorno de desarrollo en **Fedora Workstation 43** orientado a **PHP** y **Laravel**.

A lo largo de esta guía configuraremos los componentes necesarios para trabajar de forma cómoda y ordenada con estas tecnologías, cubriendo, entre otros, los siguientes temas:

- Instalación y configuración de **Apache**.
- Asignación de permisos a la carpeta `/var/www/html/`.
- Instalación y configuración de **MySQL**.
- Instalación y configuración de **PostgreSQL**.
- Instalación de **PHP** y **Composer**.
- Instalación y configuración de **phpMyAdmin**.
- Ajustes adicionales útiles para el desarrollo con Laravel.

El objetivo final es contar con un entorno de desarrollo estable, reproducible y fácil de mantener para proyectos basados en PHP/Laravel.

## 1. Instalación de Apache (httpd)

En este primer paso instalaremos y habilitaremos el servidor web **Apache (httpd)**, que utilizaremos como base para ejecutar nuestras aplicaciones PHP/Laravel.

### 1.1 Instalar el paquete httpd

Ejecuta el siguiente comando para instalar Apache:

```bash
sudo dnf install httpd -y
```

El parámetro `-y` acepta automáticamente las confirmaciones de instalación.

### 1.2 Iniciar el servicio de Apache

Una vez instalado, inicia el servicio:

```bash
sudo systemctl start httpd.service
```

Opcionalmente, puedes verificar que el servicio esté en ejecución con:

```bash
systemctl status httpd.service
```

### 1.3 Habilitar Apache al inicio del sistema

Para que Apache se inicie automáticamente cada vez que arranque el sistema, ejecuta:

```bash
sudo systemctl enable httpd.service
```

Con esto, el servicio httpd quedará instalado, iniciado y configurado para arrancar automáticamente con Fedora Workstation 43.

## 2. Asignar permisos a `/var/www/html/`

Por defecto, el directorio raíz de los sitios web en Apache en Fedora es:

`/var/www/html`

Este directorio suele pertenecer al usuario y grupo del sistema (por ejemplo, `root`), lo que obliga a usar `sudo` para crear o editar archivos. Para facilitar el trabajo en un entorno de desarrollo local, es común cambiar el propietario del directorio al usuario que desarrolla en la máquina.

En este ejemplo, el usuario del sistema es `orlandoduranpy`, por lo que se puede ejecutar el siguiente comando:

```bash
sudo chown orlandoduranpy /var/www/html
```

### ¿Qué hace este comando?

- `sudo`

  Ejecuta el comando con privilegios de administrador (superusuario). Es necesario porque se están modificando permisos de un directorio del sistema.

- `chown`

  Significa change owner (cambiar propietario). Permite cambiar el usuario y/o grupo propietario de un archivo o carpeta.

- `orlandoduranpy`

  Es el nuevo propietario que se asignará al directorio. En este caso, corresponde al usuario del sistema que desarrollará en Fedora.

- `/var/www/html`

  Es el directorio donde Apache sirve los archivos web por defecto.

Después de ejecutar este comando, el usuario `orlandoduranpy` podrá crear, editar y eliminar archivos en `/var/www/html` sin necesidad de utilizar `sudo` constantemente.

✅ Nota:
Si se desea que el cambio aplique también a todos los archivos y subdirectorios dentro de `/var/www/html`, se puede usar la opción `-R` (recursivo):

```bash
sudo chown -R orlandoduranpy /var/www/html
```

Esta opción debe utilizarse con cuidado, ya que modificará el propietario de todos los elementos contenidos en ese directorio.
