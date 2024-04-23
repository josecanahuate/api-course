****************************    LARAVEL API's ******************************

Documentacion Instalacion JWT TOKEN --> 	

	https://jwt-auth.readthedocs.io/en/develop/laravel-installation/


1- Crear proyecto

2- Crear la BD

	Modificar el archivo .env en DATABASE que tenga el mismo nombre con la 	BD nueva que vamos a Crear. (usualmente tomamos el mismo nombre que 	tiene predifinido pero si queremos que nuestra BD tenga otro nombre 	debemos cambiarlo en en .env)

3- incluir el codigo schema en appserviceprovider (boot)

use Illuminate\Support\Facades\Schema;

public function boot(): void
    {
        Schema::defaultStringLength(191);
    }


4- migrar las tablas a la BD

	php artisan migrate

5- ejecutar comando:

	composer require tymon/jwt-auth

6- Una vez que se complete la instalación, copia y pega esta linea en 
config\app.php:

    App\Providers\RouteServiceProvider::class,

        Tymon\JWTAuth\Providers\LaravelServiceProvider::class,

7- Una vez pegada, publicamos las configuraciones del paquete ejecutando el siguiente comando:

php artisan vendor:publish --provider="Tymon\JWTAuth\Provider \LaravelServiceProvider"

	Esto copiará los archivos de configuración necesarios en tu proyecto 	para que puedas personalizar la configuración de JWT Auth según tus 	necesidades.

9- Luego, genera la clave secreta que se utilizará para firmar los tokens JWT ejecutando el siguiente comando (esta clave se creara automaticamente en .env):

	php artisan jwt:secret

10- Cambiar forma de autenticacion predefinida de Laravel en config\auth.php, lo colocaremos exactamente de esta manera, aca debajo:

https://jwt-auth.readthedocs.io/en/develop/quick-start/#configure-auth-guard


    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

    
    'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
            'hash' => false,
    ],

	
	




11- Ir al Models/User y agregar lo siguiente:
https://jwt-auth.readthedocs.io/en/develop/quick-start/#update-your-user-model

a) use Tymon\JWTAuth\Contracts\JWTSubject;

b) class User extends Authenticatable implements JWTSubject 			(se reemplazara esta por la anterior)

c) agregar esto despues de protected $cast

   public function getJWTIdentifier() {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims() {
        return [];
    }  
}





12 - Ahora definiremos las rutas en routes/api.php

https://jwt-auth.readthedocs.io/en/develop/quick-start/#add-some-basic-authentication-routes


Route::middleware('api')->prefix('auth')->group(function () {
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout']);
    Route::post('refresh', [AuthController::class, 'refresh']);
    Route::post('me', [AuthController::class, 'me']);
});

12.1) Crear el Controlador AuthController
	
	php artisan make:controller AuthController

12.2) Modificar el Auth Controller
https://jwt-auth.readthedocs.io/en/develop/quick-start/#add-some-basic-authentication-routes --> (Create the AuthController)

app\Http\Controllers\AuthController.php





13) Probar con PostMan:
Rutas: Route::middleware('api')->prefix('auth')->group(function () {
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout']);
    Route::post('refresh', [AuthController::class, 'refresh']);
    Route::post('me', [AuthController::class, 'me']);
    Route::post('register', [AuthController::class, 'register']);
});

api = middleware
auth= prefix
register = ruta

A) Registrar un Nuevo Usuario: http://127.0.0.1:8000/api/auth/register
a.a) Vamos al body, seleccionar form-data, y en key creamos 3 campos: nombre, email y password & LES ASIGNAMOS VALORES A CADA UNO.

Nombre: Juan Carlos
Email: jcromero@gmail.com
Password: 12345678

				PRESIONAR SEND!!!!


B) HACER LOGIN:
	
	http://127.0.0.1:8000/api/auth/login

b.a) Deseleccionamos el ganchito de name y Presionamos SEND!! 

b.b) SE NOS IMPRIMIRA UN TOKEN, CON ESTE TOKEN REALIZAREMOS EL INICIO DE SESION.

b.c) Copiamos este Token, nos vamos a la ruta (Perfil de Usuario):

	http://127.0.0.1:8000/api/auth/me

Nos dirigimos a la parte de autorization en postman, elegimos autorization bearer, y colocamos el token. (si tiene un token lo quitamos y colocamos el nuevo)

