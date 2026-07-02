# Configuración de Fedora Workstation para PHP y Laravel

Este repositorio tiene como objetivo documentar la configuración de un entorno de desarrollo en **Fedora Workstation** orientado a **PHP** y **Laravel**.

A lo largo de esta guía configuraremos los componentes necesarios para trabajar de forma cómoda y ordenada con estas tecnologías, cubriendo, entre otros, los siguientes temas:

- Instalación y configuración de **Apache**.
- Instalación y configuración de **Nginx** (alternativa a Apache).
- Asignación de permisos a la carpeta `/var/www/html/`.
- Instalación de **PHP** y **Composer**.
- Instalación de **Node.js** y **npm**.
- Instalación y configuración de **MySQL**.
- Instalación y configuración de **PostgreSQL**.
- Instalación y configuración de **Redis**.
- Instalación y configuración de **Mailpit**.
- Instalación de **DBeaver** como administrador gráfico de bases de datos.
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

## 2. Instalación de Nginx (alternativa a Apache)

**Nginx** es un servidor web de alto rendimiento que puede utilizarse como alternativa a Apache para servir aplicaciones PHP y Laravel. Se destaca por su bajo consumo de recursos y su eficiencia en el manejo de conexiones concurrentes.

Se usa **uno u otro**, no ambos a la vez: por defecto Apache y Nginx compiten por el puerto 80. Si prefieres Nginx, sigue esta sección; si te quedas con Apache (sección 1), puedes omitirla.

> ⚠️ Si ya tienes Apache (`httpd`) instalado y en ejecución, detén el servicio antes de iniciar Nginx, ya que ambos compiten por el puerto 80:
>
> ```bash
> sudo systemctl stop httpd.service
> sudo systemctl disable httpd.service
> ```

### 2.1 Instalar Nginx

Para instalar Nginx en Fedora, se ejecuta el siguiente comando:

```bash
sudo dnf install nginx -y
```

### 2.2 Iniciar y habilitar el servicio

Una vez instalado, se inicia el servicio y se configura para arrancar automáticamente con el sistema:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Para verificar que el servicio está activo:

```bash
systemctl status nginx
```

Si aparece como `active (running)`, Nginx está funcionando correctamente.

### 2.3 Instalar PHP-FPM

A diferencia de Apache, Nginx no ejecuta PHP de forma nativa. Necesita comunicarse con el proceso **PHP-FPM** (FastCGI Process Manager) para procesar archivos PHP.

Para instalar PHP-FPM:

```bash
sudo dnf install php-fpm -y
```

> Nota: PHP se instala en detalle en la sección **4. Instalación de PHP y Composer**. Este comando instala el componente `php-fpm`; si todavía no has instalado PHP, se instalará junto con sus dependencias.

Luego se inicia y habilita el servicio:

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

Se puede comprobar su estado con:

```bash
systemctl status php-fpm
```

### 2.4 Configurar un Virtual Host para Laravel

Nginx utiliza archivos de configuración en `/etc/nginx/conf.d/` para definir los sitios que va a servir. A continuación se muestra una configuración básica para un proyecto Laravel.

Crear el archivo de configuración del sitio:

```bash
sudo nano /etc/nginx/conf.d/laravel.conf
```

Y añadir el siguiente contenido (ajusta `server_name` y `root` según tu proyecto):

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/html/mi-proyecto-laravel/public;
    index index.php index.html;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/run/php-fpm/www.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Donde:

- `root` apunta al directorio `public/` del proyecto Laravel, que es el punto de entrada de la aplicación.
- `try_files` redirige todas las peticiones al `index.php` de Laravel si el archivo o directorio no existe físicamente (necesario para el enrutamiento de Laravel).
- `fastcgi_pass unix:/run/php-fpm/www.sock` indica a Nginx que delegue la ejecución de PHP al proceso PHP-FPM a través de un socket Unix.
- El bloque `location ~ /\.` niega el acceso a archivos ocultos como `.env`.

