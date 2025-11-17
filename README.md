# Configuración de Fedora Workstation para PHP y Laravel

Este repositorio tiene como objetivo documentar la configuración de un entorno de desarrollo en **Fedora Workstation** orientado a **PHP** y **Laravel**.

A lo largo de esta guía configuraremos los componentes necesarios para trabajar de forma cómoda y ordenada con estas tecnologías, cubriendo, entre otros, los siguientes temas:

- Instalación y configuración de **Apache**.
- Asignación de permisos a la carpeta `/var/www/html/`.
- Instalación y configuración de **MySQL**.
- Instalación y configuración de **PostgreSQL**.
- Instalación de **PHP** y **Composer**.
- Instalación y configuración de **phpMyAdmin**.
- Ajustes adicionales útiles para el desarrollo con Laravel.

El objetivo final es contar con un entorno de desarrollo estable, reproducible y fácil de mantener para proyectos basados en PHP/Laravel.

## 1. Instalar Apache (httpd)

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

Con esto, el servicio httpd quedará instalado, iniciado y configurado para arrancar automáticamente con Fedora Workstation.

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

## 3. Instalación de MySQL

En este paso se instalará y habilitará el servidor de base de datos MySQL, que será utilizado por las aplicaciones desarrolladas con PHP y Laravel.

### 3.1 Instalar el servidor MySQL

Para instalar el servidor MySQL en Fedora, se ejecuta el siguiente comando:

```bash
sudo dnf install mysql-server
```

Este comando descargará e instalará el paquete `mysql-server` junto con sus dependencias.

### 3.2 Iniciar y habilitar el servicio MySQL

Una vez instalado, se debe iniciar el servicio de MySQL:

```bash
sudo systemctl start mysqld
```

Para que el servicio se inicie automáticamente cada vez que arranque el sistema, se habilita con:

```bash
sudo systemctl enable mysqld
```

Opcionalmente, se puede comprobar el estado del servicio con:

```bash
systemctl status mysqld
```

### 3.3 Configuración inicial con `mysql_secure_installation`

Después de la instalación, es recomendable ejecutar el asistente de configuración de seguridad de MySQL para definir la contraseña del usuario administrador (`root`) y aplicar algunas medidas básicas de seguridad.

Para iniciar el asistente, se utiliza el siguiente comando:

```bash
sudo mysql_secure_installation
```

Al ejecutar este comando, MySQL mostrará algo similar a lo siguiente:

```bash
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No:
```

En este punto, el asistente solicita la contraseña actual del usuario `root` y confirma que el componente `validate_password` está instalado y configurado.

A continuación, se preguntará si se desea cambiar la contraseña de `root`:

```bash
Change the password for root ? ((Press y|Y for Yes, any other key for No) :
```

Si se responde `y` o `Y`, el asistente permitirá establecer una nueva contraseña. Durante este proceso MySQL mostrará un estimado de la fortaleza de la contraseña:

```bash
New password:

Re-enter new password:

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) :
```

Si la contraseña no cumple con la política definida por `validate_password`, MySQL mostrará un mensaje de error indicando que no satisface los requisitos de la política, y solicitará ingresar una nueva contraseña hasta que sea aceptada.

Una vez configurada la contraseña, el asistente continúa con otras preguntas de seguridad:

- **Eliminar usuarios anónimos:**

```bash
Remove anonymous users? (Press y|Y for Yes, any other key for No) :
```

En entornos de desarrollo se puede responder `y` para mantener un entorno más limpio.

- **Restringir el acceso remoto de `root`:**

```bash
Disallow root login remotely? (Press y|Y for Yes, any other key for No) :
```

En entornos de producción se recomienda responder `y`. En desarrollo, se puede dejar el acceso remoto deshabilitado o habilitado según la necesidad.

- **Eliminar la base de datos de pruebas (`test`):**

```bash
Remove test database and access to it? (Press y|Y for Yes, any other key for No) :
```

En producción se recomienda eliminarla; en desarrollo es opcional.

- **Recargar las tablas de privilegios:**

```bash
Reload privilege tables now? (Press y|Y for Yes, any other key for No) :
```

Lo habitual es responder `y` para aplicar inmediatamente todos los cambios.

Al finalizar, el asistente mostrará un mensaje similar a:

```bash
Success.

All done!
```

Con esto, la instalación de **MySQL** queda asegurada con una contraseña para `root` y con una configuración básica de seguridad adecuada para continuar con el entorno de desarrollo.

### 3.4 Verificar acceso a MySQL

Como último paso, es recomendable validar que el acceso al servidor MySQL funciona correctamente con el usuario `root` y la contraseña configurada en el asistente anterior.

Para conectarse a MySQL se utiliza:

```bash
sudo mysql -u root -p
```

Donde:

- `sudo` ejecuta el comando con privilegios de superusuario.

- `-u` root indica que se usará el usuario root de MySQL.

- `-p` indica que se solicitará la contraseña al iniciar sesión.

