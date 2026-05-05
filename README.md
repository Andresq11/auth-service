# Auth Service

Este es el servicio de autenticacion de la aplicacion. Su unica responsabilidad es manejar usuarios y generar tokens de acceso. Cuando el usuario inicia sesion desde el frontend, este servicio valida las credenciales y devuelve un token que los demas servicios usan para verificar que la peticion es valida.

## Como funciona dentro de la arquitectura

La aplicacion esta compuesta por tres servicios independientes que se comunican entre si. Este servicio es el punto de entrada para la autenticacion. El flujo es el siguiente:

1. El usuario abre el frontend y escribe su correo y contrasena
2. El frontend envia esas credenciales a este servicio en el endpoint /api/login
3. Este servicio valida las credenciales contra la base de datos
4. Si son correctas genera un token con Laravel Sanctum y lo devuelve
5. El frontend guarda ese token y lo envia en cada peticion al pieces-service
6. El pieces-service consulta a este servicio si el token es valido antes de responder

## Tecnologias

- PHP 8.4.6
- Laravel 13.7.0
- Laravel Sanctum 4.x
- MySQL 8.0
- Composer 2.x

## Requisitos previos

Antes de instalar este proyecto necesitas tener en tu maquina:

- PHP 8.2 o superior
- Composer
- MySQL
- XAMPP o cualquier servidor local con MySQL activo

## Instalacion paso a paso

Paso 1 - Clona el repositorio

git clone https://github.com/Andresq11/auth-service.git

Paso 2 - Entra a la carpeta del proyecto

cd auth-service

Paso 3 - Instala las dependencias de PHP

composer install

Paso 4 - Copia el archivo de configuracion

cp .env.example .env

En Windows usa este comando en su lugar

copy .env.example .env

Paso 5 - Genera la clave de la aplicacion

php artisan key:generate

Paso 6 - Configura la base de datos en el archivo .env

Abre el archivo .env y busca estas lineas, asegurate de que queden exactamente asi

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=auth_service
DB_USERNAME=root
DB_PASSWORD=

Paso 7 - Crea la base de datos

Abre phpMyAdmin en http://localhost/phpmyadmin y crea una base de datos llamada auth_service con cotejamiento utf8mb4_general_ci

Paso 8 - Corre las migraciones

Este comando crea todas las tablas necesarias en la base de datos

php artisan migrate

Las tablas que se crean son users, personal_access_tokens, cache y jobs

Paso 9 - Crea el usuario de prueba

php artisan db:seed

Esto crea un usuario con el que puedes hacer pruebas

Paso 10 - Levanta el servidor

php artisan serve --port=8001

El servicio queda corriendo en http://127.0.0.1:8001

## Usuario de prueba

Email: admin@test.com
Password: password123

## Endpoints disponibles

POST /api/login

No requiere token. Recibe el correo y la contrasena y devuelve un token de acceso.

Body que se envia

email: admin@test.com
password: password123

Respuesta cuando las credenciales son correctas

access_token: el token generado por Sanctum
token_type: Bearer
user: nombre, correo y datos del usuario

Respuesta cuando las credenciales son incorrectas

message: Credenciales incorrectas
codigo HTTP: 401

GET /api/me

Requiere token en el header. Devuelve los datos del usuario autenticado.

Header requerido: Authorization: Bearer TU_TOKEN

Respuesta: id, name, email, created_at del usuario

POST /api/logout

Requiere token en el header. Revoca el token en la base de datos, lo que significa que ese token ya no servira para nada aunque alguien lo tenga guardado.

Header requerido: Authorization: Bearer TU_TOKEN

Respuesta: message: Sesion cerrada correctamente

## Estructura de archivos importantes

app/Http/Controllers/AuthController.php es el controlador principal. Tiene tres metodos: login que valida credenciales y genera el token, me que devuelve el usuario autenticado, y logout que revoca el token.

app/Models/User.php es el modelo del usuario. Tiene el trait HasApiTokens de Sanctum que es lo que permite generar y revocar tokens.

routes/api.php define las tres rutas del servicio. El login es publico y las otras dos estan protegidas por el middleware auth:sanctum.

database/migrations contiene las migraciones que crean las tablas en la base de datos.

database/seeders/DatabaseSeeder.php crea el usuario de prueba cuando corres php artisan db:seed.

bootstrap/app.php configura el proyecto incluyendo el archivo de rutas api.php.

## Variables de entorno

APP_NAME - Nombre de la aplicacion - AuthService
APP_URL - URL donde corre el servicio - http://localhost:8001
DB_CONNECTION - Motor de base de datos - mysql
DB_HOST - Host de MySQL - 127.0.0.1
DB_PORT - Puerto de MySQL - 3306
DB_DATABASE - Nombre de la base de datos - auth_service
DB_USERNAME - Usuario de MySQL - root
DB_PASSWORD - Contrasena de MySQL - vacio si no tiene contrasena

## Decisiones tecnicas

Se uso Laravel Sanctum en lugar de JWT porque Sanctum viene integrado con Laravel, no necesita paquetes externos y es suficiente para este tipo de proyecto. Cada token queda guardado en la tabla personal_access_tokens, lo que permite revocarlos en cualquier momento desde el endpoint de logout.

Las rutas estan separadas en dos grupos. El grupo publico solo tiene el login. El grupo protegido tiene me y logout, y usa el middleware auth:sanctum que verifica el token antes de ejecutar cualquier accion.

Cada servicio tiene su propia base de datos para respetar el principio de separacion de responsabilidades en microservicios. El auth-service es el unico dueno de la informacion de los usuarios.