### 2.5 Verificar la configuración y reiniciar Nginx

Antes de reiniciar Nginx, es recomendable verificar que la sintaxis del archivo de configuración sea correcta:

```bash
sudo nginx -t
```

La salida esperada, si no hay errores, es:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Si la verificación es exitosa, se aplican los cambios reiniciando el servicio:

```bash
sudo systemctl restart nginx
```

### 2.6 Ajustar permisos para Nginx

El proceso Nginx corre bajo el usuario `nginx`. Para que pueda leer los archivos del proyecto Laravel en `/var/www/html/`, es necesario que el directorio y sus contenidos sean accesibles para ese usuario.

Una forma sencilla en entornos de desarrollo es añadir el usuario `nginx` al grupo del usuario propietario del directorio, o bien ajustar los permisos:

```bash
sudo usermod -aG orlandoduranpy nginx
```

Luego reiniciar PHP-FPM y Nginx para que el cambio de grupo surta efecto:

```bash
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

✅ Nota:
En entornos de producción se recomienda configurar los permisos con mayor precisión, asegurándose de que solo los directorios de almacenamiento (`storage/`) y caché (`bootstrap/cache/`) tengan permisos de escritura para el servidor web.

## 3. Asignar permisos a `/var/www/html/`

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

  Es el directorio donde Apache sirve los archivos web por defecto. Si usas **Nginx**, su directorio raíz predeterminado es `/usr/share/nginx/html`, pero en esta guía se mantiene `/var/www/html` como ubicación de los proyectos (el `root` del Virtual Host de Nginx apunta a la carpeta `public` del proyecto dentro de `/var/www/html`).

Después de ejecutar este comando, el usuario `orlandoduranpy` podrá crear, editar y eliminar archivos en `/var/www/html` sin necesidad de utilizar `sudo` constantemente.

✅ Nota:
Si se desea que el cambio aplique también a todos los archivos y subdirectorios dentro de `/var/www/html`, se puede usar la opción `-R` (recursivo):

```bash
sudo chown -R orlandoduranpy /var/www/html
```

Esta opción debe utilizarse con cuidado, ya que modificará el propietario de todos los elementos contenidos en ese directorio.

## 4. Instalación de PHP y Composer

En este paso se instalará PHP junto con un conjunto de extensiones comunes necesarias para trabajar con Laravel y aplicaciones web en general. Posteriormente se verá la instalación de Composer.

### 4.1 Instalación de PHP y extensiones comunes

Para instalar PHP y algunas extensiones recomendadas para Laravel, se puede utilizar el siguiente comando:

```bash
sudo dnf install php php-mysqlnd php-pdo php-gd php-mbstring php-xml php-cli php-common php-json php-opcache php-pgsql php-pdo_pgsql php-bcmath -y
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

- `php-bcmath`
  Proporciona funciones matemáticas de precisión arbitraria. Requerida por algunas funciones de Laravel y paquetes de terceros que trabajan con números decimales de alta precisión.

✅ Nota:
Las extensiones `php-curl`, `php-zip` y `php-intl` vienen incluidas por defecto en la instalación de PHP en Fedora. Se recomienda verificar que estén activas con `php -m | grep -E "curl|zip|intl"`.

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

```bash
composer -V
```

La salida mostrará la versión instalada de Composer, lo que confirma que la herramienta está lista para ser utilizada en la gestión de dependencias de proyectos PHP y Laravel.

### 4.3 Reiniciar el servidor web para aplicar los cambios

Después de instalar PHP y sus extensiones, así como Composer, es recomendable reiniciar el servicio del servidor web para asegurarse de que cargue correctamente los módulos de PHP recién instalados.

**Si usas Apache**, para reiniciarlo se utiliza el siguiente comando:

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

**Si usas Nginx (con PHP-FPM)**, el intérprete de PHP lo gestiona PHP-FPM, por lo que hay que reiniciar ese servicio (y recargar Nginx) para aplicar los cambios:

```bash
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

Opcionalmente, se puede verificar que ambos servicios sigan en ejecución con:

```bash
systemctl status php-fpm
systemctl status nginx
```

Si ambos aparecen como `active (running)` y no se muestran errores, Nginx está funcionando correctamente con soporte para PHP a través de PHP-FPM.

## 5. Instalación de Node.js y npm

Laravel utiliza **Vite** como herramienta de compilación de assets (CSS, JavaScript) a partir de Laravel 9. Para ejecutar `npm run dev` y `npm run build` dentro de un proyecto Laravel es necesario tener Node.js y npm instalados en el sistema.

### 5.1 Instalar Node.js y npm

En Fedora, Node.js y npm se pueden instalar directamente desde los repositorios oficiales con el siguiente comando:

```bash
sudo dnf install nodejs -y
```

Este comando instala tanto Node.js como npm en su misma operación, ya que npm viene incluido como dependencia del paquete `nodejs`.

### 5.2 Verificar la instalación

Una vez finalizada la instalación, se puede confirmar que ambas herramientas están disponibles verificando sus versiones:

```bash
node --version
npm --version
```

La salida mostrará las versiones instaladas, por ejemplo:

```
v22.22.2
10.9.7
```

Con Node.js y npm disponibles, es posible instalar las dependencias de frontend de un proyecto Laravel ejecutando `npm install` dentro del directorio del proyecto, y compilar los assets con `npm run dev` (modo desarrollo) o `npm run build` (producción).

## 6. Instalación de MySQL

En este paso se instalará y habilitará el servidor de base de datos MySQL, que será utilizado por las aplicaciones desarrolladas con PHP y Laravel.

### 6.1 Instalar el servidor MySQL

Para instalar el servidor MySQL en Fedora, se ejecuta el siguiente comando:

```bash
sudo dnf install mysql-server
```

Este comando descargará e instalará el paquete `mysql-server` junto con sus dependencias.

### 6.2 Iniciar y habilitar el servicio MySQL

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

### 6.3 Configuración inicial con `mysql_secure_installation`

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

### 6.4 Verificar acceso a MySQL

Como último paso, es recomendable validar que el acceso al servidor MySQL funciona correctamente con el usuario `root` y la contraseña configurada en el asistente anterior.

Para conectarse a MySQL se utiliza:

```bash
sudo mysql -u root -p
```

Donde:

- `sudo` ejecuta el comando con privilegios de superusuario.

- `-u root` indica que se usará el usuario root de MySQL.

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

- `performance_schema`

- `sys`

Si se puede iniciar sesión correctamente y se muestran estas bases de datos sin errores, se puede considerar que la instalación y configuración básica de MySQL se ha realizado con éxito y que el servidor está listo para utilizarse en el entorno de desarrollo.

### 6.5 Crear usuario con acceso completo (recomendado en desarrollo)

Por defecto, el único usuario con acceso total en MySQL es `root`. Para el trabajo diario en desarrollo es conveniente crear un usuario propio con privilegios completos sobre todas las bases de datos, evitando así usar `root` directamente.

Para acceder a MySQL como `root` y crear el nuevo usuario:

```bash
sudo mysql -u root -p
```

Dentro del monitor de MySQL se ejecutan los siguientes comandos:

```sql
CREATE USER 'nombre_usuario'@'localhost' IDENTIFIED BY 'contraseña_segura';
GRANT ALL PRIVILEGES ON *.* TO 'nombre_usuario'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
\q
```

Donde:

- `CREATE USER` crea el nuevo usuario con su contraseña.
- `GRANT ALL PRIVILEGES ON *.*` otorga permisos sobre todas las bases de datos (`*`) y todas las tablas (`*`).
- `WITH GRANT OPTION` permite que el usuario pueda a su vez otorgar privilegios a otros usuarios.
- `FLUSH PRIVILEGES` aplica los cambios inmediatamente en el servidor.
- `\q` cierra el monitor de MySQL.

Para verificar que el usuario fue creado correctamente con privilegios de superusuario:

```bash
sudo mysql -u root -p -e "SELECT user, host, Super_priv FROM mysql.user WHERE user='nombre_usuario';"
```

La salida esperada es:

```
+----------------+-----------+------------+
| user           | host      | Super_priv |
+----------------+-----------+------------+
| nombre_usuario | localhost | Y          |
+----------------+-----------+------------+
```

El valor `Y` en `Super_priv` confirma que el usuario tiene privilegios de superusuario sobre el servidor MySQL.

✅ Nota:
Otorgar todos los privilegios es adecuado en entornos de desarrollo local. En producción se recomienda aplicar el principio de mínimo privilegio y conceder únicamente los permisos estrictamente necesarios para cada aplicación.

## 7. Instalación de PostgreSQL

En este paso se instalará y configurará el servidor de base de datos **PostgreSQL**, que puede utilizarse como alternativa a MySQL en proyectos Laravel.

### 7.1 Instalar el servidor PostgreSQL

Para instalar PostgreSQL en Fedora, se ejecuta el siguiente comando:

```bash
sudo dnf install postgresql-server -y
```

Este comando descargará e instalará el paquete `postgresql-server` junto con sus dependencias.

### 7.2 Inicializar el clúster de base de datos

A diferencia de MySQL, PostgreSQL requiere un paso adicional antes de iniciar el servicio: inicializar el directorio de datos. Esto se realiza con:

```bash
sudo postgresql-setup --initdb
```

Este comando crea la estructura inicial de archivos y directorios necesarios para que PostgreSQL funcione, ubicados en `/var/lib/pgsql/data/`.

La salida esperada es similar a:

```
 * Initializing database in '/var/lib/pgsql/data'
 * Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log