Al ejecutar el comando, el sistema pedirá la contraseña:

```bash
Enter password:
```

Tras introducir la contraseña correcta, se mostrará el monitor de MySQL (prompt interactivo), identificado por algo similar a:

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
mysql>
```

Dentro del monitor de MySQL, se puede listar las bases de datos disponibles con:

```bash
SHOW DATABASES;
```

La salida típica incluirá al menos las siguientes bases de datos del sistema:

- `information_schema`

- `mysql`

- `performance_schema

- `sys`

Si se puede iniciar sesión correctamente y se muestran estas bases de datos sin errores, se puede considerar que la instalación y configuración básica de MySQL se ha realizado con éxito y que el servidor está listo para utilizarse en el entorno de desarrollo.

## 4. Instalación de PHP y Composer

En este paso se instalará PHP junto con un conjunto de extensiones comunes necesarias para trabajar con Laravel y aplicaciones web en general. Posteriormente se verá la instalación de Composer.

### 4.1 Instalación de PHP y extensiones comunes

Para instalar PHP y algunas extensiones recomendadas para Laravel, se puede utilizar el siguiente comando:

```bash
sudo dnf install php php-mysqlnd php-pdo php-gd php-mbstring php-xml php-cli php-common php-json php-opcache php-pgsql php-pdo_pgsql -y
```

Este comando instala:

- `php`
  Paquete principal de PHP (intérprete del lenguaje).

- `php-mysqlnd`
  Controlador nativo de MySQL para PHP. Permite que PHP se conecte a bases de datos MySQL/MariaDB.

- `php-pdo`
  Proporciona la capa PDO (PHP Data Objects), utilizada por Laravel para conectarse a bases de datos de forma abstracta.

- `php-gd`
  Librería para manipulación de imágenes (redimensionar, recortar, generar imágenes, etc.).

- `php-mbstring`
  Extensión para manejar cadenas multibyte (UTF-8, acentos, caracteres especiales). Es requerida por Laravel y muchas librerías.

- `php-xml`
  Permite trabajar con XML, necesario para ciertas extensiones y herramientas del ecosistema PHP.

- `php-cli`
  Versión de PHP para línea de comandos. Es esencial para ejecutar Artisan, Composer y otros scripts.

- `php-common`
  Archivos comunes compartidos por distintos módulos de PHP.

- `php-json`
  Soporte para JSON, ampliamente utilizado en APIs y respuestas de Laravel.

- `php-opcache`
  Módulo de caché de opcode que mejora el rendimiento de PHP al almacenar en caché el código ya compilado.

- `php-pgsql`
  Extensión que permite a PHP conectarse directamente a bases de datos PostgreSQL.

- `php-pdo_pgsql`
  Controlador PDO específico para PostgreSQL. Laravel lo utiliza cuando la conexión se configura con el driver pgsql.

El parámetro `-y` acepta automáticamente las confirmaciones de instalación.

Después de la instalación, se puede verificar la versión de PHP instalada con:

```bash
php -v
```

Esto confirma que PHP está disponible en la línea de comandos y listo para utilizarse en el entorno de desarrollo.

### 4.2 Instalación de Composer

**Composer** es el gestor de dependencias más utilizado en el ecosistema PHP y es una herramienta fundamental para trabajar con `Laravel`, ya que permite instalar el framework y las librerías necesarias para cada proyecto.

En Fedora, Composer se puede instalar directamente desde los repositorios oficiales con el siguiente comando:

```bash
sudo dnf install composer -y
```

Este comando:

- Descarga e instala el paquete `composer`.

- Deja disponible el comando `composer` en la línea de comandos del sistema.

- Utiliza la opción `-y` para aceptar automáticamente las confirmaciones de instalación.

Una vez finalizada la instalación, se puede verificar que Composer esté correctamente instalado y disponible ejecutando:

Una vez finalizada la instalación, se puede verificar que Composer esté correctamente instalado y disponible ejecutando:

```bash
composer -V
```

La salida mostrará la versión instalada de Composer, lo que confirma que la herramienta está lista para ser utilizada en la gestión de dependencias de proyectos PHP y Laravel.

### 4.3 Reiniciar Apache para aplicar los cambios

Después de instalar PHP y sus extensiones, así como Composer, es recomendable reiniciar el servicio de Apache para asegurarse de que cargue correctamente los módulos de PHP recién instalados.

Para reiniciar Apache se utiliza el siguiente comando:

```bash
sudo systemctl restart httpd.service
```

Este comando:

- Detiene y vuelve a iniciar el servicio `httpd`.

- Hace que Apache cargue la configuración y módulos actualizados, incluyendo el soporte para PHP.

Opcionalmente, se puede verificar que Apache siga en ejecución con:

```bash
systemctl status httpd.service
```

Si el servicio aparece como `active (running)` y no se muestran errores, Apache está funcionando correctamente con soporte para PHP y se puede continuar con la configuración del entorno Laravel.