b.d) SI TODO SALE BIEN, NOS DEVOLVERA LOS DATOS DEL USUARIO:
{
    "id": 1,
    "name": "Juan Carlos Romero",
    "email": "jcromero@gmail.com",
    "email_verified_at": null,
    "created_at": "2024-03-12T01:14:38.000000Z",
    "updated_at": "2024-03-12T01:14:38.000000Z"
}





****************  DESARROLLAR API REST CRUD CON LARAVEL *******************

REALIZAREMOS UN API QUE SE BASARA EN UN CRUD CON VUE + POSTMAN

1 - Crear Proyecto

2- Modificar app/Providers, incluir el codigo schema en appserviceprovider (boot)

use Illuminate\Support\Facades\Schema;

public function boot(): void
    {
        Schema::defaultStringLength(191);
    }

3- Modificar el archivo .env en DATABASE que tenga el mismo nombre con la BD nueva que vamos a Crear. (usualmente tomamos el mismo nombre que tiene predifinido pero si queremos que nuestra BD tenga otro nombre debemos cambiarlo en en .env) (EL NOMBRE DEL DATABASE-NAME DEL .env DEBE SER EL MISMO QUE EL NOMBRE DE LA BD DE phpmyadmin)


3- Migrar las tablas a la BD

	php artisan migrate


4- Crear el Modelo
	
	php artisan make:model Articulo

5- Crearemos la tabla articulos junto con su migracion.

	php artisan make:migration create_articulos_table


6- Modificar la migracion: database\migrations\2024_03_12_015049_create_articulos_table.php

    public function up(): void
    {
        Schema::create('articulos', function (Blueprint $table) {
            $table->id();
            $table->string('descripcion');
            $table->float('precio');
            $table->integer('stock');
            $table->timestamps();
        });
    }


7- Modificar Modelo: app\Models\Articulo.php

class Articulo extends Model
{
    use HasFactory;
    protected $fillable = ['descripcion', 'precio', 'stock'];
}

8- Migrar Nuevamente para que se actualizen los CAMBIOS.

	php artisan migrate

-REVISAR SI LOS CAMBIOS SE HICIERON!!!
-SINO SE HICIERON EJECUTAR: php artisan migrate:refresh


9- Crear el Controlador tipo CRUD para Articulos:

	php artisan make:controller ArticuloController --resource

AL EJECUTARLO CON (--resource) YA NOS TRAE DEFINIDO TODOS LOS METODOS DEL CRUD: index, store, create, delete, update, edit y destroy.


9.1) Modificamos el ArticuloController con los metodos CRUD ya predefinidos.

app\Http\Controllers\ArticuloController.php


<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Articulo;

class ArticuloController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $articulos = Articulo::all();
        return $articulos;
    }

    /**
     * Store a newly created resource in storage.
     */
    //ALMACENANDO LOS VALORES
    public function store(Request $request)
    {
        $articulo = new Articulo();
        $articulo->descripcion = $request->descripcion;
        $articulo->precio = $request->precio;
        $articulo->stock = $request->stock;
    }


    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request)
    {
        $articulo = Articulo::findOrFail($request->id);
        $articulo->descripcion = $request->descripcion;
        $articulo->precio = $request->precio;
        $articulo->stock = $request->stock;

        $articulo->save();

        return $articulo;

    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(Request $request)
    {
        $articulo = Articulo::destroy($request->id);
        return $articulo;
    }
}




10- Crear las rutas en routes/api.php

Route::get('/articulos', [ArticuloController::class, 'index']); // para mostrar los articulos
Route::get('/articulos', [ArticuloController::class, 'store']); //para almacenar
Route::get('/articulos/{id}', [ArticuloController::class, 'update']); //para actualizar
Route::get('/articulos/{id}', [ArticuloController::class, 'destroy']); //para eliminar


11- REVISAR QUE CORS ESTE PRESENTE Ubicacion: config\cors.php(para las versiones nuevas ya CORS viene instalado) (dejar el archivo asi tal como esta, NO MODIFICAR)

Laravel 9 y 10 ya incluyen funcionalidades para manejar CORS de forma nativa, por lo que el paquete fruitcake/laravel-cors se vuelve obsoleto en esas versiones.

12 - borrar cache
php artisan config:cache

13- Ejecutar nuevamente php artisan serve













********Desarrollo de una API con Laravel utilizando TDD y JSON:API*********

	ES PROYECTO NO LLEGUE A TERMINARLO DIO ERRORRRRRRRRRRRRRRRR