```

### 7.3 Iniciar y habilitar el servicio PostgreSQL

Una vez inicializado, se inicia el servicio:

```bash
sudo systemctl start postgresql
```

Para que el servicio se inicie automáticamente en cada arranque del sistema:

```bash
sudo systemctl enable postgresql
```

Se puede verificar que el servicio está activo con:

```bash
systemctl status postgresql
```

### 7.4 Habilitar conexiones por TCP/IP

Por defecto, PostgreSQL en Fedora solo acepta conexiones a través de un **socket Unix**, no por TCP/IP. Esto es suficiente para acceder con `sudo -u postgres psql`, pero impide que herramientas como **DBeaver**, Laravel o cualquier cliente que se conecte vía `host:puerto` puedan establecer la conexión (se obtiene un error de tipo `Connection refused`).

Para habilitarlo, se edita el archivo de configuración principal:

```bash
sudo nano /var/lib/pgsql/data/postgresql.conf
```

Y se descomenta (quitando el `#`) la siguiente línea:

```
listen_addresses = 'localhost'
```

### 7.5 Configurar el método de autenticación

Por defecto, `pg_hba.conf` utiliza el método de autenticación **`ident`** para las conexiones por host, el cual valida la identidad contra el usuario del sistema operativo en lugar de pedir una contraseña. Esto provoca el error `la autentificación Ident falló para el usuario`.

Se edita el archivo:

```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf
```

Y se cambia el método `ident` por `scram-sha-256` en las líneas correspondientes a `127.0.0.1/32` y `::1/128`:

