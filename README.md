# Auth Service

Servicio de autenticacion de la aplicacion. Se encarga de validar credenciales y generar tokens de acceso que los demas servicios usan para verificar que las peticiones son validas.

## Tecnologias

- PHP 8.4.6
- Laravel 13.7.0
- Laravel Sanctum 4.x
- MySQL 8.0
- Composer 2.x

## Requisitos previos

- PHP 8.2 o superior
- Composer
- MySQL
- XAMPP o cualquier servidor local

## Instalacion

Clona el repositorio

git clone https://github.com/Andresq11/auth-service.git

Entra a la carpeta

cd auth-service

Instala las dependencias

composer install

Copia el archivo de configuracion

cp .env.example .env

Genera la clave de la aplicacion

php artisan key:generate

Abre el .env y configura la base de datos asi

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=auth_service
DB_USERNAME=root
DB_PASSWORD=

Crea la base de datos auth_service en MySQL o phpMyAdmin

Corre las migraciones

php artisan migrate

Crea el usuario de prueba

php artisan db:seed

Levanta el servidor

php artisan serve --port=8001

El servicio queda corriendo en http://127.0.0.1:8001

## Usuario de prueba

Email: admin@test.com
Password: password123

## Endpoints

POST /api/login - Inicia sesion y retorna un token de acceso. No requiere token.

Ejemplo de para ingresar

email: admin@test.com
password: password123

Respuesta exitosa

access_token: el token generado
token_type: Bearer
user: datos del usuario

GET /api/me - Retorna la informacion del usuario autenticado. Requiere el header Authorization: Bearer TU_TOKEN

POST /api/logout - Cierra la sesion y revoca el token en la base de datos. Requiere el header Authorization: Bearer TU_TOKEN

## Variables de entorno

APP_NAME - Nombre de la app - AuthService
APP_URL - URL de la app - http://localhost:8001
DB_CONNECTION - Motor de base de datos - mysql
DB_HOST - Host de la base de datos - 127.0.0.1
DB_PORT - Puerto de MySQL - 3306
DB_DATABASE - Nombre de la base de datos - auth_service
DB_USERNAME - Usuario de MySQL - root
DB_PASSWORD - Contrasena de MySQL - vacio por defecto

## Como funciona

Cuando el usuario hace login el servicio valida el correo y la contrasena contra la base de datos. Si son correctos genera un token con Sanctum y lo devuelve en la respuesta. Ese token se guarda en el frontend y se envia en cada peticion al pieces-service para demostrar que el usuario esta autenticado.

Cuando el usuario cierra sesion el token se revoca en la base de datos, lo que significa que aunque alguien tenga el token ya no podra usarlo.

## Decisiones tecnicas

Se uso Laravel Sanctum para la generacion de tokens porque es la opcion oficial de Laravel para APIs. Es facil de configurar y cada token queda guardado en la base de datos en la tabla personal_access_tokens, lo que permite revocarlos en cualquier momento.

Se decidio no usar JWT porque Sanctum es suficiente para este proyecto y viene integrado con Laravel sin necesidad de paquetes externos.

Las rutas estan separadas en dos grupos: las publicas como el login que no requieren token, y las protegidas como me y logout que verifican el token antes de responder.