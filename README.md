# Auth Service

Este es el servicio de autenticacion del proyecto. Se encarga de manejar los usuarios y generar tokens de acceso que los demas servicios usan para verificar que las peticiones son validas. Corre en el puerto 8001.

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

## Descripcion del servicio

Este servicio es uno de tres que forman la arquitectura de microservicios del proyecto. Su unica responsabilidad es la autenticacion. No sabe nada de piezas ni de proyectos, solo maneja usuarios y tokens.

Cuando el usuario inicia sesion desde el frontend este servicio valida las credenciales y genera un token con Laravel Sanctum. Ese token viaja al frontend, se guarda en el localStorage, y desde ese momento se envia automaticamente en cada peticion al pieces-service. El pieces-service a su vez consulta a este servicio para verificar si el token es valido antes de responder.

---

## Endpoints principales

### POST /api/login

No requiere token. Recibe el correo y la contrasena y devuelve un token de acceso.

Body:

```json
{
    "email": "admin@test.com",
    "password": "password123"
}
```

Respuesta exitosa codigo 200:

```json
{
    "access_token": "5|v5FFuMW3Bjs2AA8IEdeEiM1EepkHtl2qPIIPpyUEcf6cfb30",
    "token_type": "Bearer",
    "user": {
        "id": 3,
        "name": "Admin",
        "email": "admin@test.com"
    }
}
```

Respuesta credenciales incorrectas codigo 401:

```json
{
    "message": "Credenciales incorrectas."
}
```

### GET /api/me

Requiere token en el header. Devuelve los datos del usuario autenticado. Este endpoint lo usa el pieces-service para verificar si un token es valido.

Header: Authorization: Bearer TU_TOKEN

### POST /api/logout

Requiere token en el header. Elimina el token de la base de datos.

Header: Authorization: Bearer TU_TOKEN

Respuesta:

```json
{
    "message": "Sesion cerrada correctamente."
}
```


## Variables de entorno

APP_NAME - nombre de la aplicacion - AuthService
APP_TIMEZONE - zona horaria - America/Bogota
DB_CONNECTION - motor de base de datos - mysql
DB_HOST - host de MySQL - 127.0.0.1
DB_PORT - puerto de MySQL - 3306
DB_DATABASE - nombre de la base de datos - auth_service
DB_USERNAME - usuario de MySQL - root
DB_PASSWORD - contrasena de MySQL - vacio si no tiene contrasena



## Pasos de ejecucion

Paso 1 - Clona el repositorio

git clone https://github.com/Andresq11/auth-service.git
cd auth-service

Paso 2 - Instala las dependencias

composer install

Paso 3 - Copia el archivo de configuracion

En Windows: copy .env.example .env
En Mac o Linux: cp .env.example .env

Paso 4 - Genera la clave de la aplicacion

php artisan key:generate

Paso 5 - Configura el .env

APP_NAME=AuthService
APP_TIMEZONE=America/Bogota
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=auth_service
DB_USERNAME=root
DB_PASSWORD=

Paso 6 - Crea la base de datos auth_service en phpMyAdmin con cotejamiento utf8mb4_general_ci

Paso 7 - Corre las migraciones

php artisan migrate

Paso 8 - Crea el usuario de prueba

php artisan db:seed

Usuario: admin@test.com
Password: password123

Paso 9 - Levanta el servidor

php artisan serve --port=8001



## Como funciona el flujo completo

### El usuario inicia sesion

El frontend manda una peticion POST a http://127.0.0.1:8001/api/login con el correo y la contrasena. El controlador en app/Http/Controllers/AuthController.php recibe esa peticion y hace esto:

```php
$user = User::where('email', $request->email)->first();

if (! $user || ! Hash::check($request->password, $user->password)) {
    return response()->json(['message' => 'Credenciales incorrectas.'], 401);
}

$user->tokens()->delete();

$token = $user->createToken('auth_token')->plainTextToken;
```

Primero busca el usuario en la base de datos. Luego verifica la contrasena con Hash::check que compara la contrasena escrita con la que esta encriptada en la base de datos. Si todo esta bien elimina los tokens anteriores del usuario para no acumularlos y genera uno nuevo con Sanctum.

Sanctum guarda el token en la tabla personal_access_tokens de la base de datos auth_service. El token se ve asi:

5|v5FFuMW3Bjs2AA8IEdeEiM1EepkHtl2qPIIPpyUEcf6cfb30

El numero antes del pipe es el ID del registro en la tabla. El frontend guarda ese token en el localStorage.

### El pieces-service valida el token

Cada vez que el frontend hace una peticion al pieces-service envia el token en el header Authorization. El pieces-service antes de responder hace una peticion GET a http://127.0.0.1:8001/api/me con ese mismo token.

La ruta /api/me esta protegida por el middleware auth:sanctum en routes/api.php:

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
    Route::post('/logout', [AuthController::class, 'logout']);
});
```

Sanctum busca el token en la tabla personal_access_tokens. Si lo encuentra devuelve los datos del usuario con codigo 200. Si no lo encuentra devuelve 401. El pieces-service usa ese codigo para decidir si responde o no.

### El usuario cierra sesion

El frontend manda una peticion POST a /api/logout. El servidor elimina el token de la base de datos:

```php
public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();
    return response()->json(['message' => 'Sesion cerrada correctamente.']);
}
```

Desde ese momento el token ya no existe en la base de datos y aunque alguien lo tenga guardado no podra usarlo porque Sanctum no lo va a encontrar.


## Decisiones tecnicas

Use Sanctum en lugar de JWT porque viene incluido con Laravel y no necesita paquetes externos. Cada token queda en la base de datos lo que permite eliminarlo cuando el usuario cierra sesion. Con JWT eso no se puede hacer facilmente porque el token vive en el cliente.

Agregue $user->tokens()->delete() antes de crear el token nuevo porque en las pruebas me di cuenta que se iban acumulando tokens viejos en la tabla personal_access_tokens cada vez que el usuario iniciaba sesion.

Configure el timezone en America/Bogota porque por defecto Laravel usa UTC y las fechas aparecian con 5 horas de diferencia con la hora de Colombia.

En Laravel 13 el archivo routes/api.php no viene configurado por defecto, entonces tuve que registrarlo manualmente en bootstrap/app.php para que las rutas de la API funcionaran.