```
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

Después de modificar ambos archivos, se reinicia el servicio para aplicar los cambios:

```bash
sudo systemctl restart postgresql
```

Se puede confirmar que PostgreSQL ya está escuchando en el puerto TCP con:

```bash
sudo ss -tlnp | grep 5432
```

### 7.6 Crear usuario y base de datos

PostgreSQL utiliza el usuario del sistema `postgres` como administrador. Para acceder al intérprete interactivo de PostgreSQL (`psql`) con ese usuario se ejecuta:

```bash
sudo -u postgres psql
```

Dentro del prompt de `psql`, se crea un usuario y una base de datos para el proyecto:

```sql
CREATE USER nombre_usuario WITH PASSWORD 'contraseña_segura';
CREATE DATABASE nombre_base_de_datos OWNER nombre_usuario;
\q
```

Donde:

- `CREATE USER` crea un nuevo rol de base de datos con contraseña.
- `CREATE DATABASE ... OWNER` crea la base de datos y asigna el usuario como propietario.
- `\q` cierra el intérprete `psql`.

### 7.7 Verificar acceso a PostgreSQL

Para confirmar que el usuario y la base de datos creados funcionan correctamente, se puede intentar una conexión directa:

```bash
psql -U nombre_usuario -d nombre_base_de_datos -h 127.0.0.1 -c "\conninfo"
```

Donde:

- `-U` especifica el usuario de PostgreSQL con el que se conecta.
- `-d` indica la base de datos a la que se conecta.
- `-h 127.0.0.1` indica que la conexión se realiza a través de TCP/IP al host local.
- `-c "\conninfo"` ejecuta el comando `\conninfo`, que muestra los detalles de la conexión activa.

Si la conexión es exitosa, `psql` solicitará la contraseña y mostrará una tabla con la información de la sesión, incluyendo la base de datos, el usuario, el host y el puerto (`5432` por defecto).

Con esto, la instalación y configuración básica de **PostgreSQL** queda lista para utilizarse en proyectos Laravel.

### 7.8 Otorgar privilegios de superusuario (opcional, recomendado en desarrollo)

Por defecto, el usuario creado en el paso anterior solo tiene acceso a la base de datos de la que es propietario. Para un entorno de desarrollo local es conveniente otorgarle privilegios de **superusuario**, lo que permite crear, eliminar y acceder a cualquier base de datos sin necesidad de usar el usuario `postgres`.

Para elevar los privilegios del usuario se ejecuta:

```bash
sudo -u postgres psql -c "ALTER USER nombre_usuario WITH SUPERUSER;"
```

Para verificar que el cambio se aplicó correctamente:

```bash
sudo -u postgres psql -c "\du"
```

La salida mostrará el listado de roles. El usuario debe aparecer con el atributo `Superusuario`:

```
                          Listado de roles
 Nombre de rol  |                         Atributos
----------------+------------------------------------------------------------
 nombre_usuario | Superusuario
 postgres       | Superusuario, Crear rol, Crear BD, Replicación, Ignora RLS
```

✅ Nota:
Otorgar privilegios de superusuario es adecuado en entornos de desarrollo local. En entornos de producción se recomienda aplicar el principio de mínimo privilegio y conceder únicamente los permisos estrictamente necesarios.

## 8. Instalación de Redis

**Redis** es una base de datos en memoria que Laravel puede utilizar como backend para **caché**, **sesiones** y **colas** (`queues`), ofreciendo un rendimiento muy superior al de los drivers basados en archivos o en base de datos.

### 8.1 Instalar el servidor Redis

Para instalar Redis en Fedora, se ejecuta el siguiente comando:

```bash
sudo dnf install redis -y
```

### 8.2 Iniciar y habilitar el servicio

Una vez instalado, se inicia el servicio y se habilita para que arranque junto con el sistema:

```bash
sudo systemctl start redis
sudo systemctl enable redis
```

Se puede comprobar que el servicio está activo con:

```bash
systemctl status redis
```

### 8.3 Verificar el funcionamiento de Redis

Redis incluye un cliente de línea de comandos (`redis-cli`) que permite probar la conexión al servidor:

```bash
redis-cli ping
```

Si el servidor responde correctamente, la salida será:

```
PONG
```

### 8.4 Instalar la extensión de PHP para Redis

Para que Laravel pueda comunicarse con Redis a través de PHP, es necesario instalar la extensión `php-redis`:

```bash
sudo dnf install php-redis -y
```

✅ Nota:
En Fedora, `php-redis` es un alias del paquete real `php-pecl-redis6`. Si al ejecutar el comando anterior aparece un mensaje indicando que `php-pecl-redis6` ya está instalado, significa que la extensión ya se encuentra disponible y no es necesario hacer nada más en este paso.

Después de instalar la extensión, se debe reiniciar el servicio PHP correspondiente para que los cambios surtan efecto, según el servidor web utilizado:

**Si usas Apache:**

```bash
sudo systemctl restart httpd.service
```

**Si usas Nginx (con PHP-FPM):**

```bash
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