API ARTICULOS -> https://www.youtube.com/watch?v=fsiPXKzcH2M&list=PL1ZB-EKP1eiTX6WJmNvPPJFryOfMQf3YJ

1 - Creacion de proyecto te permite especificar la versión del framework y algunas otras opciones adicionales:
	
	composer create-project laravel/laravel company --prefer-dist

2 - Para crear el modelo, controlador y las migraciones en un solo COMANDO para departamento y empleado:

php artisan make:model Department -mcrf --api
php artisan make:model Employee -mcrf --api


3- Creacion de un Controlador Adicional AuthController que nos ayudara a realizar el login, logout y la creacion de usuarios para generar los tokens.

	php artisan make:controller AuthController 
		

4- Ir a .env y cambiamos el Database por 'company'

5- Nos Dirigimos a database\migrations\2024_03_12_191235_create_departments_table.php

En la tabla de departamentos agregamos el campo name de 100 caracteres:
    public function up(): void
    {
        Schema::create('departments', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100);
            $table->timestamps();
        });
    }

6 - Nos Dirigimos a la tabla de empleados: database\migrations\2024_03_12_191241_create_employees_table.php

    public function up(): void
    {
        Schema::create('employees', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100);
            $table->string('email', 80);
            $table->string('phone', 15);
            $table->string('department_id')->constrained('departments')
            ->onUpdate('cascade')->onDelete('restrict');
            $table->timestamps();
        });
 }

7- NOS DIRIGIMOS A LOS MODELOS:

**Modelo departamentos:

class Department extends Model
{
    use HasFactory;
    protected $fillable = ['name'];
}


**Modelo Empleados:

class Employee extends Model
{
    use HasFactory;
    protected $fillable = ['name', 'email', 'phone', 'department_id'];
}


8- Vamos a database\factories\DepartmentFactory.php

-Generaremos registros falsos

**Departamento

    public function definition(): array
    {
        return [
            'name'=> $this->faker->jobTitle
        ];
    }	


**Empleados

    public function definition(): array
    {
        return [
            'name'=> $this->faker->name, 
            'email'=> $this->faker->email, 
            'phone'=> $this->faker->e164PhoneNumber, 
            'department_id'=> $this->faker->numberBetween(1,6) 
        ];
    }


9 - IR a database\seeders\DatabaseSeeder.php

    public function run(): void
    {
        // User::factory(10)->create();
        \App\Models\Department::factory(6)->create();
        \App\Models\Employee::factory(25)->create();

        User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
    }


10- EJECUTAR LAS MIGRACIONES JUNTO CON LOS SEEDERS

	php artisan migrate --seed

11- *********** PROBAR ***********

EN POSTMAN HACER LO SIGUIENTE:

--Headers

Key            Value	
Accept         application/json


--Body (form-data)
key                       value
name	                Dpto Nuevo			

el name verificar si se queda asi mismo este tiene que ser el mismo nombre de la variable que estamos enviando & el value es el nombre del departamento que le pondremos.

EN EL CONTROLLER COLOCAR ESTO, DESPUES QUITAR:
dd($request->all());//solo para ver que esta llegando en la peticion

****************************************************************************





https://www.youtube.com/watch?v=VnIKwzysvHM

+++++CREAR PROYECTO CON VERSION 10
composer create-project --prefer-dist laravel/laravel:^10.10 nombre-del-proyecto ------> PROBAR

 
      **************************** API ***********************************
1- Crear Proyecto

2- Crear el modelo + otras configuraciones adicionales del API

	php artisan make:model Cliente -m --api

3- Agregar campos: 'nombre' y 'apellidos' a create_clientes
Y migrar los cambios php artisan migrate

4- Para ver todas las rutas
	php artisan route:list

5- POSTMAN


***********METODO POST
--Headers

Key            Value	
Accept         application/json


--Body (form-data)
key                       value
nombres 		  Jose
apellidos	         Canahuate	



***********METODO PUT
Deshabilitar en Body, los ganchitos habilitados y SELECCIONAR x-www-form-urlencoded

MODIFICAREMOS EL CLIENTE #7
	colocar nueva http en postman/put
	http://127.0.0.1:8000/api/clientes/7


7
test7
rodriguez7
2024-03-13 04:30:33
2024-03-13 04:30:33


				A

nuevo 7:

    {
        "id": 7,
        "nombres": "Luis",
        "apellidos": "Alvarado Romero",
        "created_at": "2024-03-13T04:30:33.000000Z",
        "updated_at": "2024-03-13T04:36:45.000000Z"
    }




If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
