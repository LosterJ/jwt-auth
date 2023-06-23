1. Tao project + connect db + dung tables in db
    composer create-project laravel/laravel laravel-jwt-auth --prefer-dist

    Chinh .env file

    php artisan migrate

2. Tai va cau hinh jwt
    composer require tymon/jwt-auth
3. config/app.php (add providers)
    'providers' => [
        ....
        ....
        Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
    ],
    'aliases' => [
        ....
        'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class,
        'JWTFactory' => Tymon\JWTAuth\Facades\JWTFactory::class,
        ....
    ],
4. php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

5. php artisan jwt:secret

II. Setup user
1. app/Models/User.php
    use Tymon\JWTAuth\Contracts\JWTSubject;

    implements JWTSubject

    protected $fillable = [];
    protected $hidden = ['password','remember_token'];

    public funtion getJWTIdentifier() {
        return $this->getKey();
    }

    public function getJWTCustomClaims() {
        return [];
    }
2. config/auth.php
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
III. Create AuthController
1. php artisan make:controller AuthController
2. app/Http/Controllers/AuthController.php
    use Illuminate\Support\Facades\Auth;
    use App\Models\User;
    use Validator;

    public function __construct() {
        $this->middleware('auth:api', ['except' => ['login', 'register']]);
    }
IV. Add auth route
1. routes/api.php
    use Illuminate\Support\Facades\Route;
    use App\Http\Controllers\AuthController;

    Route::group([
        'middleware' => 'api',
        'prefix' => 'auth'
    ], function ($router) {
        Route::post('/login', [AuthController::class, 'login']);
        Route::post('/register', [AuthController::class, 'register']);
        Route::post('/logout', [AuthController::class, 'logout']);
        Route::post('/refresh', [AuthController::class, 'refresh']);
        Route::get('/user-profile', [AuthController::class, 'userProfile']);    
    });
V. php artisan serve

**Test**
POST	/api/auth/register          body.form-data:name,email,password,password_confirmation
POST	/api/auth/login             body.form-data:email,password
GET	    /api/auth/user-profile      |
POST	/api/auth/refresh           |
POST	/api/auth/logout            |
| -> auth.token