Para confirmar que la extensión está activa:

```bash
php -m | grep redis
```

Si la salida muestra `redis`, la extensión está correctamente instalada y disponible para Laravel.

### 8.5 Configurar Laravel para utilizar Redis

En el archivo `.env` del proyecto Laravel, se deben ajustar las variables de conexión a Redis:

```env
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
```

Donde:

- `CACHE_STORE=redis` indica a Laravel que utilice Redis como almacén de caché.
- `SESSION_DRIVER=redis` hace que las sesiones de usuario se gestionen a través de Redis.
- `QUEUE_CONNECTION=redis` permite que los `jobs` encolados se procesen utilizando Redis como backend de colas.

✅ Nota:
Por defecto, Redis en Fedora solo acepta conexiones locales (`127.0.0.1`), lo cual es adecuado para un entorno de desarrollo. En producción se recomienda configurar una contraseña (`requirepass` en `/etc/redis/redis.conf`) y restringir el acceso únicamente a los hosts necesarios.

## 9. Instalación de Mailpit

**Mailpit** es un servidor SMTP de pruebas que captura todos los correos enviados por la aplicación durante el desarrollo, sin entregarlos realmente a destinatarios externos. Permite visualizar los correos enviados por Laravel (registro de usuarios, recuperación de contraseña, notificaciones, etc.) a través de una interfaz web.

### 9.1 Descargar el binario de Mailpit

Mailpit no está disponible en los repositorios oficiales de Fedora, por lo que se instala mediante su script oficial de instalación:

```bash
sudo bash -c "$(curl -sL https://raw.githubusercontent.com/axllent/mailpit/develop/install.sh)"
```

Este comando descarga el binario de Mailpit correspondiente a la arquitectura del sistema y lo instala en `/usr/local/bin/mailpit`.

✅ Nota:
Es recomendable revisar el script en el repositorio oficial de [Mailpit](https://github.com/axllent/mailpit) antes de ejecutarlo, como buena práctica de seguridad al instalar software mediante scripts de terceros.

### 9.2 Crear un servicio systemd para Mailpit

Para que Mailpit se ejecute como un servicio en segundo plano y se inicie automáticamente con el sistema, se crea un archivo de unidad de systemd:

```bash
sudo nano /etc/systemd/system/mailpit.service
```

Con el siguiente contenido:

```ini
[Unit]
Description=Mailpit SMTP Test Server
After=network.target

[Service]
ExecStart=/usr/local/bin/mailpit --listen 127.0.0.1:8025 --smtp 127.0.0.1:1025
Restart=on-failure
User=orlandoduranpy

[Install]
WantedBy=multi-user.target
```

Donde:

- `--listen 127.0.0.1:8025` define el puerto de la interfaz web donde se visualizan los correos capturados.
- `--smtp 127.0.0.1:1025` define el puerto SMTP que utilizará Laravel para enviar los correos.
- `User=orlandoduranpy` ejecuta el proceso bajo el usuario del sistema en lugar de `root`.

### 9.3 Iniciar y habilitar el servicio

Después de crear el archivo de servicio, se recarga la configuración de systemd y se inicia Mailpit:

```bash
sudo systemctl daemon-reload
sudo systemctl start mailpit
sudo systemctl enable mailpit
```

Se puede verificar el estado del servicio con:

```bash
systemctl status mailpit
```

### 9.4 Acceder a la interfaz web de Mailpit

Una vez iniciado el servicio, la interfaz web de Mailpit estará disponible en:

```
http://127.0.0.1:8025
```

Desde esta interfaz se pueden visualizar todos los correos capturados, junto con su contenido en HTML y texto plano.

### 9.5 Configurar Laravel para utilizar Mailpit

En el archivo `.env` del proyecto Laravel, se deben ajustar las variables de configuración de correo:

```env
MAIL_MAILER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hola@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

Con esta configuración, cualquier correo enviado desde Laravel (mediante `Mail::send`, notificaciones, etc.) será capturado por Mailpit y podrá visualizarse desde su interfaz web, en lugar de ser entregado a un servidor de correo real.

✅ Nota:
Mailpit es ideal únicamente para entornos de desarrollo. En producción se debe configurar un proveedor de correo real (SMTP, Mailgun, SES, etc.).

## 10. Instalación de DBeaver

**DBeaver** es un administrador gráfico de bases de datos de código libre, compatible con MySQL, PostgreSQL y muchos otros motores. Permite explorar bases de datos, ejecutar consultas SQL, editar datos y administrar conexiones desde una interfaz de escritorio, como alternativa a herramientas como phpMyAdmin.

### 10.1 Instalar DBeaver mediante Flatpak

En Fedora, la forma más sencilla de instalar DBeaver es a través de **Flatpak**, ya que no está disponible en los repositorios oficiales de `dnf`.

Si Flatpak no está habilitado en el sistema, primero se agrega el repositorio de Flathub:

```bash
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Luego se instala DBeaver Community Edition:

```bash
flatpak install flathub io.dbeaver.DBeaverCommunity -y
```

### 10.2 Iniciar DBeaver

Una vez instalado, se puede iniciar desde el menú de aplicaciones de Fedora, o ejecutando:

```bash
flatpak run io.dbeaver.DBeaverCommunity
```

### 10.3 Conectarse a MySQL desde DBeaver

Para crear una nueva conexión a MySQL:

1. Abrir DBeaver y hacer clic en **Nueva conexión a base de datos** (icono del enchufe).
2. Seleccionar **MySQL** en la lista de controladores.
3. Completar los datos de conexión:
   - **Host:** `127.0.0.1`
   - **Puerto:** `3306`
   - **Usuario:** el usuario creado en la sección 6.5 (`nombre_usuario`).
   - **Contraseña:** la contraseña configurada para ese usuario.
4. Hacer clic en **Probar conexión...** para verificar que los datos son correctos.
5. Si DBeaver solicita descargar el controlador JDBC de MySQL la primera vez, aceptar la descarga.
6. Hacer clic en **Finalizar** para guardar la conexión.

### 10.4 Conectarse a PostgreSQL desde DBeaver

Para crear una nueva conexión a PostgreSQL:

1. Abrir DBeaver y hacer clic en **Nueva conexión a base de datos**.
2. Seleccionar **PostgreSQL** en la lista de controladores.
3. Completar los datos de conexión:
   - **Host:** `127.0.0.1`
   - **Puerto:** `5432`
   - **Base de datos:** la base de datos creada en la sección 7.6 (`nombre_base_de_datos`).
   - **Usuario:** el usuario creado en esa misma sección (`nombre_usuario`).
   - **Contraseña:** la contraseña configurada para ese usuario.
4. Hacer clic en **Probar conexión...** para verificar que los datos son correctos.
5. Si DBeaver solicita descargar el controlador JDBC de PostgreSQL la primera vez, aceptar la descarga.
6. Hacer clic en **Finalizar** para guardar la conexión.

✅ Nota:
Si el usuario de PostgreSQL fue elevado a superusuario (sección 7.8), también podrá visualizar y administrar el resto de bases de datos del servidor desde la misma conexión en DBeaver.

✅ Nota:
Si al conectarte obtienes el error `Connection refused`, revisa que hayas habilitado las conexiones por TCP/IP (sección 7.4). Si obtienes un error de autenticación tipo `la autentificación Ident falló`, revisa el método configurado en `pg_hba.conf` (sección 7.5).
